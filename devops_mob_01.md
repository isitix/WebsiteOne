# Devops mob 20200525 : running WSO as a docker service

## Identifying the required files

| File | Role |
|----|-----------|
| Dockerfile | Websiteone front end image definition |
| docker-compose.yml | Docker service definition, both front end and back end, with the web app and the database server (postgres) |
| docker-entrypoint.sh | The entry point is the script which runs when the image goes up. It is configured in the Dockerfile. The entrypoint checks that every gem in the Gemfile.lock is up-to-date and installed. If it's not installed, it does the installation. Then, the entrypoint executes other commands sent by the user on the command line. See <https://stackoverflow.com/questions/39082768/what-does-set-e-and-exec-do-for-docker-entrypoint-scripts> |
| docker/README.md | To run the service architecture defined in compose, you need an environment file .env that you get from someone who knows it. Read the Readme to know more about it. R the documentation here <https://docs.docker.com/compose/environment-variables/> to know more about environment variables. |
| docker/setup.sh | This script removes images and volumes before creating a new environment from scratch <https://docs.docker.com/compose/reference/down/> and removes intermediate containers while creating the target image <https://docs.docker.com/compose/reference/build/> and then sets up the development and test databases. |
| docker/start.sh | This script only executes docker-compose down then up |
| docker/stop.sh | This script only executes docker-compose down |

## Docker-compose.yml file

docker-compose is a docker tools which lets you compose containers to create a service with, for instance, a back-end database and front-end web application, each one running in a separate container. 

docker-compose offers services to ease the process :

- a internal docker netwrok between the containers which compose the service
- a name service base on service names so that containers can communicate between each others without knowing their IP addresses
- simple orchestration rules (dependencies) so that when a container depends on another one, the dependent container starts after the required container
- data persistency using docker volumes


docker-compose is built upon a docker-compose file. The file defines the service configuration. To run the service, you run "docker-compose up" in the directory where docker-compose.yml file resides.

WSO docker-compose file has the following characteristics: 

- version: '3' : the version of the docker-compose file. Please use a docker-compose version that is compatible with the docker-compose file version as mentioned in this document <https://docs.docker.com/compose/compose-file/compose-versioning/>

Main point about version 3 : it's compatible with both docker-compose and swarm.

- Two services in WSO service-stack :

1. DB

- Built upon a standard postgres image
- A volume is created to store postgres data. The volume is mounted in the default data directory on the container

2. Web

- web container image is built from the Dockerfile
- When ran, web container runs rails app using bundle exec

The web container depends on the db container. It uses docker name service and docker default network to communicate with the db container.

4 volumes are configured for the web container : 

- The first volume is bind-mounted <https://docs.docker.com/storage/bind-mounts/>
- The three others are regular ones
