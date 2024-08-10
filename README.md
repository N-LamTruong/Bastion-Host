# Bastion Host - Jump Server v2
## Mục tiêu
1. Setup Jump Server trên instance Bastion Host
2. Setup xác thực MFA, user instance, assets instance, assets permissions, web terminal
3. Tạo user Jump Server, phân quyền truy cập (roles, user login instance,...)
4. Quản lý nâng cao (Command Filters, Sessions audit, Batch Command)
5. Triển khai NGINX Reverse Proxy 2 lớp
6. Bảo mật nâng cao: kết nối SSL cho container Database, Redis

Chi tiết xem tại trang chủ: **[here](https://docs.jumpserver.org/zh/v2/)**
## Yêu cầu môi trường
- Linux (Trong bài hướng dẫn này sử dụng Ubuntu 24.04 LTS)
- wget curl tar gettext iptables python
- Phần cứng tối thiểu: 2Core/8GB RAM/60G HDD
## Mô hình
![Architect](Picture/Workflow.png)

### 1. Setup Jump Server và tạo SSH Key
- **Step 1**: Download Jump Server từ file nén
```console
sudo apt-get update
sudo apt-get install -y wget curl tar gettext iptables
cd /opt
wget https://github.com/jumpserver/installer/releases/download/v2.28.8/jumpserver-installer-v2.28.8.tar.gz
tar -xf jumpserver-installer-v2.28.8.tar.gz
cd jumpserver-installer-v2.28.8
```
- **Step 2**: Setup Jump Server
    
    Sửa đổi file config nếu cần thiết. Nếu không biết rõ mục đích, có thể bỏ qua!
    ```console
    cat config-example.txt
    ```
    Cài đặt và khởi động
    ```console
    ./jmsctl.sh install
    ./jmsctl.sh start
    ```
    Sau khi cài đặt xong, file config tại: /opt/jumpserver/config
    ```console
    config.txt  core  koko  mariadb  mysql  nginx  redis
    ```
    Các lệnh vận hành được thao tác tại **/opt/jumpserver-installer-v2.28.8** (chi tiết các lệnh vận hành sử dụng **./jmsctl.sh --help**)
    ```console
    ./jmsctl.sh start
    ./jmsctl.sh stop
    ./jmsctl.sh down
    ./jmsctl.sh restart
    ./jmsctl.sh backup
    ./jmsctl.sh upgrade
    ./jmsctl.sh uninstall
    ...
    ```
    

- **Step 3**: Setup SSH Key (nếu đã có ssh key ở máy cá nhân thì bỏ qua bước này)
    
    Có nhiều cách, nhiều phần mềm để tạo SSH Key, trong bài toán này mình sẽ sử dụng trực tiếp trên server (lưu ý làm cách nào để sau bạn có thể lấy cả file private key và public key ra ngoài máy cá nhân được)

    Tạo SSH Key:
    ```console
    sudo ssh-keygen -t rsa
    ```
    Copy public key sang các instance cần ssh
    ```console
    ssh-copy-id user@IP
    ```
    Lưu file private key về máy tính cá nhân
### 2. Setup truy cập căn bản
![Mo hinh](Picture/Mo%20hinh.png)

- **Step 1**: Bật xác thực MFA

    ![MFA](Picture/Authentication%20MFA.jpg)

    Logout ra và sử dụng phần mềm xác thực bảo mật 2 lớp (Google Authenticator, Authy, Duo Security, Azure Multi-Factor Authentication, Lastpass Authenticator,...)

- **Step 2**: Tạo user login vào các instance

    ![user instance 1](Picture/User%20instance%201.png)

    Nhập các thông tin cần thiết như:

    Basic: Tên, user login instance
    
    Authorization: Dùng password hoặc dùng SSH Key (trong hướng dẫn này sẽ sử dụng SSH Key để bảo mật hơn)
    
    **Chú ý**: Lấy file private key đã lưu ở phần 1

    JumpServer chỉ nhận đúng định dạng PEM của private key (BEGIN RSA PRIVATE KEY ... END RSA PRIVATE KEY). Nếu bạn tạo SSH Key trực tiếp trên server thì định dạng mặc định là PEM. Nhưng nếu bạn dùng các phần mềm khác để tạo thì cần chú ý định dạng ví dụ PuTTYgen thì private key có định dạng PPK cần phải convert về PEM thì mới import vào JumpServer thành công.
- **Step 3**: Create node, instance assets

    Tạo các node theo mục đích sử dụng (phòng ban, roles,...)
    
    ![create node](Picture/Create%20node.png)
    
    Tạo các instance để truy cập và gán chúng vào các node tương ứng. Nhập các thông tin cần thiết như: "Basic, Auth, Node: Tên instance, IP,..."

    ![instance assets](Picture/Create%20instance.png)
    
- **Step 4**: Tạo các asset permissions để truy cập bằng web terminal

    Điền các thông tin cần thiết như: Basic, User, Asset (Tên permission, user Jump Server, node, user instance,...)

    ![Asset Permissions](Picture/Asset%20permissions.png)

    Test truy cập bằng Web Terminal

    ![Web Terminal](Picture/Web%20Terminal.png)

    ![Test connect](Picture/Test%20connect%20web%20terminal.png)

### 3. Tạo user Jump Server và cấp quyền truy cập tương ứng
- **Step 1**: Tạo user SOC và gán role user

    ![Create SOC user](Picture/Create%20user%20Jump%20Server.png)

    ![Info user SOC](Picture/Info%20user%20Jump%20Server.png)

    Ảnh dưới là các quyền thao tác của **roles user**, các **roles mặc định không thể chỉnh sửa**. Nếu muốn tự custom roles bạn có thể tự tạo và tùy chỉnh theo mong muốn

    ![Roles user](Picture/Role%20defau%20for%20user.png)

- **Step 2**: Phân quyền truy cập cho user SOC (theo node instance, user login instance)

    ![permission user soc](Picture/Permission%20for%20user%20soc.png)

    Như vậy bạn có thể test truy cập bằng web terminal với **user Jump Server là soc** -> Chỉ được phép login vào **instance SOC** -> Với **user login instance là soc**

**=> Chú ý**: Hướng dẫn trên chỉ demo cách thức tạo user và phân quyền đơn giản:
- Có thể cho user truy cập thêm các node instance khác với các user login được chỉ định rõ ràng
- Cân nhắc nếu instance không có nhiều user thì có thể cấp phép cho login bằng user có sẵn
- Giới hạn các thao tác ngay trong phần tạo permission
- ...

### 4. Sử dụng Command Filters, Sessions audit, Batch Command

**Command Filters**

- Sử dụng để giới hạn quyền truy cập của người dùng vào các lệnh cụ thể trên các instance

- Điều này giúp ngăn chặn người dùng không có quyền truy cập vào các lệnh quan trọng, nhằm giảm thiểu các rủi ro bảo mật và đảm bảo tính an toàn cho hệ thống

VD: reboot, rm, systemctl enable, systemctl disable, ...

![Command 1](Picture/Command%20Filters%201.png)

Sau khi tạo Command Filters, click vào tên Command Filters để tạo rules

![Command 2](Picture/Command%20Filters%202.png)

Test reboot

![Command 3](Picture/Command%20Filters%203.png)

**Sessions audit**

- Quản lý và giám sát truy cập, giúp quản trị viên ghi lại hoạt động của người dùng khi họ truy cập và thực hiện các hoạt động trên hệ thống. Sessions audit có thể được sử dụng để giám sát và phát hiện các hành vi đáng ngờ hoặc bất thường, cũng như giúp quản trị viên giải quyết các vấn đề bảo mật
- Trong **sessions audit** 2 tính năng thường được sử dụng là: **Session list** và **Commands**

    Session list:
    - Tab Sessions online: Nếu bạn có đủ quyền hạn thì bạn có thể kết thúc phiên làm việc của các user khác (Terminate), xem trực tiếp họ đang thao tác bằng cách chọn (Monitor)
    - Tab Sessions offline: Danh sách toàn bộ các phiên đã truy cập vào hệ thống server của bạn. Bạn có thể xem và tải về
    
    ![Session list](Picture/Session%20list.png)

    Commands

    ![Commands](Picture/Commands.png)

**Batch Command**

- Cho phép người dùng thực hiện một chuỗi lệnh trên nhiều thiết bị mạng cùng một lúc

    ![Batch Command](Picture/Batch%20Command.png)

### 5. Triển khai NGINX Reverse Proxy 2 lớp
Ban đầu **SSL mặc định không sử dụng**. Trong bài toán này mình sẽ thiết lập SSL sử dụng Certbot Let's Encrypt. Bản chất khi triển khai JumpServer đã chạy qua container nginx, nhưng chúng ta sẽ cài đặt thêm lớp nginx thứ 2 trực tiếp trên server để làm **Multi-layer nginx reverse proxy**

**Chú ý**: Do container nginx web đang sử dụng port 80. Chúng ta sẽ thay đổi cấu hình container sang port khác để nginx server bên ngoài về sau sẽ dùng được port 80 ([Vì Certbot mặc định lấy port 80/443 để cấp SSL](https://serverfault.com/questions/1084029/how-do-i-specify-a-port-other-than-80-when-adding-ssl-certificate-using-certbot)). Stop JumpServer và thay đổi config tại **/opt/jumpserver/config/config.txt** tìm dòng chứa **HTTP_PORT** sửa thành 81 sau đó Start lại JumpServer (các thao tác vận hành xem lại phần 1)

**Yêu cầu**: Đã có domain và trỏ đến IP của JumpServer
- **Step 1**: Cài đặt nginx bản stable nhất từ [trang chủ nginx](https://nginx.org/en/linux_packages.html#Ubuntu)
    ```console
    sudo apt install curl gnupg2 ca-certificates lsb-release ubuntu-keyring
    ```
    ```console
    curl https://nginx.org/keys/nginx_signing.key | gpg --dearmor \
        | sudo tee /usr/share/keyrings/nginx-archive-keyring.gpg >/dev/null
    ```
    ```console
    gpg --dry-run --quiet --no-keyring --import --import-options import-show /usr/share/keyrings/nginx-archive-keyring.gpg
    ```
    ```console
    echo "deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] \
    http://nginx.org/packages/ubuntu `lsb_release -cs` nginx" \
        | sudo tee /etc/apt/sources.list.d/nginx.list
    ```
    ```console
    echo -e "Package: *\nPin: origin nginx.org\nPin: release o=nginx\nPin-Priority: 900\n" \
        | sudo tee /etc/apt/preferences.d/99nginx
    ```
    ```console
    sudo apt update
    sudo apt install nginx
    ```
- **Step 2**: Tạo file config nginx cho JumpServer và cài đặt Certbot for Let's encrypt SSL
    ```console
    nano /etc/nginx/conf.d/jumpserver.conf
    ```
    Sử dụng IP Private
    
    ```
    server {

        listen 80;
        server_name bastion.lamtruong2001.id.vn;

        client_max_body_size 4096m;

        location / {
                proxy_pass http://172.31.28.12:81;
                proxy_http_version 1.1;
                proxy_buffering off;
                proxy_request_buffering off;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection "upgrade";
                proxy_set_header Host $host;
                proxy_set_header X-Forwarded-For $remote_addr;
        }
    }
    ```
    ```console
    nginx -t
    sudo systemctl restart nginx
    ```
    Cài đặt và sử dụng Certbot

    ```console
    sudo snap install core; sudo snap refresh core
    sudo snap install --classic certbot
    sudo ln -s /snap/bin/certbot /usr/bin/certbot
    sudo certbot --nginx -d example.com -d www.example.com
    ```
    **Chú ý**: Điền đúng domain sử dụng JumpServer của bạn. Sau khi chạy sẽ nhập 1 số thông tin cần thiết và xác nhận Certbot sẽ cấp SSL và tự động cấu hình nginx cho bạn. Đến bước này bạn có thể thử truy cập JumpServer thông qua domain

- **Step 3**: Cấu hình container nginx JumpServer sử dụng cùng một SSL (Multi-layer nginx reverse proxy)

    SSL khi sử dụng Certbot ở bước 2 sẽ lưu ở **/etc/letsencrypt/live/your_domain**
    
    Copy 2 file **fullchain.pem** và **privkey.pem** vào **/opt/jumpserver/config/nginx/cert**
    ```console
    cp fullchain.pem /opt/jumpserver/config/nginx/cert/
    cp privkey.pem /opt/jumpserver/config/nginx/cert/
    ```
    Sau đó cần stop JumpServer để sửa cấu hình SSL
    ```console
    cd /opt/jumpserver-installer-v2.28.8/ && ./jmsctl.sh stop
    nano /opt/jumpserver/config/config.txt
    ```
    Nhắc lại lần nữa thì do nginx bên ngoài đã sử dụng port 80/443 để dùng cho [Certbot Let's Encrypt](https://serverfault.com/questions/1084029/how-do-i-specify-a-port-other-than-80-when-adding-ssl-certificate-using-certbot) nên trong container sẽ đổi port HTTPS
    ```
    HTTPS_PORT=8443
    SERVER_NAME=bastion.lamtruong2001.id.vn
    SSL_CERTIFICATE=/opt/jumpserver/config/nginx/cert/fullchain.pem
    SSL_CERTIFICATE_KEY=/opt/jumpserver/config/nginx/cert/privkey.pem
    ```
- **Step 4**: Cập nhật lại cấu hình nginx bên ngoài để chuyển tiếp yêu cầu đến container nginx JumpServer thông qua HTTPS
    ```console
    nano /etc/nginx/conf.d/jumpserver.conf
    ```
    ```
    proxy_pass https://172.31.28.12:8443;
    ```
    ```console
    nginx -s reload
    ```
    Như vậy là đã xong phần thiết lập NGINX Reverse Proxy 2 lớp cho JumpServer. Cùng mở web browser và trải nghiệm ngay thôi nào!!

### 6. Bảo mật nâng cao: kết nối SSL cho container Database, Redis

Chi tiết mình sẽ update từ trang chủ trong thời gian tới: [here](https://docs.jumpserver.org/zh/v2/install/install_security/)

### Ngoài ra còn rất nhiều các tính năng hữu ích khác, các bạn có thể tự tìm hiểu và thực hành. Chúc các bạn thành công :3

### Nếu bạn gặp lỗi không thể tự xử lý vui lòng tạo "Issues" trên github hoặc cần gấp có thể liên hệ qua Zalo mình 0326 681 592

### Các bạn có thể tích vào ngôi sao ở repo để giúp mình có thêm động lực cống hiến và phát triển hơn ạ ❤️ Cảm ơn mọi người ❤️