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
