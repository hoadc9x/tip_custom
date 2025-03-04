# Setup Laravel 7 + MySQL 8 + PHP 7.4 + Nginx
**Step1: Cập nhật & cài đặt**
-----------------------------
### **1.1. Cập nhật hệ thống**
`sudo apt update && sudo apt upgrade -y`

### **1.2. Cài đặt Nginx**
`sudo apt install nginx -y`

Khởi động và bật Nginx chạy khi khởi động:

`sudo systemctl enable nginx`

`sudo systemctl start nginx`

### **1.3. Cài đặt MySQL 8**

`sudo apt install mysql-server -y`

Sau khi cài đặt xong, chạy lệnh sau để bảo mật MySQL:

`sudo mysql_secure_installation`

Trả lời các câu hỏi như sau:
-   **VALIDATE PASSWORD COMPONENT?** → Chọn `N` (nếu bạn không muốn yêu cầu mật khẩu phức tạp).
-   **Remove anonymous users?** → `Y`
-   **Disallow root login remotely?** → `Y`
-   **Remove test database and access to it?** → `Y`
-   **Reload privilege tables now?** → `Y`
Khởi động MySQL:

`sudo systemctl enable mysql`

`sudo systemctl start mysql`

### **1.4. Cài đặt PHP 7.4 và PHP-FPM**

`sudo apt install php7.4 php7.4-fpm php7.4-mysql php7.4-xml php7.4-mbstring php7.4-bcmath php7.4-curl php7.4-zip unzip -y`

Kiểm tra PHP-FPM đã chạy chưa:

`sudo systemctl enable php7.4-fpm`

`sudo systemctl start php7.4-fpm`

* * * * *

**Bước 2: Cấu hình MySQL cho Laravel**
--------------------------------------

Đăng nhập vào MySQL:

`sudo mysql -u root -p`

Tạo database và user cho Laravel:

`CREATE DATABASE laravel_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;`

`CREATE USER 'laravel_user'@'localhost' IDENTIFIED BY 'your_secure_password';`

`GRANT ALL PRIVILEGES ON laravel_db.* TO 'laravel_user'@'localhost';`

`FLUSH PRIVILEGES;`

`EXIT;`

* * * * *

**Bước 3: Cài đặt Laravel**
---------------------------

### **3.1. Cài đặt Composer**
`sudo apt install curl -y`

`curl -sS https://getcomposer.org/installer | php`

`sudo mv composer.phar /usr/local/bin/composer`

### **3.2. Tải và cài đặt Laravel**
`cd /var/www/html`

`sudo git clone https://github.com/laravel/laravel.git laravel_project`

`cd laravel_project`

`sudo composer install --no-dev --optimize-autoloader`

.....
### **3.3. Cấu hình `.env`**

Tạo file `.env` từ `.env.example`:

`cp .env.example .env`

Cập nhật thông tin database trong `.env`:

`DB_CONNECTION=mysql`

`DB_HOST=127.0.0.1`

`DB_PORT=3306`

`DB_DATABASE=laravel_db`

`DB_USERNAME=laravel_user`

`DB_PASSWORD=your_secure_password`

### **3.4. Thiết lập quyền thư mục**
`sudo chown -R www-data:www-data /var/www/html/laravel_project`

`sudo chmod -R 775 /var/www/html/laravel_project/storage /var/www/html/laravel_project/bootstrap/cache`

### **3.5. Tạo key và chạy migration**
`php artisan key:generate`

`php artisan migrate --force`

* * * * *

**Bước 4: Cấu hình Nginx cho Laravel**
--------------------------------------

Tạo file cấu hình mới:

`sudo nano /etc/nginx/sites-available/laravel_project`

Thêm nội dung sau:
```sh
server {
    listen 80;
    server_name yourdomain.com;
    root /var/www/html/laravel_project/public;

    index index.php index.html index.htm;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php7.4-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.ht {
        deny all;
    }

    error_log /var/log/nginx/laravel_project_error.log;
    access_log /var/log/nginx/laravel_project_access.log;
}
```

Kích hoạt cấu hình:

`sudo ln -s /etc/nginx/sites-available/laravel_project /etc/nginx/sites-enabled/`

Xóa file mặc định (nếu có):

`sudo rm /etc/nginx/sites-enabled/default`

Kiểm tra lỗi cấu hình:

`sudo nginx -t`

Nếu không có lỗi, khởi động lại Nginx:

`sudo systemctl restart nginx`

**Bước 5: Kiểm tra và hoàn tất**
--------------------------------

### **5.1. Kiểm tra Laravel**

`php artisan config:clear`

`php artisan cache:clear`

`php artisan route:cache`

### **5.2. Kiểm tra quyền thư mục**

`sudo chown -R www-data:www-data /var/www/html/laravel_project`

`sudo chmod -R 775 /var/www/html/laravel_project/storage /var/www/html/laravel_project/bootstrap/cache`

### **5.3. Restart services**

`sudo systemctl restart nginx`
`sudo systemctl restart php7.4-fpm`
`sudo systemctl restart mysql`

### **5.4. Kiểm tra website**

Truy cập `http://yourdomain.com` hoặc `http://your_vps_ip` để kiểm tra.

Nếu có lỗi, kiểm tra log:

`sudo tail -f /var/log/nginx/laravel_project_error.log`

`sudo tail -f /var/log/nginx/error.log`

* * * * *

