
# Data Engineering Zoomcamp - Course Notes

## Description

This document contains comprehensive notes from the Data Engineering Zoomcamp course, featuring useful command-line tools and techniques for data engineering workflows. These notes serve as a quick reference guide for essential commands and best practices throughout the course.


# Docker

-- list container including the ones down
docker ps -a

-- delete all containers
docker rm $(docker ps -aq)

-- Create a container for the first time
docker run

-- Run a existing container
docker start container_name_or_pid

-- Create a postgresql db with a bind mount
-- Bind mount means that the volume have a specific path issue by the user. If not specified, the volume should be created in 
-- the default location: /var/lib/docker/volumes/
docker run -it -e POSTGRES_USER="root"   -e POSTGRES_PASSWORD="root"   -e POSTGRES_DB="ny_taxi" -v $(pwd)/ny_taxi_postgres_data:/var/lib/postgresql   -p 5432:5432   postgres:18

-- Create a virtual network for docker containers
docker network create pg-network

-- List virtual networks
docker network ls

-- Run postgresql with a virtual network
docker run -it \
  -e POSTGRES_USER="root" \
  -e POSTGRES_PASSWORD="root" \
  -e POSTGRES_DB="ny_taxi" \
  -v $(pwd)/ny_taxi_postgres_data:/var/lib/postgresql \
  -p 5432:5432 \
  --network=pg-network \
  --name pgdatabase \
  postgres:18

default volume: pgadmin_data:/var/lib/pgadmin \
bind volume: $(pwd)/ny_taxi_postgres_data:/var/lib/postgresql

-- Build dockerfile
docker build -f pipeline/Dockerfile -t taxi_ingest:v001 .

-- Remove all stopped containers
docker container prune

-- List all images
docker images

-- Remove specific image
docker rmi taxi_ingest:v001

-- Remove all unused images
docker image prune -a

-- List volumes
docker volume ls

-- Remove specific volumes
docker volume rm ny_taxi_postgres_data

-- Remove all unused volumes
docker volume prune

-- Remove all unused networks
docker network prune

-- ⚠️ Warning: This removes ALL Docker resources!
docker system prune -a --volumes

-- Remove parquet files
rm *.parquet

-- Remove Python cache
rm -rf __pycache__ .pytest_cache

-- Remove virtual environment (if using venv)
rm -rf .venv

# Docker Compose

* docker-compose allows us to launch multiple containers using a single configuration file, so that we don't have to run multiple complex docker run commands separately.
* Makes use of YAML files
* We don't have to specify a network because docker compose takes care of it: every single container (or "service", as the file states) will run within the same network and will be able to find each other according to their names (pgdatabase and pgadmin in this example).
* All other details from the docker run commands (environment variables, volumes and ports) are mentioned accordingly in the file following YAML syntax.

-- Start services
docker-compose up

-- detached mode
docker-compose up -d

-- stop services
docker-compose down

docker-compose down: Detiene Y elimina los contenedores + la red + volúmenes (opcional)
docker stop {container}:	Solo detiene el contenedor, y lo mantiene. no lo borra

-- Userful commands
-- View logs
docker-compose logs

-- Stop and remove volumes
docker-compose down -v

If you want to re-run the dockerized ingest script when you run Postgres and pgAdmin with docker compose, you will have to find the name of the virtual network that Docker compose created for the containers

# Postgresql

-- install pgcli with uv
uv add --dev pgcli (not needed in production, for that the --dev flag mark)

-- connect to db with pgcli
uv run pgcli -h localhost -p 5432 -u root -d data_base_name

-- webide for postgresql: pgadmin
-- can run in a container like the db but need a virtual network so they can find each other
-- NOTE: to add the server connection in the pgadmin UI
the server should be the container name of the db
docker run -it \
  -e PGADMIN_DEFAULT_EMAIL="admin@admin.com" \
  -e PGADMIN_DEFAULT_PASSWORD="root" \
  -e PGADMIN_CONFIG_ENHANCED_COOKIE_PROTECTION="False" \
  -e PGADMIN_CONFIG_WTF_CSRF_ENABLED="False" \
  -e PGADMIN_CONFIG_WTF_CSRF_SSL_STRICT="False" \
  -e PGADMIN_CONFIG_WTF_CSRF_TIME_LIMIT="None" \
  -e PGADMIN_CONFIG_PROXY_X_FOR_COUNT=1 \
  -e PGADMIN_CONFIG_PROXY_X_PROTO_COUNT=1 \
  -e PGADMIN_CONFIG_PROXY_X_HOST_COUNT=1 \
  -e PGADMIN_CONFIG_PROXY_X_PORT_COUNT=1 \
  -e PGADMIN_CONFIG_PROXY_X_PREFIX_COUNT=1 \
  -v pgadmin_data:/var/lib/pgadmin \
  -p 8085:80 \
  --network=pg-network \
  --name pgadmin \
  dpage/pgadmin4




# Jupyter

-- install jupyter
uv add --dev jupyter

-- run
uv run jupyter notebook

-- convert notebook to script
uv run jupyter nbconvert --to=script notebook.ipynb

# Python

-- install with uv
uv add package_name

-- progress bar
uv add tqdm
"""
    from tqdm.auto import tqdm
    for df_chunk in tqdm(df_iter):
"""

-- command line arguments
import click

-- run containerized ingestion

* We need to provide the network for Docker to find the Postgres container. It goes before the name of the image.
* Since Postgres is running on a separate container, the host argument will have to point to the container name of Postgres (pgdatabase).
* You can drop the table in pgAdmin beforehand if you want, but the script will automatically replace the pre-existing table.

docker run -it \
  --network=pg-network \
  taxi_ingest:v001 \
    --pg-user=root \
    --pg-pass=root \
    --pg-host=pgdatabase \
    --pg-port=5432 \
    --pg-db=ny_taxi \
    --target-table=yellow_taxi_trips