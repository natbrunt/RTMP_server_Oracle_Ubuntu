# RTMP_server_Oracle_Ubuntu

Step 1

`ssh -i -<SSH_KEY_NAME>  ubuntu@<YOUR_PUBLIC_IP>`

Step 2
```
sudo apt update
sudo apt install -y build-essential libpcre3 libpcre3-dev libssl-dev zlib1g-dev git
```
Step 3
```
cd /usr/local/src sudo wget http://nginx.org/download/nginx-1.26.0.tar.gz
sudo tar -zxvf nginx-1.26.0.tar.gz
sudo git clone https://github.com/arut/nginx-rtmp-module.git
cd nginx-1.26.0
```
Step 4
```
sudo ./configure --prefix=/etc/nginx \
  --sbin-path=/usr/sbin/nginx \
  --conf-path=/etc/nginx/nginx.conf \
  --pid-path=/var/run/nginx.pid \
  --with-http_ssl_module \
  --with-http_v2_module \
  --add-module=../nginx-rtmp-module
sudo make
sudo make install
```
Step 5

`sudo nano /lib/systemd/system/nginx.service`

paste this in the file

```
[Unit]
Description=A high performance web server and a reverse proxy server
After=network.target
[Service]
ExecStart=/usr/sbin/nginx
ExecReload=/usr/sbin/nginx -s reload
ExecStop=/usr/sbin/nginx -s quit
PIDFile=/var/run/nginx.pid
Restart=on-failure
Type=forking
[Install]
WantedBy=multi-user.target
sudo systemctl daemon-reexec
sudo systemctl daemon-reload
```
Step 6

`sudo nano /etc/nginx/nginx.conf `

paste this in the file

```
worker_processes  auto;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    server {
        listen 80;
        location / {
            return 200 'RTMP server running';
            add_header Content-Type text/plain;
        }
    }
}
rtmp {
    server {
        listen 1935;
        chunk_size 4096;
        application live {
            live on;
            record off;
            allow publish all;
            allow play all;
            drop_idle_publisher 5s;
        }
    }
}
```
Step 7
```
sudo systemctl enable nginx
sudo systemctl start nginx
```
Step 8

`sudo nano /etc/iptables/rules.v4`

edit the file to include port 1935

`sudo iptables-restore < /etc/iptables/rules.v4`

Step 9 (debugging)

locate the attached nginx conf, test if it is valid

```sudo nginx -t```

live view of nginx error log

```sudo tail -f /etc/nginx/logs/error.log```
