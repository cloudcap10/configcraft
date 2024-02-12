## About The Project

Incorporating [dockge](https://github.com/louislam/dockge), a Docker compose.yaml stack-oriented manager, this project further streamlines the deployment and management of containerized applications. Dockge introduces an intuitive layer for handling Docker Compose files, enabling users to effortlessly create, edit, and manage their service stacks. With Caddy serving as the primary entry point for traffic routing, Dockge complements this setup by offering a centralized platform for managing the underlying Docker Compose configurations. This enhances operational efficiency, simplifies the process of scaling services, and provides a unified interface for overseeing the entire Dockerized environment. The combination of Caddy's reverse proxy capabilities with Dockge's stack management tools presents a robust solution for maintaining a secure, scalable, and well-organized container ecosystem.

# Caddy as a reverse proxy in docker

Caddy will be deployed within a Docker container acting as a reverse proxy, with ports 80 and 443 accessible to the public through DMZ. Its role includes routing traffic to other containers or network devices. Importantly, only this container will be exposed publicly; all other internal communications should occur through Docker Compose's networking capabilities. It's crucial to utilize the `expose` parameter for other containers, ensuring they are only accessible internally within the Docker network.

### - Create a new docker network

`docker network create dmz_net`<br>
`docker network create internal_net`

All the future containers and Caddy must be on `internal_net` network.
  
Can be named whatever you want, but it must be a new custom named network.
Otherwise [dns resolution would not work](https://docs.docker.com/network/drivers/bridge/)
and containers would not be able to target each other just by the hostname.

- Files and directory structure
```
/opt/
└── /
    └── stacks/
        └── caddy/
            ├── 🗋 .env
            └── 🗋 docker-compose.yml
/mnt/
└── /
    └── docker-volumes
            ├── caddy
                ├── 🗁 caddy_config/
                ├── 🗁 caddy_data/
                ├── 🗋 Caddyfile
```
* `caddy_config/` - a directory containing configs that Caddy generates,
  most notably `autosave.json` which is a backup of the last loaded config
* `caddy_data/` - a directory storing TLS certificates
* `.env` - a file containing environment variables for docker compose
* `Caddyfile` - Caddy configuration file
* `docker-compose.yml` - a docker compose file, telling docker how to run containers

You only need to provide the three files.<br>
The directories are created by docker compose on the first run, 
the content of these is visible only as root of the docker host.

### - Create docker-compose.yml and .env file

Basic simple docker compose, using the official caddy image.<br>
Ports 80 and 443 are pusblished/mapped on to docker host as Caddy
is the one in charge of any traffic coming there.<br>

`docker-compose.yml`
```yml
version: "3.5"
services:
  caddy:
    image: caddy:2.7.6-alpine
    container_name: caddy_dmz
    restart: always
    env_file: .env
    ports:
      - 80:80
      - 443:443
      - 443:443/udp
    volumes:
      - $DOCKER_VOLUME_STORAGE/caddy/Caddyfile:/etc/caddy/Caddyfile
      - $DOCKER_VOLUME_STORAGE/caddy/caddy_data:/data
      - $DOCKER_VOLUME_STORAGE/caddy/caddy_config:/config
      - $DOCKER_VOLUME_STORAGE/caddy/caddy_logs:/var/log/caddy
    logging:
      driver: json-file
      options:
        max-size: 1M
        max-file: "10"
    networks:
      dmz_net:
      default:
networks:
  dmz_net:
    name: $DOCKER_DMZ_NETWORK
    external: true
  default:
    name: $DOCKER_INTERNAL_NETWORK
    external: true
```

`.env`
```php
# GENERAL
TZ=Europe/Bratislava
DOCKER_MY_NETWORK=caddy_net
MY_DOMAIN=example.com
```

You obviously want to change `example.com` to your domain.


### - Create Caddyfile

`Caddyfile`
```
a.{$MY_DOMAIN} {
    reverse_proxy whoami:80
}

b.{$MY_DOMAIN} {
    reverse_proxy nginx:80
}
```

`a` and `b` are the subdomains, can be named whatever.<br>
For them to work they **must have type-A DNS record set**, that points
at your public ip set on Cloudflare, or wherever the domains DNS is managed.<br>

# Create directories that store your stacks and stores Dockge's stack
mkdir -p /opt/stacks /opt/dockge
cd /opt/dockge

# Download the compose.yaml
curl https://raw.githubusercontent.com/louislam/dockge/master/compose.yaml --output compose.yaml

# Start the server
docker compose up -d

# If you are using docker-compose V1 or Podman
# docker-compose up -d