# Conceptual Overview

A reverse proxy acts as an intermediary server that sits between clients and your backend application. In this configuration:

- Nginx listens on ports 80 (HTTP) and 443 (HTTPS)
- It forwards incoming requests to your MicroOcpp application on port 8000
- All communication appears to come directly from your domain to users

# Prerequisites

1. Nginx installed on your system
2. MicroOcpp running on port 8000
3. Domain name (for SSL certificates)
4. Root access to your server

# Step-by-Step Configuration

1. Instal Nginx (if not installed)
```
sudo apt update
sudo apt install nginx
```

2. Create a configuration file
```
sudo nano /etc/nginx/sites-available/mo_simulator.conf
```

4. Add reverse proxy configuration mapping 80 and 443 to 8000
```
server {
    listen 80;
    server_name 10.199.67.55 www.charge.okq8.se charge.okq8.se;

    location / {
        proxy_pass http://10.199.67.55:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

server {
    listen 443 ssl;
    server_name 10.199.67.55 www.charge.okq8.se charge.okq8.se;

    ssl_certificate /etc/nginx/certs/selfsigned.crt;
    ssl_certificate_key /etc/nginx/certs/selfsigned.key;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;

    location / {
        proxy_pass http://10.199.67.55:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

5. Security Considerations
    - SSL Certificate Setup
    Generate self-signed certificate (development):
    ```
    sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout /etc/nginx/certs/selfsigned.key \
    -out /etc/nginx/certs/selfsigned.crt \
    -subj "/C=SWE/ST=Uppland/L=se/O=OKQ8/CN=microocp-simulator"
    ```

    - Firewall Configuration, firewall may cause problems with RDC and SSH
    ```
    sudo ufw allow ssh
    sudo ufw allow 3389
    sudo ufw allow http
    sudo ufw allow https
    sudo ufw enable
    ```

6. Enable the configuration
```
sudo ln -s /etc/nginx/sites-available/mo_simulator.conf /etc/nginx/sites-enabled/
sudo nginx -t
```

7. Restart Nginx
```
sudo systemctl restart nginx
```

8. Troubleshooting Tips
- Check Nginx logs for errors:
```
tail -f /var/log/nginx/error.log
```
- Verify MicroOcpp is accessible
```
curl 10.199.67.55:80
curl 10.199.67.55:8000
curl -I 10.199.67.55:80
curl -I 10.199.67.55:8000
```
- Test both HTTP and HTTPS connections
```
curl http://10.199.67.55
curl https://10.199.67.55
```
- List TCP Listened ports 
```
ss -lntp
```