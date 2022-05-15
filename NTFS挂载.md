## 理论
Mac本身是支持NTFS写入的，只是NTFS是微软开发，由于版权和一些技术细节原因，苹果不愿公开说自己支持NTFS写入，也是有自己以后可能不支持NTFS写入的考量

## 操作流程
挂载上你的NTFS硬盘，查看硬盘名称
编辑/etc/fstab文件，使其支持NTFS写入
将/Volumes中的NTFS磁盘快捷方式到Finder
## 详细流程
* 插上硬盘后，查看你的硬盘名称，这里假设名称是AngleDisk
打开Applications的Terminal, 你也可以直接spotlight输入terminal打开
* 在终端输入sudo nano /etc/fstab 敲击回车
现在你看到了一个编辑界面，输入LABEL=AngleDisk none ntfs rw,auto,nobrowse后，敲击回车，再Ctrl+X，再敲击Y，再敲击回车
* 此时，退出你的移动硬盘，再重新插入，你会发现磁盘没有显示在桌面或是Finder之前出现的地方，别慌
* 打开Finder，Command+Shift+G，输入框中输入/Volumes，回车，你就可以看到你的磁盘啦！是可以读写的哟，Enjoy
* 方便起见，你可以直接把磁盘拖到Finder侧边栏中，这样下次使用就不用进入到/Volumes目录打开le
## 参考
https://www.zhihu.com/question/19571334