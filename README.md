# kong-deploy
docker compose -f docker-compose-https4.yaml run --rm kong kong migrations bootstrap

docker compose -f docker-compose-https4.yaml up -d


curl -k https://kong.parahyangandigital.cloud:8444/status


#Create sample helloworld

sudo mkdir -p /var/www/parahyangandigital.cloud
echo "Hello, World!" | sudo tee /var/www/parahyangandigital.cloud/index.html
sudo nano /etc/nginx/sites-available/parahyangandigital.cloud

server {
    listen 80;
    server_name parahyangandigital.cloud www.parahyangandigital.cloud;

    root /var/www/parahyangandigital.cloud;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}

sudo ln -s /etc/nginx/sites-available/parahyangandigital.cloud /etc/nginx/sites-enabled/

sudo nginx -t
sudo systemctl restart nginx

=============

docker compose -f docker-compose-konga-http.yaml up -d kong-db
docker compose -f docker-compose-konga-http.yaml up -d kong-migrations
docker logs kong-migrations

docker compose -f docker-compose-konga-http.yaml up -d kong konga

sudo mkdir -p /var/www/konga.parahyangandigital.cloud

sudo nano /etc/nginx/sites-available/parahyangandigital.cloud

sudo ln -s /etc/nginx/sites-available/parahyangandigital.cloud /etc/nginx/sites-enabled/

#http virtualhost

server {
    listen 80;
    server_name konga.parahyangandigital.cloud;

    # Location block untuk reverse proxy ke Konga
    location / {
        proxy_pass http://localhost:1337;  # Arahkan ke port 1337 tempat Konga UI berjalan di container
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

sudo ln -s /etc/nginx/sites-available/konga.parahyangandigital.cloud /etc/nginx/sites-enabled/

sudo nginx -t


#Https virtualhost

server {
    listen 80;
    server_name konga.parahyangandigital.cloud;

    # Redirect HTTP ke HTTPS
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name konga.parahyangandigital.cloud;

    # Path ke sertifikat SSL (pastikan Anda sudah mengkonfigurasi SSL dengan Let's Encrypt atau lainnya)
    ssl_certificate /etc/letsencrypt/live/konga.parahyangandigital.cloud/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/konga.parahyangandigital.cloud/privkey.pem;

    # Protokol SSL yang aman
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers 'ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384';

    # SSL session cache
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 1d;
    ssl_prefer_server_ciphers on;

    # Set headers untuk keamanan tambahan
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;
    add_header X-Frame-Options DENY;
    add_header X-Content-Type-Options nosniff;
    add_header Referrer-Policy no-referrer-when-downgrade;

    # Location block untuk reverse proxy ke Konga
    location / {
        proxy_pass http://localhost:1337;  # Arahkan ke port 1337 tempat Konga UI berjalan
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

