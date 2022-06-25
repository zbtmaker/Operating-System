# Guava Cache缓存解读
## 缓存结构

### LRU缓存替换算法

### Concurrent HashMap

### Key Reference Queue

### Value Reference Queue

### Recency Queue

如果key 只在Cache当中，那么每次从Cache中获取（get）操作只需要将key放入到Recency Queue当中。这个操作主要提高并发读取的性能。

如果一个key不在Cache当中，那么首先需要将key存入Cache当中，然后将对应的需要将所有在Recency Queue中的Key存入到Eviction Queue中，然后根据LRU过期算法，将Queue对首元素出栈这样。


### Eviction Queue
如果key不在Cache中，那么需要先将key-value存入Cache中，然后将对应的key存入evictionQueue中。如果key在Cache中，每一次更新都需要将key保存在evictionQueue中。


### Write Queue

### Access Queue

## get操作

```java
    V get(K key, CacheLoader<? super K, V> loader) throws ExecutionException {
        int hash = hash(checkNotNull(key));
        return segmentFor(hash).get(key, hash, loader);
    }
```

从Segment中获取Key
```java
        V get(K key, int hash, CacheLoader<? super K, V> loader) throws ExecutionException {
            checkNotNull(key);
            checkNotNull(loader);
            try {
                if (count != 0) { // read-volatile
                    // don't call getLiveEntry, which would ignore loading values
                    ReferenceEntry<K, V> e = getEntry(key, hash);
                    if (e != null) {
                        long now = map.ticker.read();
                        V value = getLiveValue(e, now);
                        if (value != null) {
                            recordRead(e, now);
                            statsCounter.recordHits(1);
                            return scheduleRefresh(e, key, hash, value, now, loader);
                        }
                        ValueReference<K, V> valueReference = e.getValueReference();
                        if (valueReference.isLoading()) {
                            return waitForLoadingValue(e, key, valueReference);
                        }
                    }
                }

                // at this point e is either null or expired;
                return lockedGetOrLoad(key, hash, loader);
            } catch (ExecutionException ee) {
                Throwable cause = ee.getCause();
                if (cause instanceof Error) {
                    throw new ExecutionError((Error) cause);
                } else if (cause instanceof RuntimeException) {
                    throw new UncheckedExecutionException(cause);
                }
                throw ee;
            } finally {
                postReadCleanup();
            }
        }
```

```java
        V getLiveValue(ReferenceEntry<K, V> entry, long now) {
            if (entry.getKey() == null) {
                tryDrainReferenceQueues();
                return null;
            }
            V value = entry.getValueReference().get();
            if (value == null) {
                tryDrainReferenceQueues();
                return null;
            }

            if (map.isExpired(entry, now)) {
                tryExpireEntries(now);
                return null;
            }
            return value;
        }
```

```java
        V lockedGetOrLoad(K key, int hash, CacheLoader<? super K, V> loader) throws ExecutionException {
            ReferenceEntry<K, V> e;
            ValueReference<K, V> valueReference = null;
            LoadingValueReference<K, V> loadingValueReference = null;
            boolean createNewEntry = true;

            lock();
            try {
                // re-read ticker once inside the lock
                long now = map.ticker.read();
                preWriteCleanup(now);

                int newCount = this.count - 1;
                AtomicReferenceArray<ReferenceEntry<K, V>> table = this.table;
                int index = hash & (table.length() - 1);
                ReferenceEntry<K, V> first = table.get(index);

                for (e = first; e != null; e = e.getNext()) {
                    K entryKey = e.getKey();
                    if (e.getHash() == hash
                            && entryKey != null
                            && map.keyEquivalence.equivalent(key, entryKey)) {
                        valueReference = e.getValueReference();
                        if (valueReference.isLoading()) {
                            createNewEntry = false;
                        } else {
                            V value = valueReference.get();
                            if (value == null) {
                                enqueueNotification(
                                        entryKey, hash, value, valueReference.getWeight(), RemovalCause.COLLECTED);
                            } else if (map.isExpired(e, now)) {
                                // This is a duplicate check, as preWriteCleanup already purged expired
                                // entries, but let's accommodate an incorrect expiration queue.
                                enqueueNotification(
                                        entryKey, hash, value, valueReference.getWeight(), RemovalCause.EXPIRED);
                            } else {
                                recordLockedRead(e, now);
                                statsCounter.recordHits(1);
                                // we were concurrent with loading; don't consider refresh
                                return value;
                            }

                            // immediately reuse invalid entries
                            writeQueue.remove(e);
                            accessQueue.remove(e);
                            this.count = newCount; // write-volatile
                        }
                        break;
                    }
                }

                if (createNewEntry) {
                    loadingValueReference = new LoadingValueReference<>();

                    if (e == null) {
                        e = newEntry(key, hash, first);
                        e.setValueReference(loadingValueReference);
                        table.set(index, e);
                    } else {
                        e.setValueReference(loadingValueReference);
                    }
                }
            } finally {
                unlock();
                postWriteCleanup();
            }

            if (createNewEntry) {
                try {
                    // Synchronizes on the entry to allow failing fast when a recursive load is
                    // detected. This may be circumvented when an entry is copied, but will fail fast most
                    // of the time.
                    synchronized (e) {
                        return loadSync(key, hash, loadingValueReference, loader);
                    }
                } finally {
                    statsCounter.recordMisses(1);
                }
            } else {
                // The entry already exists. Wait for loading.
                return waitForLoadingValue(e, key, valueReference);
            }
        }
```
