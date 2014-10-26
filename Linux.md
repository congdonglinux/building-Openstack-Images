##1. Tạo máy ảo và cài đặt OS##
- Để tạo được máy ảo thì các bạn phải cài đặt libvirt và virt-manager, các bạn có thể tìm các hướng dẫn trên mạng và làm theo, bài viết này mình sẽ bỏ qua phần đó.

Sau khi cài đặt virt-manager xong, các bạn khởi động virt-manager và có giao diện (gần) giống như hình dưới:
http://i.imgur.com/cItI7CJ.png
1.1 Tạo máy ảo
File => New Virtual Machine => Local Install media (ISO image or CDROM)
http://i.imgur.com/LVjAqmN.png
=> Use ISO image => Chỉ đến file ISO của CentOS (mình chọn CentOS-6.5-x86_64-minimal.iso)
http://i.imgur.com/kaOlB4p.png

=> Điều chỉnh thông số  như RAM, CPU, HDD  cho phù hợp. Ở đây mình chọn 2048MB RAM, 2CPU và 2GB HDD
http://i.imgur.com/jYBugns.png

=> Đặt lại tên VM cho dễ nhớ và check vào Customize configuration before install''. Sau đó nó sẽ hiện ra 1 cái (gần) giống như thế này
http://i.imgur.com/h9Lrjmx.png

=> Chọn ‘Disk 1’ => Advanced Options => Chọn ‘VirtIO’ tại mục Disk bus => Apply
http://i.imgur.com/eny9YbQ.png
=> Chọn ‘NIC :xx:x:xx’ => Chọn ‘virtio’ tại mục Device Model => Apply
http://i.imgur.com/Bl2DScC.png
=> Begin Installation để bắt đầu quá trình cài đặt. Bạn tiến hành cài đặt như cài server bình thường (Đừng hỏi mình các bước cài đặt OS lên server nhé vì mình ko biết :( ). Có 1 lưu ý khi cài đặt OS đó là lúc chia phân vùng đĩa cứng các bạn sử dụng tòan bộ ổ đĩa cho 'thư mục' ‘/’ và không tạo SWAP Space (Để tối ưu hóa quá trình resize partition lúc tạo máy ảo)

##2. Cài đặt các phần mềm cần thiết và update hệ thống##

2.1. Cấu hình card eth0 tự động active khi hệ thống boot-up
vi /etc/sysconfig/network-script/ifcfg-eth0
ONBOOT=yes

Xóa 2 dòng :
HWADDR=xx:xx:xx:xx:xx:xx
UUID=.....

# Active network interface
ifup eth0


2.2 Cài đặt, cấu hình các  phần mềm cần thiết 
Ở đây mình sẽ cài thêm 1 số phần mềm cần thiết, các bạn có thể cài đặt bất cứ gì nhưng đừng cài nhiều quá vì các bạn chỉ có 2GB HDD thôi (nếu muôn cài nhiều thì lúc tạo máy ảo, bạn chọn HDD nhiều hơn 1 chút).
yum install vim openssh-clients rsync -y
yum update -y 

# Cài đặt cloud-utils-growpart để resize đĩa cứng lần đầu boot
rpm -Uvh http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
yum install cloud-utils-growpart dracut-modules-growroot cloud-init -y

Bởi vì cloud-utils-growpart chỉ tới kernel 3.8 thì mới hỗ trợ update partition tables của đĩa cứng sau khi đã mount nên nếu chỉ cài đặt cloud-utils-growpart không thì đĩa cứng máy ảo sẽ không được tự động reszie (vì CentOS sử dụng kernel 2.6). Muốn tự động resize thì ta phải thực hiện trước khi kernel được load. Mình sẽ sử dụng initrd để làm việc này. 

# Rebuild initrd file
dracut -f

# Kiểm tra sau khi rebuild
lsinitrd | grep grow
-rwxr-xr-x   1 root     root          133 Nov 22  2013 cmdline/99growroot-dummy.sh
-rwxr-xr-x   1 root     root         2167 Nov 22  2013 pre-mount/99growroot.sh
-rwxr-xr-x   1 root     root        16069 Nov 22  2013 usr/bin/growpart

# Cấu hình grub để  ‘phun’ log ra cho nova (Output của lệnh : nova get-console-output)
vim /boot/grub/grub.conf
Thay phần ‘rhgb quiet’
Bằng : "console=tty0 console=ttyS0,115200n8"

# Cấu hình cloud-init
vim /etc/cloud/cloud.cfg
disable_root: 0
ssh_pwauth:   1

# Cleaning and Poweroff
yum clean all
poweroff


##3 Xóa thông tin ‘phần cứng’##
cd /var/lib/libvirt/images/
sudo virt-sysprep -a centos6.5.qcow2

# Reduce image size
sudo virt-sparsify --compress centos6.5.qcow2 centos6.5.cloud.qcow2

##4 Upload lên glance##
Qúa trình tạo template đã xong, bạn upload file centos6.5.cloud.qcow2 lên Openstack là có thể sử dụng được.
