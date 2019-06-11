# HeidiSQL SSH 链接 mysql 配置

## 1. 秘钥转换：SecureCRT to PuTTY
* SecureCRT : Tools -> Convert Private Key to OpenSSH Format...
* puttygen.exe : Conversions -> Import key -> Save private key

## 2. HeidiSQL配置：
* 设置 -> 网络类型： MySQL (SSH tunnel)
* SSH隧道 -> plink.exe 位置 ：putty安装目录\PuTTY\plink.exe
* SSH隧道 -> 秘钥文件： 即步骤1中转换的.ppk文件