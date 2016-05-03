#1.这样获取手机所有挂载的存储器？
Android是没有提供显式的接口的，首先肯定是要阅读系统设置应用“存储”部分的源码，看存储那里是通过什么方式获取的。最后找到StorageManager和StorageVolume这2个重要的类，然后通过反射获取StorageVolume[]列表。
#2.用什么标示一个存储器的唯一性？
存储路径？不行（有些手机不插TF卡，内置存储路径是/storage/sdcard0，插上TF卡后，内置存储路径变成/storage/sdcard1，TF卡变成/storage/sdcard0）。
存储卡名称？不行（可能会切换系统语言，导致名称匹配失败，名称的resId也不行，较低的系统版本StorageVolume没有mDescriptionId这一属性）。
经过测试，发现使用mStorageId可以标示存储器的唯一性，存储器数量改变，每个存储器的id不会改变。
#3.如何获得存储器的名称？
经测试，不同的手机主要有3种获取存储器名换的方法：getDescription()、getDescription(Context context)、先获得getDescriptionId()再通过resId获取名称。
#4.任务文件下载一半时，切换文件保存存储器，怎么处理？
有2种方案：
4.1 切换时，如果新的存储空间足够所有文件转移，先停止所有下载任务，将所有下载完和下载中的文件拷贝到新的存储空间，然后再更新下载数据库下载任务的存储路径，再恢复下载任务；
4.2 切换时，先拷贝所有下载完成的文件到新的存储空间，下载任务继续下载，下载完成再移动到新的存储空间。
#5.在4.4系统上，第三方应用无法读取外置存储卡的问题。（参考“ External Storage ”）
google为了在程序卸载时，能够完全彻底的将程序所有数据清理干净，应用将不能向2级存储区域写入文件。
“The WRITE_EXTERNAL_STORAGE permission must only grant write access to the primary external storage on a device. Apps must not be allowed to write to secondary external storage devices, except in their package-specific directories as allowed by synthesized permissions. Restricting writes in this way ensures the system can clean up files when applications are uninstalled.”
要能够在4.4系统上TF卡写入文件，必须先root，具体方法可以google。
所以4.4系统上，切换会导致文件转移和下载失败，用户如果要切换到TF卡，至少需要提醒用户，并最好给出4.4上root解决方法。
