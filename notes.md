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

- the best thing about using docker compose and volumes is that they rerun when changes are made to the app files.
- multistep dockerfiles allow us to do a build with one image, and run with another like nginx.

## New commands

`docker build -f Dockerfile.dev .` can specify a new dockerfile to build from.

`docker run -p 3000:3000 -v /app/node_modules -v $(pwd):/app <image>` runs a docker container with an image that maps the present working directory files on local to the app folder in the container but it also doesn't touch the /app/node_modules folder and keeps a bookmark so it doesn't get overwritten. We can replace this with a docker compose command instead. Bookmarking is like holding onto the volume inside of the container so it doesn't get deleted.

To run tests:
`docker run -it <tag> npm run test` the -it flag hooks the container up to standard in so we can run npm test commands in the container when it prompts us. standard out is already hooked up by default.

to override command and run test and have it be interactive:
`docker run -it <image> npm run test`

`docker exec -it <container> sh`
- that executes an run a command in the container and starts a shell and a connection to standard in with -it.

`docker attach <container>`
- attaches to standard in and out of primary process.. always attaches to primary process of container, not sub processes like test suites. so that's why tehre's that shortcoming of testing in docker containers and why interactive commands don't work. we atatched to the wrong process in the container.

## New Dockerfile commands


- if you do `FROM node:alpine as builder`, it means everything after this will be known as the builder phase. If there's is another `FROM` in a Dockerfile, then it starts another phase and it completes the previous block.
- doing `COPY . .` is fine during the build phase because we won't be changing our source code further. We use volumes when we want to develop faster and see our code changes immediately. 

```Docker
FROM node:alpine as builder

WORKDIR '/app'

COPY package.json .

RUN npm install

COPY . .

RUN npm run build


FROM nginx
COPY --from=builder /app/build /usr/share/nginx/html

```
two phases, the second phase just throws out everything except for the build folder which is created when `npm run build` is run.
- `COPY --from=builder /app/build /usr/share/nginx/html` we copy everything from the builder phase from the /app/build folder to the appropriate nginx folder is used by nginx to serve static content. `COPY --from` specifies the phase builder.

-to run the above run `docker build .` and then `docker run -p 8080:80 fed9975d68c6`
- now we have a production application with nginx!


when we visit `localhost:8080` we see:

```text
docker run -p 8080:80 fed9975d68c6
172.17.0.1 - - [16/Dec/2019:16:18:39 +0000] "GET / HTTP/1.1" 200 2217 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/77.0.3865.120 Safari/537.36" "-"
172.17.0.1 - - [16/Dec/2019:16:18:39 +0000] "GET /static/js/2.dd1fcc93.chunk.js HTTP/1.1" 200 130053 "http://localhost:8080/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/77.0.3865.120 Safari/537.36" "-"
172.17.0.1 - - [16/Dec/2019:16:18:39 +0000] "GET /static/css/main.d1b05096.chunk.css HTTP/1.1" 200 1051 "http://localhost:8080/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/77.0.3865.120 Safari/537.36" "-"
172.17.0.1 - - [16/Dec/2019:16:18:39 +0000] "GET /static/js/main.649334a6.chunk.js HTTP/1.1" 200 1134 "http://localhost:8080/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/77.0.3865.120 Safari/537.36" "-"
```

# Section 7

## Notes

- my work for section 7 with travis and AWS is in git@github.com:Jwan622/docker-react.git
- Travis can pull github repo with new changes and do any work like test, deploy, delete it all if you want. Traditionally for testing and deployment to AWS.
- travis will use the Dockerfile.dev from section 6 because only it has the test suite. The production Dockerfile gets rid of it and only keeps the app/build folder.

- for travis to use docker, it needs superlevel permissions so `sudo:required` is in the `.travis.yml` file.
- in `.travis.yml`, the `services: - Docker` is to help Travis understand it needs Docker installed.
- AWS Elastic Beanstalk will scale everything up for us using a load balancer and it will spin up more VMs with our container  as traffic reaches a threshold. Requests go to the load balancer first.

- in `.traviss.yml`

```text

before_install:
  - docker build -f Dockerfile.dev .
```

the above tells to find the Dockerfile.dev in the current context with the .

- need to tag it because travis won't copy paste it for you. Use a tag
- the `.travis.yml` file has to be toplevel in teh github repo, mine was not and I was stuck for a while because I'm an idiot.

- So how do you have travis deploy to AWS once tests have pass? We can add to the `.travis.yml`

```text
deploy:
  provider: elasticbeanstalk
```

that helps travis deploy to AWS ESB.

- When travis deploys, it takes the files inside github repo, zips them, and sends them to s3, thats why we need to specify a `bucket_Name` in our travis deploy section. whebn it's uploaded, travis will poke ESB (elastic beanstalk) and say it's uplaoded, and ESB will then take the zip file from the bucket. That's why we need to specify the bucket_name in the travis file. Find the bucket in s3.

```text
deploy:
  provider: elasticbeanstalk
  region: "us-east-1"
  app: "docker-react"
  env: "Docker-env"
  bucket_name: "elasticbeanstalk-us-east-1-516088479088"
  bucket_path: "docker-react"
  on:
    branch: master
```

the bucket_path is the name of the path/folder that will be created in the bucket_name. It will created teh first time we deploy. the `on master` part tells travis to only deploy when the master branch is updated with new code, not feature branches.

__ON AWS ESB__
- an application is a common set of configs that houses environments but the environment is the actual application. 

__ON IAM__
- programmtic access in IAM is for when you never access it via the AWS console, jsut through network requests.
- you can add policies directly to user or to a group. when creating a user for travis, we can attach policy directly.
- we need to include AWS keys into travis CI because we're allowing travis CI to access resources in our AWS account. That's why we put AWS keys in our `.travis.yml` file.

- takeaway from this is that docker made deployment easier. In our `travis.yml` file, all we had to do was a before install and script that had a `docker build` and a `docker run` command that had an override to run tests. The rest of the file can be the same if you swap out a different image in the build step. So docker images simplify deployment because you just have to specify the image.
- I did get this all working!
## New commands
## New Dockerfile commands

`EXPOSE 80` on our laptop does nothing and it's more communication to developers. BUT AWS beanstalk will look for expose instruction and will ESB will port map automatically from the load balancer to this port.


# Section 8

## Notes

- we're building this:

![archiecture](images/section_8/architecture.png)

- a few things, redis is needed to store the index of the fibonacci sequence, not the values themselves. the worker pulls the index and calculates the fibonacci value slowly which is why the worker adn redis are needed. postgres will store the fib indices that have already been calculated. This is a multicontainer application that will have several containers talking to each other.

- problem with section 7 is that travis was building the image and so was AWS ESB (elastic beanstalk), why build the image twice? Our web server was building the image which seems like a nono.


## New commands
## New Dockerfile commands
