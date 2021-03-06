# 如果遇到问题，可以去原作者大佬的博客查看评论区找解决办法：
https://blog.kuoruan.com/116.html
# 系统要求
已测试通过的系统： Ubuntu 14.04 x64、Ubuntu 16.04 x64、CentOS 6 x64、CentOS 7 x64 只支持 64 位系统，要求 glibc 版本 2.14 以上。
# 需要配置的有如下几个选项：
1.需要加速的端口，即的 SS 端口。加速开启之后，流量会先经过 BBR 处理，之后再发送给后端的 SS。    
2.可能需要配置 “公网接口名称”，即你服务器上具有公网 IP 的接口名称。搬瓦工 OpenVZ 上默认都是 venet0，但是有朋友可能需要安装在其他服务器上，所以我加入了此选项。   
3.需要注意的是，在有 firewalld 的服务器上安装的时候，firewalld 会干扰 iptables 的规则，造成网络不通（现在具体原因未知，谁有解决方案可以提示一下）。所以在装有 firewalld 的服务器上需要先退出 firewalld：    
systemctl disable firewalld    
systemctl stop firewalld
# 安装
wget --no-check-certificate https://raw.githubusercontent.com/Ache1123/shell-scripts/master/ovz-bbr/ovz-bbr-installer.sh && chmod +x ovz-bbr-installer.sh && bash ovz-bbr-installer.sh
# 更新 glibc（我更建议你重装系统，换个高版本的系统，如果更新glibc搞不定）
# CentOS 6 更新 glibc，首先下载如下几个文件：
wget http://ftp.redsleeve.org/pub/steam/glibc-2.15-60.el6.x86_64.rpm \
http://ftp.redsleeve.org/pub/steam/glibc-common-2.15-60.el6.x86_64.rpm \
http://ftp.redsleeve.org/pub/steam/glibc-devel-2.15-60.el6.x86_64.rpm \
http://ftp.redsleeve.org/pub/steam/glibc-headers-2.15-60.el6.x86_64.rpm \
http://ftp.redsleeve.org/pub/steam/nscd-2.15-60.el6.x86_64.rpm
# 然后安装：
# rpm -Uvh glibc-2.15-60.el6.x86_64.rpm \
glibc-common-2.15-60.el6.x86_64.rpm \
glibc-devel-2.15-60.el6.x86_64.rpm \
glibc-headers-2.15-60.el6.x86_64.rpm \
nscd-2.15-60.el6.x86_64.rpm
# 如果以上步骤无法更新，可以手动编译更新（来自网友的方法）
wget http://ftp.gnu.org/gnu/glibc/glibc-2.15.tar.gz
wget http://ftp.gnu.org/gnu/glibc/glibc-ports-2.15.tar.gz
tar -zxf glibc-2.15.tar.gz
tar -zxf glibc-ports-2.15.tar.gz
mv glibc-ports-2.15 glibc-2.15/ports
mkdir glibc-build-2.15
cd glibc-build-2.15
../glibc-2.15/configure --prefix=/usr --disable-profile --enable-add-ons --with-headers=/usr/include --with-binutils=/usr/bin
make all && make install
# 检查一下：
# ldd --version
ldd (GNU libc) 2.15
Copyright (C) 2012 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
Written by Roland McGrath and Ulrich Drepper.
# 已经升级到 glibc 2.15 了。
# 测试是否开启成功
ping 10.0.0.2
# 卸载	
./ovz-bbr-installer.sh uninstall
# 多端口加速
vi /usr/local/haproxy-lkl/etc/port-rules    
> 8800 或者给出范围 8800-8810
# 多端口配置后重启haproxy
systemctl restart haproxy-lkl   
service haproxy-lkl restart
# 错误说明
有些机器一切正常，但是加速失败。从网友的反馈来看，可能需要将 SS 的监听地址从 vps IP 改到 127.0.0.1 或者 0.0.0.0，具体未测试，加速失败的朋友可以试一试
