## Hướng dẫn cài đặt NextCloud trên CentOS 7

- [1. Giới thiệu](#1)
- [2. Các bước tiến hành](#2)
	- [2.1 Cài đặt LAMP Stack](#21)
	- [2.2 Cấu hình MariaDB](#22)
	- [2.3 Tạo User và Database cho NextCloud](#23)
	- [2.4 Tải và giải nén NextCloud](#24)
	- [2.5 Cấu hình NextCloud](#25)
	
<a name="1" />
	
### 1. Giới thiệu

[NextCloud](https://nextcloud.com) là một giải pháp lưu trữ dữ liệu giống như Dropbox, Google Drive hay OneDrive,... Bài viết này sẽ hướng dẫn các bạn cài đặt NextCloud trên CentOS 7.

| Thông tin máy chủ | |
|--|--|
| OS | CentOS 7 |
| CPU | 1 |
| RAM | 2 GB |
| IP Address | 192.168.100.192/24 |
| Gateway | 192.168.100.1 |


<a name="2" />

### 2. Các bước tiến hành

<a name="21" />

#### 2.1 Cài đặt LAMP Stack

NextCloud được viết bằng PHP và sử dụng MariaDB/MySQL. Sau đây, chúng ta sẽ cài đặt LAMP Stack (Linux Apache MariaDB/MySQL và PHP)

Trước khi cài đặt LAMP, chúng ta phải cài đặt repository EPEL để hỗ trợ các gói cài đặt mở rộng.

```
yum -y install epel-release wget git unzip
yum update -y
```

- **Bước 1**: Cài đặt và cấu hình Apache

	- Cài đặt

	```sh
	yum -y install httpd
	```

	- Cho phép sử dụng file `.htaccess`

	```sh
	sed -i 's/AllowOverride None/AllowOverride all/' /etc/httpd/conf/httpd.conf
	sed -i 's/AllowOverride none/AllowOverride all/' /etc/httpd/conf/httpd.conf	
	echo -e "ServerSignature Off \nServerTokens Prod" >> /etc/httpd/conf/httpd.conf
	```
	
	- Chạy dịch vụ Apache và cho phép khởi động cùng hệ thống

	```sh
	systemctl start httpd.service
	systemctl enable httpd.service
	```

- **Bước 2**: Cài đặt PHP 7.0

	- Cài đặt PHP
	
	```sh
	rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm
	yum install -y php70w php70w-dom php70w-mbstring php70w-gd php70w-pdo php70w-json php70w-xml php70w-zip php70w-curl php70w-mcrypt php70w-pear setroubleshoot-server bzip2 php70w-mysql
	```
	
	- Cấu hình PHP
	
	Chúng ta chỉnh lại phần cấu hình kích thước File cho phép Upload lên NextCloud của PHP.
	
	```sh
	sed -i "s/post_max_size = 8M/post_max_size = 200M/" /etc/php.ini
	sed -i "s/upload_max_filesize = 2M/upload_max_filesize = 200M/" /etc/php.ini
	```
	
	- Khởi động lại Apache
	
	```sh
	systemctl restart httpd.service
	```

- **Bước 3**: Cài đặt MariaDB

	- Cài đặt MariaDB
	
	```sh
	yum -y install mariadb mariadb-server
	```
	
	- Chạy dịch vụ MariaDB và cho phép khởi động cùng hệ thống
	
	```sh
	systemctl start mariadb
	systemctl enable mariadb
	```

<a name="22" />

#### 2.2 Cấu hình MariaDB

Tại đây, chúng ta sẽ đặt mật khẩu cho user `root` của MariaDB và xóa database có tên là `test`.

```sh
mysql -uroot -e "set password for 'root'@'localhost' = password('$PASSWORD');"
mysql -uroot -p$PASSWORD -e "DROP DATABASE test;"
mysql -uroot -p$PASSWORD -e "DELETE from mysql.user where Password = '';"
mysql -uroot -p$PASSWORD -e "CREATE USER 'root'@'%' IDENTIFIED BY '$PASSWORD';"
mysql -uroot -p$PASSWORD -e "GRANT ALL PRIVILEGES ON *.* TO 'root'@'%';"
mysql -uroot -p$PASSWORD -e "FLUSH PRIVILEGES;"
```

**Chú ý**: Thay thế password của `root` mà bạn muốn đặt vào `$PASSWORD`.

<a name="23" />

#### 2.3 Tạo User và Database cho NextCloud

```sh
mysql -uroot -p$PASSWORD -e "CREATE DATABASE nextcloud;"
mysql -uroot -p$PASSWORD -e ""GRANT ALL PRIVILEGES ON nextcloud.* TO 'nextclouduser'@'localhost' IDENTIFIED BY 'PASSWORD_OC';
mysql -uroot -p$PASSWORD -e "FLUSH PRIVILEGES;"
```

- **Chú ý**: 
	- Thay thế password của `root` vào `$PASSWORD`.
	- Thay thế password của `nextclouduser` mà bạn muốn đặt vào `PASSWORD_OC`

<a name="24" />

#### 2.4 Tải và giải nén NextCloud

Trong bài viết này, chúng tôi sẽ hướng dẫn các bạn cài bản NextCloud version 13 (mới nhất trong thời điểm viết bài).

- **Bước 1**: Tải NextCloud

```sh
cd /opt/
wget https://download.nextcloud.com/server/releases/nextcloud-13.0.0.zip
```

- **Bước 2**: Giải nén NextCloud

```sh
cd /opt/
unzip nextcloud-13.0.0.zip
```

- **Bước 3**: Chuyển thư mục và phân quyền cho thư mục

```
cd /opt/
mv nextcloud/* /var/www/html/
chmod 755 -R /var/www/html/
chown apache. -R /var/www/html/
```
		
<a name="25" />

#### 2.5 Cấu hình NextCloud

- **Bước 1**: Truy cập vào địa chỉ IP bằng trình duyệt

	<img src="/images/nc-1.png" />
	
	- `1`: Khai báo thông tin user quản trị
	- `2`: Khai báo thông tin thư mục lưu trữ dữ liệu. Chú ý: Phân quyền cho user `apache` sở hữu thư mục này. 
		
	
- **Bước 2**: Khai báo thông tin DATABASE

	<img src="/images/nc-2.png" />
	
	- `1`: Chọn kiểu DATABASE là `MySQL/MariaDB`
	- `2`: Khai báo thông tin về DATABASE mà chúng ta tạo ở bước [trên](#23).
	- `3`: Hoàn tất việc cài đặt

- **Bước 3**: Sau khi cài đặt xong, giao diện của NextCloud sẽ xuất hiện

	<img src="/images/nc-3.png" />

<a name="3" />

### 3. Tham khảo

- https://www.digitalocean.com/community/tutorials/how-to-install-linux-apache-mysql-php-lamp-stack-on-centos-7
- https://www.marksei.com/install-nextcloud-12-centos-7/

