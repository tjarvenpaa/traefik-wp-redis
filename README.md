# traefik-wp-redis
Docker compose for Traefik front Wordpress server with redis accelerator

docker-compose.yml tiedostoon vaihdettava host nimi wordpress asennukselle.

Palvelu ajetaan ylös käskyllä docker compose up -d

Tämä edellyttää docker enginen ja docker compose pluginin asentamista. 

Docker Convenience script hoitaa homman:
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
