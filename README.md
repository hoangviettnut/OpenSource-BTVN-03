# OpenSource-BTVN-03
# SINH VIÊN THỰC HIỆN: LƯƠNG HOÀNG VIỆT - K225480106073
# MÔN HỌC: PHÁT TRIỂN ỨNG DỤNG VỚI MÃ NGUỒN MỞ
# BÀI TẬP 03
# BÀI LÀM
## 1. Tạo Folder chứa project: viet_wp 
## 2. Tạo file docker-compose.yml cho BTVN03
Bài này vẫn sử dụng chung nginx và cloudflare với bài 2.
```
version: '3.8'
services:
  btvn03_db:
    image: mariadb:latest
    container_name: btvn03_db
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wpuser
      MYSQL_PASSWORD: wppassword
    volumes:
      - mariadb_data:/var/lib/mysql
    networks:
      - web-gateway

  btvn03_pma:
    image: phpmyadmin:latest
    container_name: btvn03_pma
    environment:
      PMA_HOST: btvn03_db
      MYSQL_ROOT_PASSWORD: rootpassword
    depends_on:
      - btvn03_db
    networks:
      - web-gateway

  btvn03_wordpress:
    image: wordpress:latest
    container_name: btvn03_wordpress
    environment:
      WORDPRESS_DB_HOST: btvn03_db:3306
      WORDPRESS_DB_USER: wpuser
      WORDPRESS_DB_PASSWORD: wppassword
      WORDPRESS_DB_NAME: wordpress
    depends_on:
      - btvn03_db
    networks:
      - web-gateway

volumes:
  mariadb_data:

networks:
  web-gateway:
    external: true
```

## 3. Khởi chạy ứng dụng: 
```
docker compose up -d
```
<img width="461" height="136" alt="image" src="https://github.com/user-attachments/assets/7150a278-6eda-4a8f-9b37-b1b8b4827f36" />


## 4. Cấu hình Nginx & Public mạng
Thêm cấu hình Nginx cho BTVN 03:
```
            # --- BTVN 03: WORDPRESS ---
    
    # 6. Web WordPress
    server {
        listen 80;
        server_name btvn03.luonghoangviet.io.vn;

        location / {
            set $upstream_wp btvn03_wordpress;
            # Trỏ vào port 80 bên trong container WordPress
            proxy_pass http://$upstream_wp:80; 
            
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto https;
        }
    }

    # 7. Giao diện phpMyAdmin (tuỳ chọn nếu thầy muốn xem)
    server {
        listen 80;
        server_name pma03.luonghoangviet.io.vn;

        location / {
            set $upstream_wp_pma btvn03_pma;
            # phpMyAdmin chạy port 80 bên trong container
            proxy_pass http://$upstream_wp_pma:80; 
            
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }
```
Thêm Subdomain trên Cloudflare:

<img width="1057" height="772" alt="image" src="https://github.com/user-attachments/assets/256f60fc-462a-4ac6-8556-e5582077c877" />

Sau đó dùng lệnh tại nơi chứa nginx.conf:
```
docker exec -it nginx-master nginx -s reload
```
## 5. Thiết lập WordPress ban đầu
Truy cập http://btvn03.luonghoangviet.io.vn, điền tên web, tạo admin và hoàn tất cài đặt.

Giao diện ban đầu khi bắt đầu thiết lập:

<img width="1920" height="1140" alt="image" src="https://github.com/user-attachments/assets/7fdd42d6-e80f-4747-b0f2-bbb5156c5871" />

Sau khi chọn ngôn ngữ, chuyển tiếp vào trang thiết lập Username và Password:

<img width="1920" height="1140" alt="image" src="https://github.com/user-attachments/assets/21d3f450-43da-49f3-9b25-857b6b614a21" />

Khi cài đặt xong truy cập https://btvn03.luonghoangviet.io.vn/ ta được giao diện ban đầu khi chưa có bài viết nào ( chỉ có những bài mặc định )

<img width="1260" height="2800" alt="test" src="https://github.com/user-attachments/assets/2ca7d832-04be-4993-a293-e1075da4efb2" />

