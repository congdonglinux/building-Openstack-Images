Bài này mình sẽ nói sơ qua về cách thức tạo template Windows cho Openstack. Trên mạng hiện nay có 1 số tool tự động hóa quá trình tạo template cho Openstack Packer (https://www.packer.io/) hoặc https://github.com/openstack/diskimage-builder. Nhiều bạn sẽ thắc mắc tại sao có nhiều tool như vậy mà mình không xài mà đi làm từng bước, tại sao phải thiết kế lại các bánh xe ?. Xin trả lời câu hỏi này là mình thích tìm hiểu cách thức tạo ra 1 sản phẩm hơn là tìm 1 công cụ giúp tạo ra sản phẩm nhanh nhất (đương nhiên tìm hiểu để biết thêm thôi chứ trong thực tế mình sẽ chọn 1 tool để làm vì nó khỏe cho mình). Okie, lan man bi nhiêu đủ rồi, vào nội dung luôn.

Tạo template Windows về cơ bản là giống như template Linux, nhưng có 1 điểm khác biệt là Windows nó ko có driver hỗ trợ VirtIO nên cài Windows trên máy ảo sử dụng hard disk driver là VirtIO thì sẽ không thấy ổ cứng. Để giải quyết vấn đề này thì có 2 giải pháp:
- 1 là bạn sử dụng chuẩn giao tiếp đĩa cứng khác như IDE, SCSI, SATA 
- 2 là cài đặt driver VirtIO trước khi cài Windows. Bài này mình sẽ làm theo cách này.

##Chuẩn  bị##
- 1 máy tính đã cài đặt KVM/QEMU và Virt-manager
- 1 file ISO Windows bất kỳ (các bạn nên chọn từ Windows Vista trở lên, lý do vì sao sẽ có trong bài). Mình chọn Windows Server 2008 R2 x64.
- VirtIO Driver (Tải về tại: https://alt.fedoraproject.org/pub/alt/virtio-win/latest/images/virtio-win-0.1-100.iso)

#####Tạo máy ảo và cài đặt#####
![Imgur](http://i.imgur.com/pGaEP5s.png)

#####Thêm 1 CD-Rom chứa virtIO Driver cho Windows#####
![Imgur](http://i.imgur.com/yoa7NIY.png)

#####Khi cài Windows các bạn sẽ gặp lỗi không tìm thấy ổ cứng như thế này#####
![Imgur](http://i.imgur.com/0Sa3bRK.png)

#####Các bạn chọn ‘Load Driver’ và chỉ đường dẫn vào thư mục như hình#####
![Imgur](http://i.imgur.com/8Un3Rp8.png)

#####Chọn tất các các driver mà Windows tìm được#####
![Imgur](http://i.imgur.com/YEUuZjh.png)

#####Sau khi cài đặt Driver, Windows sẽ nhận được HDD. Từ đây các bạn có thể cài Windows như bình thường.#####
![Imgur](http://i.imgur.com/FTFMr4g.png)

Sau khi cài đặt xong, các bạn sẽ boot vào Windows, cài đặt phần mềm tùy theo nhu cầu. Cuối cùng, bạn [download file xml mình chuẩn bị sẵn](https://github.com/vietstacker/building-Openstack-Images/blob/master/Answer%20files/Windows/2008_R2.xml), mục đích của file này là:
- Add key cho Windows
- Enable RDP
- Thêm 2 rule cho phép ICMP và RDP Input
- Resize OS Partition

Nếu bạn nào muốn tùy biến thêm thì có thể tham khảo thêm trên Google với từ khóa ‘Windows Unattended files’. Sau khi tải về các bạn lưu file tại “C:\unattend.xml”, mở Command Prompt và gõ lệnh: 
C:\Windows\System32\sysprep\sysprep.exe /generalize /oobe /shutdown /unattend:C:\unattend.xml
![Imgur](http://i.imgur.com/jWvijih.png)

Bước cuối cùng là upload file image của máy ảo này lên Openstack.

Lưu ý:
- Serial trong file xml là mình sử dụng cho bản Windows Server 2008 R2 Enterprise, nếu các bạn sử dụng không đúng key thì quá trình boot máy ảo sẽ gặp vấn đề và các bạn phải tạo lại từ đầu (Không hiểu sao Windows lại ‘kém thông minh’ đến vậy)

