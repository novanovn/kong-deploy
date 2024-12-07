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





#Cek DB kong

psql -h kong-db -U kong -d kong

\dt

#Membuat service
curl -i -X POST http://103.129.149.97:8001/services/ \
  --data name=penjualan-api \
  --data url=http://103.196.155.238:3003/penjualan


HTTP/1.1 201 Created
Date: Sat, 07 Dec 2024 14:27:11 GMT
Content-Type: application/json; charset=utf-8
Connection: keep-alive
Access-Control-Allow-Origin: *
Access-Control-Allow-Credentials: true
Content-Length: 388
X-Kong-Admin-Latency: 9
Server: kong/3.8.0

{"retries":5,"tls_verify":null,"write_timeout":60000,"tls_verify_depth":null,"path":"/penjualan","client_certificate":null,"enabled":true,"id":"696aa7d4-9604-4db4-b867-71ab89d4395c","updated_at":1733581631,"ca_certificates":null,"port":3003,"protocol":"http","connect_timeout":60000,"tags":null,"read_timeout":60000,"created_at":1733581631,"name":"penjualan-api","host":"103.196.155.238"}root@KTV-2022-181:~/apisix-deploy#

#Membuat Route
curl -i -X POST http://103.129.149.97:8001/services/penjualan-api/routes \
  --data 'hosts[]=kong.parahyangandigital.cloud' \
  --data 'paths[]=/penjualan'

#Membuat Route
curl -i -X POST http://103.129.149.97:8001/services/penjualan-api/routes \
  --data 'hosts[]=103.129.149.97' \
  --data 'paths[]=/penjualan'


curl -i -X PATCH http://localhost:8001/routes/{route_id} \
  --data 'hosts[]=api.parahyangandigital.cloud' \
  --data 'hosts[]=newhost.parahyangandigital.cloud'

HTTP/1.1 201 Created
Date: Sat, 07 Dec 2024 14:28:04 GMT
Content-Type: application/json; charset=utf-8
Connection: keep-alive
Access-Control-Allow-Origin: *
Access-Control-Allow-Credentials: true
Content-Length: 508
X-Kong-Admin-Latency: 9
Server: kong/3.8.0

{"protocols":["http","https"],"service":{"id":"696aa7d4-9604-4db4-b867-71ab89d4395c"},"preserve_host":false,"snis":null,"path_handling":"v0","methods":null,"tags":null,"sources":null,"headers":null,"paths":["/penjualan"],"destinations":null,"id":"fcaef37c-a1fc-41ef-a239-8199dd0ab9b4","strip_path":true,"hosts":["kong.parahyangandigital.cloud"],"regex_priority":0,"response_buffering":true,"https_redirect_status_code":426,"request_buffering":true,"name":null,"

curl http://103.129.149.97:8001/penjualan


curl -i -X PATCH http://localhost:8001/routes/{route_id} \
  --data 'hosts[]=api.parahyangandigital.cloud' \
  --data 'hosts[]=newhost.parahyangandigital.cloud'

curl -i -X PATCH http://localhost:8001/routes/{route_id} \
  --data 'hosts[]=api.parahyangandigital.cloud' \
  --data 'paths[]=/newpath

curl -i -X PATCH http://localhost:8001/routes/{route_id} \
  --data 'name=penjualan-api-v2' \
  --data 'hosts[]=api.parahyangandigital.cloud'





sudo nano /etc/nginx/sites-available/kong.parahyangandigitalcloud

server {
    listen 80;
    server_name kong.parahyangandigital.cloud;

    # Root directory untuk server
    #root /var/www/html;

    # Redirect ke service Kong di port 8000
    location / {
        proxy_pass http://localhost:8000;  # Mengarahkan trafik ke Kong pada port 8000
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # Set headers untuk keamanan tambahan
    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";
    add_header X-XSS-Protection "1; mode=block";
}


sudo ln -s /etc/nginx/sites-available/parahyangandigital.cloud /etc/nginx/sites-enabled/