**Bước 6: Cấu hình HTTPS (SSL)**
--------------------------------------------------
### **6.1. Dùng Let's Encrypt**
Cài certbot

`sudo apt install certbot python3-certbot-nginx -y`

Cấu hình cho yourdomain.com -> Chứng chỉ sẽ được tự động lưu vào: /etc/letsencrypt/live/yourdomain.com/

`sudo certbot --nginx -d yourdomain.com`

Cập nhật lại cấu hình Nginx:
`sudo nano /etc/nginx/sites-available/laravel_project`

```sh
server {
    listen 80;
    server_name yourdomain.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name yourdomain.com;
    root /var/www/html/clinic-dev/public;

    ssl_certificate /etc/letsencrypt/live/yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/yourdomain.com/privkey.pem;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;

    index index.php index.html index.htm;
    
    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php7.4-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.ht {
        deny all;
    }

    error_log /var/log/nginx/clinic-dev_error.log;
    access_log /var/log/nginx/clinic-dev_access.log;
}

```
Lưu lại và khởi động lại Nginx:
```sh
sudo nginx -t
sudo systemctl restart nginx
```

#### Cài tự động gia hạn SSL: (do Let's Encrypt chỉ có hạn 3 tháng)
- ##### Certbot đã tự động tạo một cronjob bằng systemd_timer nên không cần làm
    Check: `sudo systemctl list-timers`
- ##### Cách 1: Dùng crontab:
  `sudo crontab -e`

  Thêm dòng sau vào cuối file (kiểm tra và gia hạn chứng chỉ vào 3 giờ sáng mỗi ngày):

  `0 3 * * * certbot renew --quiet --post-hook "systemctl reload nginx"`

  Lưu và test thử: `sudo certbot renew --dry-run`

- ##### Cách 2: Dùng systemd timer (mặc định vps có)
  
  _Tạo systemd.service_
  
    Nếu VPS của bạn chưa có systemd service cho Certbot, hãy tạo nó bằng cách:
  
    `sudo nano /etc/systemd/system/certbot-renew.service`

    Thêm nội dung sau:
    ```sh
    [Unit]
    Description=Renew Let's Encrypt certificates
    Wants=network-online.target
    After=network-online.target
    
    [Service]
    Type=oneshot
    ExecStart=/usr/bin/certbot renew --quiet --agree-tos --no-self-upgrade
    ```  
    Lưu lại với `CTRL + X`, `Y`, `Enter`.
  
  _Tạo systemd timer để chạy định kỳ_
    Tạo file timer `/etc/systemd/system/certbot-renew.timer`:
  
    `sudo nano /etc/systemd/system/certbot-renew.timer`
  
    Thêm nội dung sau:
  
    ```sh
    [Unit]
    Description=Run Certbot renewal twice a day
    Wants=certbot-renew.service
    
    [Timer]
    OnCalendar=*-*-* 00,12:00:00
    RandomizedDelaySec=30m
    Persistent=true
    
    [Install]
    WantedBy=timers.target
    ```
    Giải thích:
    -   `OnCalendar=*-*-* 00,12:00:00` → Chạy 2 lần mỗi ngày (00:00 và 12:00).
    -   `RandomizedDelaySec=30m` → Tránh chạy cùng thời điểm với các server khác, giảm tải cho Let's Encrypt.
    -   `Persistent=true` → Nếu server bị tắt, khi bật lại sẽ chạy ngay nếu chưa chạy lần nào trong khoảng thời gian quy định.
    Lưu lại với `CTRL + X`, `Y`, `Enter`.

  _Bật và khởi động systemd timer_
  
    `sudo systemctl daemon-reload`
  
    `sudo systemctl enable certbot-renew.timer`
  
    `sudo systemctl start certbot-renew.timer`

  _Kiểm tra timer có hoạt động không_
  
    `systemctl list-timers --all | grep certbot`

  _Xem log của Certbot_

    `journalctl -u certbot-renew.service --no-pager --lines=50`

  _Chạy thử lệnh gia hạn xem có lỗi không_

    `sudo systemctl start certbot-renew.service`
  
### **6.2. Dùng SSL trả phí**
Giả sử đã nhận được hai file:
-   `fullchain.pem` (Chứng chỉ đầy đủ)
-   `privatekey.pem` (Khóa riêng tư)
  
Di chuyển chúng vào thư mục `/etc/nginx/ssl/yourdomain.com/`:
  ```sh
  sudo mkdir -p /etc/nginx/ssl/yourdomain.com
  sudo mv fullchain.pem privatekey.pem /etc/nginx/ssl/yourdomain.com/
  sudo chmod 600 /etc/nginx/ssl/yourdomain.com/*
  ```
Cấu hình lại Nginx cho yourdomain.com:

  `sudo nano /etc/nginx/sites-available/laravel_project`

Sửa lại nội dung:
```sh
server {
    listen 80;
    server_name yourdomain.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name yourdomain.com;
    root /var/www/html/clinic/public;

    ssl_certificate /etc/nginx/ssl/yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/nginx/ssl/yourdomain.com/privatekey.pem;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;

    index index.php index.html index.htm;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php7.4-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.ht {
        deny all;
    }

    error_log /var/log/nginx/clinic_error.log;
    access_log /var/log/nginx/clinic_access.log;
}
```
Lưu lại và khởi động lại Nginx:
```sh
sudo nginx -t
sudo systemctl restart nginx
```


* * * * *





