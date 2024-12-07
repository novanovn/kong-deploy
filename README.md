# kong-deploy
docker compose -f docker-compose-https4.yaml run --rm kong kong migrations bootstrap

docker compose -f docker-compose-https4.yaml up -d


curl -k https://kong.parahyangandigital.cloud:8444/status
