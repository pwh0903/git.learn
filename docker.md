# Build image
1. Make a directory and Write Dockerfile
FROM docker/whalesay:latest
RUN apt-get -y update && apt-get install -y fortunes
CMD /usr/games/fortune -a | cowsay

2. build image with Dockerfile
docker build -t docker-whale .


# Push image to docker hub
1. create repo on docker hub
eg. 
jiujiupeter/docker-whale

2. tag image on local
docker tag <image_id> jiujiupeter/docker-whale:latest

3. login docker hub
docker login

3. push image 
docker push jiujiupeter/docker-whale



FROM <image>
FROM <image>:<tag>



RUN <command>
RUN ["executable", "param1", "param2"]



CMD ["executable", "param1", "param2"]
CMD ["param1", "param2"]	# default parameters to ENTRYPOINT
CMD command param1 param2



LABEL <key>=<value> <key>=<value> <key>=<value> ...



# Container listen these ports, but not accessible to the host
# use -p or -P to publish
EXPOSE <port> [<port>...]



ENV <key> <value>
ENV <key>=<value> ...



# copy new files from src to dest
ADD <src>... <dest>
ADD ["<src>",... "<dest>"]


# use COPY not ADD at most time 
COPY <src>... <dest>
COPY ["<src>",... "<dest>"]



ENTRYPOINT
