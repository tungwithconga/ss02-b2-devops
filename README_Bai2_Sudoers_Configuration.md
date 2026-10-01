# Bài 2: Khởi tạo User thường và thiết lập đặc quyền quản trị (Sudoers Configuration)

## 1. Mục tiêu

-   Tạo một tài khoản người dùng thường (Non-root user) trên Ubuntu
    Server.
-   Hạn chế việc sử dụng trực tiếp tài khoản `root` trong quá trình quản
    trị hàng ngày.
-   Thêm user mới vào nhóm `sudo` để có thể thực hiện các tác vụ quản
    trị khi cần.
-   Sao chép cấu hình SSH Key sang user mới.
-   Thiết lập đúng owner và permission cho `.ssh` và `authorized_keys`.
-   Kiểm tra đăng nhập bằng SSH Key và quyền `sudo`.

## 2. Môi trường thực hành

-   Server: Ubuntu 22.04 LTS
-   Máy cá nhân: Windows
-   User hiện tại: `tung`
-   User DevOps được tạo: `devops-tung`
-   Phương thức SSH: ED25519 SSH Key

> Theo đề bài, user cần tạo có tên `devops`. Tuy nhiên VPS trong bài
> thực hành được sử dụng chung bởi nhiều người nên user `devops-tung`
> được sử dụng để tránh xung đột với user của người khác. Quy trình cấu
> hình và phân quyền hoàn toàn tương tự user `devops`.

## 3. Đăng nhập vào Server

Từ CMD/PowerShell trên máy cá nhân:

``` bash
ssh tung@<SERVER_IP>
```

Kiểm tra user hiện tại:

``` bash
whoami
```

Kết quả:

``` text
tung
```

## 4. Tạo Non-root User

Tạo user mới:

``` bash
sudo adduser devops-tung
```

Hệ thống yêu cầu nhập mật khẩu mới:

``` text
New password:
Retype new password:
```

Các thông tin bổ sung như Full Name, Room Number, Work Phone... có thể
để trống.

Kiểm tra user:

``` bash
id devops-tung
```

Kết quả tương tự:

``` text
uid=1002(devops-tung) gid=1002(devops-tung) groups=1002(devops-tung)
```

Điều này xác nhận user `devops-tung` đã được tạo thành công.

## 5. Thêm user vào nhóm sudo

Thực hiện:

``` bash
sudo usermod -aG sudo devops-tung
```

Kiểm tra:

``` bash
groups devops-tung
```

Kết quả mong đợi:

``` text
devops-tung : devops-tung sudo
```

User `devops-tung` lúc này vẫn là user thường nhưng có thể sử dụng
`sudo` để thực thi các lệnh cần đặc quyền quản trị.

## 6. Sao chép cấu hình SSH Key

Ở bài thực hành trước, SSH Public Key đã được cấu hình cho user `tung`
tại:

``` text
/home/tung/.ssh/authorized_keys
```

Sao chép thư mục `.ssh` từ user `tung` sang `devops-tung`:

``` bash
sudo rsync --archive --chown=devops-tung:devops-tung /home/tung/.ssh /home/devops-tung/
```

Nếu `rsync` chưa được cài đặt:

``` bash
sudo apt install rsync -y
```

sau đó chạy lại lệnh sao chép.

> Trong môi trường DigitalOcean của đề gốc, Public Key có thể nằm trong
> `/root/.ssh`. Khi đó có thể dùng cách tương ứng:
>
> ``` bash
> sudo rsync --archive --chown=devops:devops /root/.ssh /home/devops/
> ```

## 7. Thiết lập quyền cho SSH

Đảm bảo thư mục `.ssh` thuộc sở hữu của user mới:

``` bash
sudo chown -R devops-tung:devops-tung /home/devops-tung/.ssh
```

Thiết lập permission cho thư mục `.ssh`:

``` bash
sudo chmod 700 /home/devops-tung/.ssh
```

Thiết lập permission cho `authorized_keys`:

``` bash
sudo chmod 600 /home/devops-tung/.ssh/authorized_keys
```

Kiểm tra:

``` bash
sudo ls -ld /home/devops-tung/.ssh
sudo ls -l /home/devops-tung/.ssh/authorized_keys
```

Kết quả mong đợi có dạng:

``` text
drwx------ ... devops-tung devops-tung ... /home/devops-tung/.ssh
-rw------- ... devops-tung devops-tung ... authorized_keys
```

Ý nghĩa:

  Thành phần          Permission             Owner
  ------------------- ---------------------- ---------------
  `.ssh`              `700` (`drwx------`)   `devops-tung`
  `authorized_keys`   `600` (`-rw-------`)   `devops-tung`

## 8. Kiểm tra đăng nhập bằng SSH Key

Từ CMD/PowerShell trên Windows:

``` bash
ssh -i %USERPROFILE%\.ssh\id_ed25519 devops-tung@<SERVER_IP>
```

Kết quả mong đợi:

``` text
Welcome to Ubuntu 22.04.x LTS
...
devops-tung@<hostname>:~$
```

Kiểm tra user:

``` bash
whoami
```

Kết quả:

``` text
devops-tung
```

Nếu SSH đăng nhập thành công mà không yêu cầu password đăng nhập của
user, cấu hình SSH Key đã hoạt động.

## 9. Kiểm tra đặc quyền quản trị

Sau khi đăng nhập bằng `devops-tung`, chạy:

``` bash
sudo whoami
```

Hệ thống có thể yêu cầu mật khẩu của `devops-tung`:

``` text
[sudo] password for devops-tung:
```

Sau khi nhập đúng mật khẩu, kết quả mong đợi:

``` text
root
```

Điều này chứng minh user thường `devops-tung` đã được cấp quyền sử dụng
`sudo` thành công.

## 10. Kết quả thực hành

-   [x] Tạo Non-root user thành công.
-   [x] User có thư mục home riêng.
-   [x] Thêm user vào nhóm `sudo`.
-   [x] Sao chép SSH Public Key sang user mới.
-   [x] Thiết lập owner cho thư mục `.ssh`.
-   [x] Thiết lập `.ssh` permission `700`.
-   [x] Thiết lập `authorized_keys` permission `600`.
-   [x] Đăng nhập user mới bằng SSH Key.
-   [x] `sudo whoami` trả về `root`.

## 11. Minh chứng

Có thể lưu ảnh vào:

``` text
homework/session_02/ex2/
├── README.md
└── images/
    ├── create-user.png
    ├── sudo-group.png
    ├── ssh-login.png
    └── sudo-whoami.png
```

Chèn ảnh vào README:

``` markdown
### Tạo user

![Create user](images/create-user.png)

### Kiểm tra nhóm sudo

![Sudo group](images/sudo-group.png)

### SSH bằng user mới

![SSH login](images/ssh-login.png)

### Kiểm tra sudo

![sudo whoami](images/sudo-whoami.png)
```

## 12. Cấu trúc nộp bài

``` text
homework/
└── session_02/
    └── ex2/
        ├── README.md
        └── images/
```

## 13. Kết luận

Qua bài thực hành, một tài khoản Non-root riêng đã được tạo và cấu hình
để quản trị Ubuntu Server thông qua `sudo`. SSH Public Key được sao chép
sang user mới và các permission cần thiết được thiết lập đúng. User có
thể đăng nhập bằng SSH Key và chỉ nâng quyền lên `root` thông qua `sudo`
khi cần, phù hợp với nguyên tắc hạn chế sử dụng trực tiếp tài khoản root
trong vận hành hệ thống.
