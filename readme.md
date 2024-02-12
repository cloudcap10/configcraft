* [caddy_v2](caddy_v2/) - reverse proxy

Create a new docker network

docker network create dmz_net
docker network create internal_net

All the future containers and Caddy must be on this new network.

Can be named whatever you want, but it must be a new custom named network. Otherwise dns resolution would not work and containers would not be able to target each other just by the hostname.

- Files and directory structure
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


# Create directories that store your stacks and stores Dockge's stack
mkdir -p /opt/stacks /opt/dockge
cd /opt/dockge

# Download the compose.yaml
curl https://raw.githubusercontent.com/louislam/dockge/master/compose.yaml --output compose.yaml

# Start the server
docker compose up -d

# If you are using docker-compose V1 or Podman
# docker-compose up -d