# La API resuelve el nombre del servicio de base de datos dentro de la red

172.20.0.2

# Persistencia: apagar y volver a levantar conserva los datos

docker compose down:

Container app-deploy-docker-proxy-1
Container app-deploy-docker-api-1
Container app-deploy-docker-db-1
Network app-deploy-docker_interna

Removed

docker compose up -d:

Container app-deploy-docker-proxy-1 Started
Container app-deploy-docker-api-1 Started
Container app-deploy-docker-db-1 Healthy
Network app-deploy-docker_interna Created
Image app-deploy-docker-api Built

# Aislamiento: la base de datos NO debe estar publicada al exterior

docker compose port db 5432 || echo

Invalid IP:0

curl --max-time 3 http://localhost:5432 || echo

curl: (7) Failed to connect to localhost port 5432 after 0 ms: Couldn't connect to server
