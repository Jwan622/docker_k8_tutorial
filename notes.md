# Udemy docker and k8 course



## Section 1

**Docker how it works**:

the __docker run__ command looks like local images first and if it can’t find them, looks to docker hub.

Docker makes use of a container, which you think of as a allocated amount of system reosurces devoted to a process or set of process.

The kernel looks at the system call and looks at the calling application and directs it to a segment of the computer resources. Called namespacing. Namespacing allows you to devote hardware resource to a the calling process. It isolates resources per process.

A control group can limit the amount of hard resources a process can use. It can limit how much cpu and memory a process can use. even network bandwidth.

control group and namespacing together can really affect processes. A container from docker utilizes both of these. a container is a grouping of processes that have a grouping of resources assigned to it.

hello
The kernel is repsonsible for looking at the process system call and directing it to the necessary resources.

this is what happens when you run docker run.
![there](images/docker_run.png)

and this is how a container would work
![container](images/docker_container.png)


# Section 3

## Notes
When running docker build, each step in run inside and intermediate container and the output of a step is a new image that is used for the next step. At every step, you output an image with a new file system and new command

## New commands
instead of pasting an id everytime, use an ID or tag using the -t flag.

```
docker build -t jwan/redis:latest .
```
- the trailing . is the build context for `build` command... source of all files and folders to be used to build container out.
-the above command tags an image. now we can run it using the tag.

```
docker build -t jwan/redis:latest
```

Basic Dockerfile:
From alpine
Run apk add --update redis
CMD ["redis-server"]


## New Dockerfile commands






# Section 4

the : specifies an image tag:

```
FROM node:alpine

RUN npm install

CMD ["npm", "start"]
```

"alpine" is a tag, not a repository in this case. alpine means the most bare version of a repo, small and compact as possible. no preinstalled programs like git or text editing.

If you have extra files that need to be stored in the container, you have to move them there yourself.
For ex, package.json file has to be moved into the container if your docker command or RUN is npm install. So add this to your Dockerfile using the COPY commmand:

```
FROM node:alpine

# Install some dependencies
COPY ./ ./
RUN npm install

# default command
CMD ["npm", "start"]
```


## Notes


#### TAgging
tag alpine means smallest image possible. Tags come after a colon after the repo name.

- copy command moves files from our machine to temp container during build process.
COPY ./ ./folder moves everything in currenet working directory on computer to ./folder in container.
the first argument to COPY is a path on our computer, the second is a path in the container.

#### setup port mapping
- by default no traffic into our machine goes into the container, have to setup port mapping for that to happen. Container has its own isolated set of ports, but by default no traffic to our comp is directed into the container. Port mapping -anytime a request is made to local network is forwarded to container.
- specified during runtime during the run command.
if anyone makes an incoming local network request to some port inside container:
- containers can reach out (clearly because it can install) but traffic into the container is limited.
- no limitation of container to reach out to the world, just a limitation to reach into the container.

```
docker run -p 8080:8080 <image_name_or_id>
```

port forwarding command
![port_mapping1](images/section_4/port_mapping1.png)

This is conceptually what the run command is doing:


![images/section_4/port_mapping](images/section_4/port_mapping.png)

- the second number is the port inside the container. first number is localhost.
- ports can also be different.


#### working directory

- instead of copying all files into the top level, you can specify a nested working directory to avoid conflicts with the top level directories.
- the instruction is WORKDIR into the Dockerfile. any following commands will be executed relative to this folder. So 
WORKDIR /usr/app
COPY ./ ./

that will create /usr/app and then copy will copy everything from computer directory root to /usr/app.

#### prevent unnecessary rebuilds:
```
FROM node:alpine

# makes future commands relative to this folder
WORKDIR /usr/app

# Install some dependencies
COPY ./package.json ./
RUN npm install
COPY ./ ./

# default command
CMD ["npm", "start"]
```

the COPY command looks at local package.json specified by the build context argument of `docker run` and copies them over to the WORKDIR. Notice the second argument just has to be ./ and not ./package.json
- by splitting copy up into two commands, any changes to index.js will only bust cache of second COPY and npm install will not run again unless the package.json changes.


## Section 4 commands

runs a shell after running an image:
```
docker run -it <image_name> sh
```


-to run a second command inside a process, in this case a sh shell: (-it is for interactive)
```
docker exec -it <container_id> sh
```

## Section 4 new Dockerfile commands
WORKDIR - any future commands in dockerfile will be relative to this folder that we specify. So maybe we don't put copy our files into the root directory of the container.
-workdir also affects commands that affect the container issued through command line like docker exec -it <container_id> sh... it sh's into the folder. It affects commands that are exectued through the `exec` command.

```
FROM node:alpine

# makes future commands relative to this folder
WORKDIR /usr/app

# Install some dependencies
COPY ./ ./
RUN npm install

# default command
CMD ["npm", "start"]
```




# Section 5

## Notes

- So imagine if you put node and redis in the same image. Say the app keeps track of visits. The problem is thatif this scales and you have to spin up more containers, each container has its own separate and siloed redis server. So the number of visits would be siloed and no one redis would have the true count of all visits. That's a problem. Instead you should separate out the redis and node containers so you can scale the node server but not the redis server.

- when running `docker run`, no need to put in the tag like latest.

- if you run the node application and the redis server in separate containers, these two containers have no automatic communication between the two. isolated processes with no communication. So... setup network infrastructure, you have two options: 1) use docker cli that can setup a network between two containers, it's a pain in the ass 2) docker compose. docker compose is a separate CLI and is installed. docker compose exists to help you not repeat repetitive commands with docker CLI. Docker compose can also very eailsy startup multiple docker containers and auto connect them with some kind of networking, it functions as teh CLI and issues multiple commands quickly.

- `docker-compose.yml` file has special syntax and the file will be fed into docker-compose CLI and will setup the containers with the configs.

- services in docker-compose world means container, sort of. It's a type of contianer really called a service.

```yml
version: '3'
services:
  redis-server:
    image: 'redis'
  node-app:
    build: .
    ports:
      - "4001:8080"
```

the above builds a container using an image from docker hub and a Dockerfile locally.
- the array is a dash because there can be multiple port mappings. the first number 4001 is a port on our local machine, 8080 is the port in the container. 

-docker compose will create the containers on the same network so the two can exchange info without having to open ports between the two. the port declaration in node-app is for our machine to access the container, but not for the containers to access each other.


- So how do we access redis server from node js code? in our index.js, this code:

```javascript
const client = redis.createClient({
    host: 'redis-server'
});
```

will be intercepted by Docker and it will handle the resolution of that request.


- restart policy for docker-compose. No, always, on-failure, unless-stopped.

## New commands

`docker run <image>` is replaced by `docker-compose up`. `docker-compose up` will create all the services or images in our `docker-compose.yml` file.

`docker build .` and `docker run <image>` is replaced by `docker-compose up --build`

`docker-compose up -d` starts up group of containers in the background
`docker-compose down` stops all of the containers.

`docker-compose ps` is like docker ps but needs to be run from directory with docker-compose.yml directory.

## New Dockerfile commands





# Section 6

## Notes

- How can you make changes to the source code to cleverly show up in the container without having to stop the container, and rebuild it, and restart it. The container doesn't update when you make changes to local code. Docker volumes - setup a placeholder in the container. IT has references instead of folders. the placeholders point back to local machine. Volume can be thought of as port mappings but to local resources.

- Use a new command to do this: docker run -p 3000:3000 -v $(pwd):/app <image_id>

## New commands

`docker build -f Dockerfile.dev .` can specify a new dockerfile to build from.


## New Dockerfile commands