Truy cập vào Dashboard: btvn03.luonghoangviet.io.vn/wp-admin/ để bắt đầu tạo trang web

<img width="1920" height="1140" alt="image" src="https://github.com/user-attachments/assets/add6304f-849e-4333-8c92-b0fcfe135ddd" />

## 6. Tạo bài viết giới thiệu bản thân

Tại dashboard user có thể Custom cho trang web, bài viết của mình

Sau khi chọn template ưng ý (hoặc Custom ): 

Chọn Post => Add Post và điền thông tin, ảnh,... để tạo bài viết: 

Giao diện khi đang edit

<img width="1920" height="1140" alt="image" src="https://github.com/user-attachments/assets/9c1142c2-242c-4fa1-85e4-af6c8395ec47" />

Sau khi Public, truy cập https://btvn03.luonghoangviet.io.vn/2026/05/12/luonghoangviet/ để kiểm tra:

<img width="1920" height="1140" alt="image" src="https://github.com/user-attachments/assets/618329d2-a944-4a84-8b8c-94495b1ca780" />

Kiểm tra trên điện thoại bằng 4G: 

<img width="1260" height="2800" alt="test1" src="https://github.com/user-attachments/assets/f0398b19-d218-4608-bba8-f6cb572d1872" />

## 7. Viết bài Giới thiệu Ngành học 

## NOTE: Vì mới tìm hiểu về WordPress nên sẽ sử dụng nhúng Video thông qua URL thay vì upload vì hiện tại video đang bị giới hạn kích thước Upload

Tương tự với post giới thiệu bản thân, sau khi thêm các thành phần mà bản thân muốn rồi publish, truy cập https://btvn03.luonghoangviet.io.vn/2026/05/12/ktmt/ để kiểm tra:

<img width="1920" height="1140" alt="image" src="https://github.com/user-attachments/assets/d0fb7795-77d4-427e-8ad6-e9ebd5bf0a64" />

Kiểm tra trên điện thoại bằng 4G:

<img width="1260" height="2800" alt="test2" src="https://github.com/user-attachments/assets/789d83c8-26e0-4380-918e-e8d03bea17e3" />

## 8. Kiểm tra btvn03.luonghoangviet.io.vn sau khi đã tạo các bài viết

Truy cập https://btvn03.luonghoangviet.io.vn/ để xem danh sách các bài viết: 

<img width="1920" height="1140" alt="image" src="https://github.com/user-attachments/assets/7af3d56a-70a4-40af-9e71-ba1efcd51fe3" />

Test trên điện thoại: 

<img width="1260" height="2800" alt="test" src="https://github.com/user-attachments/assets/1ae911f2-00fb-419d-b657-124a30fa293c" />

Lúc này ta có thể xem các bài viết đã được đăng ( xem chi tiết thì bấm vào bài viết cần xem)

## 9. Nhận xét

  Về công sức triển khai: Rất nhanh nếu chỉ chạy nội bộ bằng Docker Compose. Tuy nhiên, khi public ra Internet qua cấu trúc Nginx Reverse Proxy và Cloudflare Tunnel thì khá tốn công sức để fix các lỗi giao thức mạng (như lỗi Mixed Content hay nghẽn REST API khi đăng bài).

  Về độ Dễ/Khó sử dụng: Giao diện quản trị (UI) cực kỳ thân thiện, dễ kéo thả, chèn ảnh/video bằng vài cú click chuột. Tuy nhiên, với dân kỹ thuật, khi hệ thống ẩn giấu lỗi sâu bên trong code hoặc Database thì việc debug (tìm và sửa lỗi) khá lằng nhằng và vất vả.

  Về tiêu tốn tài nguyên máy chủ: WordPress khá "ăn" tài nguyên. Một cụm container gồm WordPress (chạy Apache/PHP) và MariaDB ngốn kha khá RAM và CPU của máy chủ Ubuntu so với các trang web code thuần. Nếu lạm dụng cài nhiều giao diện và Plugin nặng, server sẽ rất dễ bị đuối sức.

















