docker swarm init

echo "rootPass$123!" | docker secret create root_pwd -
echo "wpPass$123!" | docker secret create wp_pwd -

Créez nginx.conf :

docker-compose.yml :

docker run -it --rm --name swarmpit-installer --volume /var/run/docker.sock:/var/run/docker.sock swarmpit/install:1.9

docker stack deploy -c docker-compose.yml wordpress_stack
