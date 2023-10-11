# Jump Server v3.0 Upgrade Guide

## 1. Những lưu ý trước khi nâng cấp

Có một số khác biệt nhất định giữa JumpServer v3.0 và JumpServer v2.x.

JumpServer v3.0 thêm một số tính năng mới, xóa một số tính năng và cấu trúc lại một số tính năng.

### Các tính năng mới

- **[Console]** Dashboard thêm màn hình biểu đồ "Asset Type Proportion - Tỷ lệ loại tài sản", chủ yếu hiển thị dữ liệu tỷ lệ của các loại tài sản khác nhau (linux, windows,...)

- **[Console]** Thêm "Account template - mẫu tài khoản" trong Accounts. Khi mật khẩu asset giống nhau, có thể trực tiếp chọn mẫu tài khoản để giảm khối lượng công việc điền tay.

- **[Console]** Đã thêm mô-đun chức năng "Command Filtering - lọc lệnh" trong Permissions

- **[Audit Desk]** function to the audit log. Đã thêm chức năng "Resource Activity Log - Nhật ký hoạt động tài nguyên" vào audit log

- **[Workbench]** Thêm mô-đun chức năng "Job Center - Trung tâm công việc", điều chỉnh "batch command - lệnh hàng loạt" trong phiên bản JumpServer v2. * thành chức năng lệnh tắt. 
    
- **[Workbench]** Thêm các chức năng như "job management - quản lý công việc" và "template management - quản lý mẫu". Hỗ trợ người dùng thực hiện các thao tác vận hành và bảo trì bằng cách viết một số tập lệnh Ansible.

- ...
### Các tính năng đã bị xóa

- **[Console]**** Xóa mô-đun chức năng "system user - người dùng hệ thống" trong asset management và liên kết trực tiếp assets với tài khoản.

- **[Console]** Xóa chức năng "command filtering" trong asset management.

- **[Console]** Xóa mô-đun chức năng "Access Control - Kiểm soát truy cập"

- ...

### Tái cấu trúc chức năng

- **[Console]** Khi ủy quyền nội dung trong quản lý quyền, system user không còn được chọn nữa và thay đổi thành chọn tài khoản sẽ được sử dụng hoặc tài khoản được nhập thủ công.

- **[Console]** Role list trong user management được chuyển sang phiên bản doanh nghiệp và phiên bản cộng đồng không còn được hỗ trợ.

- **[Console]** Trong account management, "collect users - thu thập người dùng" được đổi thành "account collection - thu thập tài khoản". Lưu ý rằng khi JumpServer v2.* nâng cấp JumpServer v3.0, task thu thập người dùng và dữ liệu tài khoản tài sản đã thu thập trước đó sẽ không được giữ lại, và cần phải được thêm lại.

- **[Audit console]** Trang tổng quan được thiết kế lại và điều chỉnh để hiển thị "display log overview data - dữ liệu tổng quan về nhật ký", "session overview data - dữ liệu tổng quan về phiên", "command overview data- dữ liệu tổng quan về lệnh", "login overview data - dữ liệu tổng quan về đăng nhập" và các thông tin khác.

- **【System Management】Xác thực JumpServer v3.0 phiên bản cộng đồng chỉ hỗ trợ LDAP và CAS. Nếu các phương thức xác thực khác đã được định cấu hình trước đó, chúng vẫn có thể được sử dụng sau khi nâng cấp nhưng không thể sửa đổi chúng được nữa.**

## 2. Chuẩn bị trước khi nâng cấp

1. JumpServer v3.0 loại bỏ system users và liên kết trực tiếp nội dung với accounts. Do đó, trước khi nâng cấp, cần đảm bảo rằng chỉ có một tài khoản có cùng user name cho mỗi asset. Nếu có nhiều tài khoản có cùng tên dưới asset trong quá trình nâng cấp, và có nhiều người dùng root, người dùng root cuối cùng sẽ được tự động hợp nhất sau khi nâng cấp và sẽ không được đồng bộ hóa với những người dùng root khác. Bạn nên sắp xếp các quy tắc ủy quyền trước khi nâng cấp.

2. Vì JumpServer v3.0 hợp nhất các nội dung và ứng dụng của các phiên bản V2.*. Do đó, trước khi nâng cấp, cần đảm bảo rằng tên của database assets và host assets không bị trùng lặp. Nếu xuất hiện trùng lặp trong quá trình nâng cấp, thông tin của database assets có thể bị mất sau khi nâng cấp V3.0. Do đó, bạn nên phân loại và chắc chắn rằng tên của database assets và host assets không nhất quán trước khi nâng cấp.

3. Lưu dữ liệu. Khi JumpServer v2.* nâng cấp lên JumpServer v3.0, gói mã hóa, người dùng đã thu thập và remote application sẽ không được migrated. Vui lòng lưu dữ liệu trước và sau đó định cấu hình sau khi nâng cấp thành công.

4. Xác nhận chức năng, phiên bản JumpServer v3.0 loại bỏ một số chức năng, chẳng hạn như dynamic user, push user người dùng sử dụng thư mục chính, group, Shell, Sudo information, etc. v.v. không còn được hỗ trợ và xác nhận xem các chức năng khác có thể đáp ứng yêu cầu hay không.

5. Đối với công việc sao lưu, vui lòng sao lưu trước, sao lưu database của phiên bản JumpServer v2.*, tham khảo chương sao lưu database trong tài liệu này: https://docs.jumpserver.org/zh/v2/install/upgrade/mariadb-mysql

## 3. Quá trình nâng cấp

1. Để nâng cấp kiến ​​trúc single node, vui lòng tham khảo: https://docs.jumpserver.org/zh/v3/jms_install/linux_stand-alone/off_line_upgrade/

2. Đối với kiến ​​trúc cluster, vui lòng dừng từng node ứng dụng trước và nâng cấp từng node một.

3. Đối với các kiến ​​trúc khác, vui lòng tự nâng cấp. Lưu ý rằng quá trình nâng cấp liên quan đến các thay đổi trong cấu trúc bảng cơ sở dữ liệu và cần dừng dịch vụ trước.

4. Vui lòng tham khảo: https://kb.fit2cloud.com/?p=4ba65333-bf41-42f7-b329-afc855e7789a **nếu nâng cấp không thành công** và quay lại phiên bản cũ

## 4. Test verification sau khi nâng cấp

1. Kiểm tra xem dữ liệu asset có bình thường không.

2. Kiểm tra xem dữ liệu authorization có bình thường không.

3. Kiểm tra xem dữ liệu account có bình thường không.

4. Kiểm tra xem kết nối asset có bình thường không.

5. Nếu bạn sử dụng phiên bản cũ để thay đổi  encryption plan, collect users, and remote applications (JumpServer v3.0 hiện chỉ hỗ trợ Chrome, DBeaver và Navicat), vui lòng cấu hình lại.