之前一直很疑惑，如果一个apk包经过V1签名后，再使用V2进行签名，如果使用的证书是同一个，或者不同的证书进行签名的时候会有什么表现呢？其实对于签名一直不是很了解，V1签名会把开发者的数字证书存放在/META-INFO目录的CSR文件里，但是使用V2签名的应用

数字证书
数字签名


## 应用签名
先使用V2签名，但是不使用V1签名
```bash
./apksigner sign --min-sdk-version 21 -verbose --ks /Users/zoubaitao/Desktop/keystore --v1-signing-enabled false --v2-signing-enabled true --ks-key-alias key0 --ks-pass pass:zbt123456 --key-pass pass:zbt123456 --out /Users/zoubaitao/Desktop/signed-v2.apk /Users/zoubaitao/Desktop/app-debug-unsigned.apk
```
获取签名信息
```bash
./apksigner verify -verbose --print-certs --min-sdk-version 24 /Users/zoubaitao/Desktop/signed-v2.apk

Verifies
Verified using v1 scheme (JAR signing): false
Verified using v2 scheme (APK Signature Scheme v2): true
Verified using v3 scheme (APK Signature Scheme v3): true
Verified using v3.1 scheme (APK Signature Scheme v3.1): false
Verified using v4 scheme (APK Signature Scheme v4): false
Verified for SourceStamp: false
Number of signers: 1
Signer #1 certificate DN: C=518101, ST=Guangdong, L=Shenzhen, O=vivo, OU=Application, CN=Zou Baitao
Signer #1 certificate SHA-256 digest: 9ef58a4abab0dd6e4d4f559b83de3994dba62d85f192ac375bc00f19c4aa1467
Signer #1 certificate SHA-1 digest: 07cfe5f15f5c8c1fcc9c953a4aa1d96693fef4fb
Signer #1 certificate MD5 digest: 8541470bad8040192e4c19a89eac52f6
Signer #1 key algorithm: RSA
Signer #1 key size (bits): 2048
Signer #1 public key SHA-256 digest: d520f5400a50c5572e3e240f3faafef4cfb45c3516964f664f69f30bf92ee9af
Signer #1 public key SHA-1 digest: f7e3eece46957e8e3e5817038cd75e4982c11801
Signer #1 public key MD5 digest: 7e4454ebfb97dc426607b801602f2664
```

如果使用signed-v2.apk上面的签名信息
```bash
zoubaitao@192 35.0.0 % ./apksigner verify -verbose --print-certs /Users/zoubaitao/Desktop/signed-v2.apk
DOES NOT VERIFY
ERROR: Missing META-INF/MANIFEST.MF
```

然后使用V3签名
```bash
./apksigner sign --min-sdk-version 21 -verbose --ks /Users/zoubaitao/Desktop/keystore --v1-signing-enabled false --v3-signing-enabled true --ks-key-alias key1 --ks-pass pass:zbt123456 --key-pass pass:zbt123456 --out /Users/zoubaitao/Desktop/signed-v3.apk /Users/zoubaitao/Desktop/signed-v2.apk
```

### 多证书签名
之前工作过程会遇到apksigner获取数字证书的md5时，解析出来会存在多个的情况，之前一直说的是存在V2和V3签名。实际上V3是不允许多证书签名的，因此这种方式可以是只针对V2签名，也可以是对APK使用两个证书key0和key3两个数字证书（key0 和 key3 是两个证书）对V1、V2签名两次。
```bash
./apksigner sign --ks /Users/zoubaitao/Desktop/keystore --ks-key-alias key0 --ks-pass pass:zbt123456 --key-pass pass:zbt123456 --next-signer --ks /Users/zoubaitao/Desktop/keystore2 --ks-key-alias key3 --ks-pass pass:zbt123456 --key-pass pass:zbt123456 --v3-signing-enabled false --in /Users/zoubaitao/Desktop/app-debug-unsigned.apk --out app-signed.apk
```
我们可以看一下/META-INFO/目录下其实是有KEY0.SF和KEY0.RSA文件、KEY3.SF和KEY3.RSA文件。从生成的两个文件，我们知道其实是通过将APK使用两个证书签名了两次。因此在apk的签名区获取的证书也会存在两个，我们获取签名工具可以看到
```bash
zoubaitao@192 35.0.0 % ./apksigner verify -verbose --print-certs /Users/zoubaitao/Desktop/app-signed.apk 
Verifies
Verified using v1 scheme (JAR signing): true
Verified using v2 scheme (APK Signature Scheme v2): true
Verified using v3 scheme (APK Signature Scheme v3): false
Verified using v3.1 scheme (APK Signature Scheme v3.1): false
Verified using v4 scheme (APK Signature Scheme v4): false
Verified for SourceStamp: false
Number of signers: 2
Signer #1 certificate DN: C=518101, ST=Guangdong, L=Shenzhen, O=vivo, OU=Application, CN=Zou Baitao
Signer #1 certificate SHA-256 digest: 9ef58a4abab0dd6e4d4f559b83de3994dba62d85f192ac375bc00f19c4aa1467
Signer #1 certificate SHA-1 digest: 07cfe5f15f5c8c1fcc9c953a4aa1d96693fef4fb
Signer #1 certificate MD5 digest: 8541470bad8040192e4c19a89eac52f6
Signer #1 key algorithm: RSA
Signer #1 key size (bits): 2048
Signer #1 public key SHA-256 digest: d520f5400a50c5572e3e240f3faafef4cfb45c3516964f664f69f30bf92ee9af
Signer #1 public key SHA-1 digest: f7e3eece46957e8e3e5817038cd75e4982c11801
Signer #1 public key MD5 digest: 7e4454ebfb97dc426607b801602f2664
Signer #2 certificate DN: C=518101, ST=Guangdong, L=Shenzhen, O=vivo, OU=Application, CN=Zou Baitao
Signer #2 certificate SHA-256 digest: d1e5b04958d31cd5c42e55fd0be9372a8ae2e5fa7713a2746c35e659d39e4102
Signer #2 certificate SHA-1 digest: d24a7d69a1d3e70ba86ee3a6186e70541685c52a
Signer #2 certificate MD5 digest: 144ef5c8fea5b8819e36e6123709ab52
Signer #2 key algorithm: RSA
Signer #2 key size (bits): 2048
Signer #2 public key SHA-256 digest: 92dfca1fbb8471c4561e6b920843e534d4312f205e7ab63dbce30decd82b2519
Signer #2 public key SHA-1 digest: 696af662e16fbcf4000b5f352385e2ce36ac74ea
Signer #2 public key MD5 digest: 2a788a5ebbd1327d412f82f8b9442ba4
```

