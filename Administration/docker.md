# run specific docker compose file (-f) using the provided env file (--env-file) in the background (-d)
docker-compose -f .\docker\docker-compose.yml --env-file .\.env up -d
 
# run services defined in the docker-compose.yml file in the background (-d)
docker-compose up -d
 
# stop and remove the containers, networks and volumes defined in docker-compose.yml file
docker-compose down
 
# view the logs of the containers created by Docker Compose
docker-compose logs
 
# display the logs from all the containers in real-time (-f or --follow)
docker-compose logs -f
 
# view the logs from a specific service
docker-compose logs db