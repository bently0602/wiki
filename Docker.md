docker login accureg.azurecr.io --username NAME --password "" 

docker pull NAME.azurecr.io/IMAGENAME:latest  

 

# find container id by name 

docker ps -aqf "name=^IMAGENAME1$" 

 

# stop container by name 

docker container stop $(docker container ls -q --filter name=NAME*) 

 

docker container rm NAME 

sudo docker run --name NAME (-d) --restart unless-stopped -p 0.0.0.0:5000:80 NAME.azurecr.io/NAME:latest 

 

docker logs -f --until=2s test 

 

From <https://docs.docker.com/engine/reference/commandline/logs/>  

docker exec -it NAME sh 

-p 8080:80 Map TCP port 80 in the container to port 8080 on the Docker host. 

-p 192.168.1.100:8080:80 Map TCP port 80 in the container to port 8080 on the Docker host for connections to host IP 192.168.1.100. 

-p 8080:80/udp Map UDP port 80 in the container to port 8080 on the Docker host. 

-p 8080:80/tcp -p 8080:80/udp Map TCP port 80 in the container to TCP port 8080 on the Docker host, and map UDP port 80 in the container to UDP port 8080 on the Docker host. 
