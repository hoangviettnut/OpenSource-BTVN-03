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
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }

    # 7. Giao diện phpMyAdmin 
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

Sau đó dùng lệnh:
```
docker exec -it nginx-master nginx -s reload
```
