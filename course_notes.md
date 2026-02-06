
# Data Engineering Zoomcamp - Course Notes

## Description

This document contains comprehensive notes from the Data Engineering Zoomcamp course, featuring useful command-line tools and techniques for data engineering workflows. These notes serve as a quick reference guide for essential commands and best practices throughout the course.


# Docker

-- list container including the ones down
docker ps -a

-- delete all containers
docker rm $(docker ps -aq)

-- Create a postgresql db with a bind mount
-- Bind mount means that the volume have a specific path issue by the user. If not specified, the volume should be created in 
-- the default location: /var/lib/docker/volumes/
docker run -it -e POSTGRES_USER="root"   -e POSTGRES_PASSWORD="root"   -e POSTGRES_DB="ny_taxi" -v $(pwd)/ny_taxi_postgres_data:/var/lib/postgresql   -p 5432:5432   postgres:18

-- Create a virtual network for docker containers
docker network create pg-network

-- List virtual networks
docker network ls

-- Run postgresql with a virtual network
# Run PostgreSQL on the network
docker run -it \
  -e POSTGRES_USER="root" \
  -e POSTGRES_PASSWORD="root" \
  -e POSTGRES_DB="ny_taxi" \
  -v $(pwd)/ny_taxi_postgres_data:/var/lib/postgresql \
  -p 5432:5432 \
  --network=pg-network \
  --name pgdatabase \
  postgres:18

# Postgresql

-- install pgcli with uv
uv add --dev pgcli (not needed in production, for that the --dev flag mark)

-- connect to db with pgcli
uv run pgcli -h localhost -p 5432 -u root -d data_base_name

-- webide for postgresql: pgadmin
-- can run in a container like the db but need a virtual network so they can find each other
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