我们使用下面的命令可以看一下，这里禁用V1和V3，只有V2签名方式可以用
```bash
./apksigner sign --ks /Users/zoubaitao/Desktop/keystore --ks-key-alias key0 --ks-pass pass:zbt123456 --key-pass pass:zbt123456 --next-signer --ks /Users/zoubaitao/Desktop/keystore2 --ks-key-alias key3 --ks-pass pass:zbt123456 --key-pass pass:zbt123456 --v1-signing-enabled false --v3-signing-enabled false --in /Users/zoubaitao/Desktop/app-debug-unsigned.apk --out /Users/zoubaitao/Desktop/app-signed2.apk
```


### 签名轮替
V3.1签名

```bash
./apksigner rotate --out /Users/zoubaitao/Desktop/lineage --old-signer --ks-key-alias key0 --ks-pass pass:zbt123456 --key-pass pass:zbt123456 --ks /Users/zoubaitao/Desktop/keystore  --new-signer --ks-key-alias key3 --ks-pass pass:zbt123456 --key-pass pass:zbt123456 --ks /Users/zoubaitao/Desktop/keystore2
```

```bash
./apksigner sign --ks-key-alias key0  --ks-pass pass:zbt123456 --key-pass pass:zbt123456 --ks /Users/zoubaitao/Desktop/keystore --next-signer --ks-key-alias key3 --ks-pass pass:zbt123456 --key-pass pass:zbt123456 --ks /Users/zoubaitao/Desktop/keystore2 --lineage /Users/zoubaitao/Desktop/lineage --in /Users/zoubaitao/Desktop/app-debug-unsigned.apk --out /Users/zoubaitao/Desktop/rotate-v3-signed.apk 
```

我们来看一下经过轮替签名后的结果
```bash
```bash
zoubaitao@192 35.0.0 % ./apksigner verify -verbose --print-certs /Users/zoubaitao/Desktop/rotate-v3-signed.apk 
Verifies
Verified using v1 scheme (JAR signing): true
Verified using v2 scheme (APK Signature Scheme v2): true
Verified using v3 scheme (APK Signature Scheme v3): true
Verified using v3.1 scheme (APK Signature Scheme v3.1): true
Verified using v4 scheme (APK Signature Scheme v4): false
Verified for SourceStamp: false
Number of signers: 1
Signer (minSdkVersion=33, maxSdkVersion=2147483647) certificate DN: C=518101, ST=Guangdong, L=Shenzhen, O=vivo, OU=Application, CN=Zou Baitao
Signer (minSdkVersion=33, maxSdkVersion=2147483647) certificate SHA-256 digest: d1e5b04958d31cd5c42e55fd0be9372a8ae2e5fa7713a2746c35e659d39e4102
Signer (minSdkVersion=33, maxSdkVersion=2147483647) certificate SHA-1 digest: d24a7d69a1d3e70ba86ee3a6186e70541685c52a
Signer (minSdkVersion=33, maxSdkVersion=2147483647) certificate MD5 digest: 144ef5c8fea5b8819e36e6123709ab52
Signer (minSdkVersion=33, maxSdkVersion=2147483647) key algorithm: RSA
Signer (minSdkVersion=33, maxSdkVersion=2147483647) key size (bits): 2048
Signer (minSdkVersion=33, maxSdkVersion=2147483647) public key SHA-256 digest: 92dfca1fbb8471c4561e6b920843e534d4312f205e7ab63dbce30decd82b2519
Signer (minSdkVersion=33, maxSdkVersion=2147483647) public key SHA-1 digest: 696af662e16fbcf4000b5f352385e2ce36ac74ea
Signer (minSdkVersion=33, maxSdkVersion=2147483647) public key MD5 digest: 2a788a5ebbd1327d412f82f8b9442ba4
Signer (minSdkVersion=24, maxSdkVersion=32) certificate DN: C=518101, ST=Guangdong, L=Shenzhen, O=vivo, OU=Application, CN=Zou Baitao
Signer (minSdkVersion=24, maxSdkVersion=32) certificate SHA-256 digest: 9ef58a4abab0dd6e4d4f559b83de3994dba62d85f192ac375bc00f19c4aa1467
Signer (minSdkVersion=24, maxSdkVersion=32) certificate SHA-1 digest: 07cfe5f15f5c8c1fcc9c953a4aa1d96693fef4fb
Signer (minSdkVersion=24, maxSdkVersion=32) certificate MD5 digest: 8541470bad8040192e4c19a89eac52f6
Signer (minSdkVersion=24, maxSdkVersion=32) key algorithm: RSA
Signer (minSdkVersion=24, maxSdkVersion=32) key size (bits): 2048
Signer (minSdkVersion=24, maxSdkVersion=32) public key SHA-256 digest: d520f5400a50c5572e3e240f3faafef4cfb45c3516964f664f69f30bf92ee9af
```

### 签名获取
