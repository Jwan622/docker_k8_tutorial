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


## New commands
## New Dockerfile commands
