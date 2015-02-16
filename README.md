1 vấn đề phải nghĩ đến sau khi đã setup 1 hệ thông Openstack là phải có template (glance images) để quá trình tạo máy ảo diễn ra nhanh hơn. Lướt qua 1 vòng google thì mình thấy Canonical có hẳn 1 ‘kho’ cung cấp [images cho các hệ thống Cloud](https://cloud-images.ubuntu.com/) (trong đó có Openstack), ngoài ra còn có [fedora](https://fedoraproject.org/en/get-fedora#clouds) và [CentOS 7](http://cloud.centos.org/centos/7/devel/). Mình không tìm thấy CentOS 6 và Windows (ngoại trừ bản Windows Server 2k12 của Cloudbase). Vì không tìm thấy những thứ mình cần nên mình cũng mày mò tìm hiểu cách build và có dc 1 vài dòng notes lại để chia sẻ với mọi người, đỡ mất thời gian tìm kiểm (để thời gian đó tìm kiếm cái khác rồi share lại cho mình nhé :v ).



#####Bài này mình sẽ chia ra làm 2 phần:

[Phần 1: Hướng dẫn cách tạo images cho Linux (CentOS 6.5)](https://github.com/d0m0reg00dthing/building-Openstack-Images/blob/master/Linux.md)

[Phần 2: Hướng dẫn cách tạo images cho Windows (Windows Server 2008 R2 Enterprise)](https://github.com/vietstacker/building-Openstack-Images/blob/master/Windows.md)

#####Mỗi phần sẽ bao gồm các bước sau:
- Tạo máy ảo và cặt đặt OS sử dụng libvirt và virt-manager để quản lý bằng GUI
- Cài đặt các phần mềm cần thiết, update hệ thống, cài đặt cloud-init
- Xóa thông tin ‘phần cứng’ như Hard Disk UUID, MAC Address.
- Nén file ảnh sau khi cài đặt
- Upload lên glance
