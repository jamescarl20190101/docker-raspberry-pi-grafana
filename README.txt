# STOP!
# Use official image!
# REF: https://hub.docker.com/r/grafana/grafana-arm32v7-linux/tags
#docker pull grafana/grafana-arm32v7-linux:latest
docker run -d --name=grafana -p 3000:3000 grafana/grafana-arm32v7-linux:6.0.2
# http://YourRaspberryPiIpAddress:3000
# REF: https://github.com/grafana/grafana/tree/master/packaging/docker


# REF https://stackoverflow.com/questions/24481564/how-can-i-find-a-docker-image-with-a-specific-tag-in-docker-registry-on-the-dock
# To get a tag list, add this bash function
function list-dh-tags(){
    wget -q https://registry.hub.docker.com/v1/repositories/$1/tags -O -  | sed -e 's/[][]//g' -e 's/"//g' -e 's/ //g' | tr '}' '\n'  | awk -F: '{print $3}'
}

list-dh-tags grafana/grafana-arm32v7-linux
latest
5.4.3
6.0.0
6.0.0-beta1
6.0.0-beta2
6.0.0-beta3
6.0.1
6.0.2
master




####################################################
# Grafana (in a Docker Container) for Raspberry Pi #
#                        REF: https://grafana.com/ #
####################################################
# ARM build/download for Raspberry Pi:
REF: https://grafana.com/grafana/download/6.0.1?platform=arm

###############################################################################
# Init
mkdir -p /mnt/grafana/conf
mkdir -p /mnt/grafana/lib/grafana
# Docker build
sudo bash
time docker build --no-cache -t jamescarl20190101/docker-raspberry-pi-grafana:6.0.1 -f Dockerfile.armhf .
docker images

# Verify 
docker run -it -p 3000:3000 jamescarl20190101/docker-raspberry-pi-grafana:6.0.1
# From another ssh session:
#docker ps

# Upload to Docker Hub
docker login
docker push jamescarl20190101/docker-raspberry-pi-grafana:6.0.1
###############################################################################


###############################################################################
# First time setup #
####################
# Grafana (stand-alone for Docker Swarm)
# REF: https://hub.docker.com/r/grafana/grafana/
# and http://docs.grafana.org/installation/configuration/

# cert generation
openssl genrsa -out grafana.key 2048
openssl req -new -key grafana.key -out grafana.csr
openssl x509 -req -days 365 -in grafana.csr -signkey grafana.key -out grafana.crt

# Create the bind mount directories and copy files to the correct node based on constraints in the docker-compose!
mkdir -p /mnt/grafana
cp grafana.crt /mnt/grafana/
cp grafana.key /mnt/grafana/
cp ldap.toml /mnt/grafana/
chown -R 472:472 /mnt/grafana
chmod a+rw -R /mnt/grafana

##########
# Deploy #
##########
# Deploy the stack into a Docker Swarm
docker stack deploy -c docker-compose.yml grafana
# docker stack rm grafana

# Verify
docker service ls | grep grafana
docker service logs -f grafana
###############################################################################
