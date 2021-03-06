#Docker 

Installation
------------
1. First we need to create an account on https://www.docker.com/
2. Download the client.
3. Install the client.
4. Login



Docker hub
----------
[https://hub.docker.com/](https://hub.docker.com/)
Docker hub is where we can browse other images, manage access and images, etc.



First run
---------
Run command: 
```
docker run hello-world
```

This command is going to download and start hello-world image.



Definitions
-----------
###Image
An image contains 2 things:

* start command

* section of the storage (codebase, config, dev environment, etc).

###Container
As a simple description: Container is an instance of image.

Contains the system and access hardware of a computer and an image.



Commands
--------
###`docker run <image-name>`

###`docker run <image-name> <command>`
for example: `docker run busybox echo hi`

This is going to start the image and run the echo command as a start command.

This command is a combination of 2 other commands:
```
docker create busybox
docker start <id> -a
```

###`docker ps`
This command shows the state of currently running container.

###`docker ps --all`
This command shows the list of containers.

###`docker system prune`
This command removes the history of executed containers and the cache which where saved for the containers.

###`docker logs <id>`
Shows the logs belonging to the container. This doesn't run it, it only shows the previous logs.

###`docker stop <id>` or `docker kill <id>`
Stop running container.

###`docker exec -it <id> <command>`
Execute a command on a container.

###`docker exec -it <id> sh`
Gives terminal access to the running container.



Creating docker images
----------------------

Dockerfile -> Docker Client -> Docker Server -> Usable Image!

Flow of 'Creating a dockerfile':
1. Specify a base image
2. Run some commands to install additional programs
3. Specify a command to run on container startup

###Example
1. Created [docker file](/redis-image/Dockerfile) in redis-image folder.
2. Run `docker build .` command to build th container.
3. Run `docker run <id>` to run the container.

###Tagging docker image
`docker build -t mihalytari/redis:latest .`
In here 'mihalytari' is the docker userid, redis is the name and latest (version) is the tag.

###Node.js web app + building container
```
# specify a base image
FROM node:alpine

WORKDIR /usr/app

# Install dependencies
COPY ./package.json ./
RUN npm install
COPY ./ ./

# Default command
CMD [ "npm", "start" ];
```

Then build the image `docker build -t mihalytari/nodejs .`

We need to use port mapping to forward the request to the container.
Then run the container `docker run -p 8080:8080 mihalytari/nodejs`

### Building custom Doackerfile
`docker build -f Dockerfile.dev -t mihalytari/monitoring .`



Docker compose with multiple local containers
---------------------------------------------
###`docker-compose up`
Use it to run containers which don't require building the image. Basically this runs multiple `docker run myimage` commands.
Otherwise run `docker-compose up --build` command.

### `docker-compose up -d`
Launch containers in the background.

### `docker-compose down`
Stop containers which where running.

###Restart policies
We can add restart policy, to restart a container after it's crashed and stopped for some reason.
Options: no, always, on-failure, unless-stopped

Usage: `restart: always`



Development flow
----------------

###Running dev Dockerfile:
`docker run -f Dockerfile.dev .`

###Using watcher to update app when a change is saved
We need to change the `docker run` command to the next:
`docker run -p 3000:3000 -v "<WORKDIR>/node_modules" -v "$(pwd):/<WORKDIR>" <image id>`

This maps the folder to our local folder, so we don't need to rebuild the miage every time when we make a change in our files.

Or we can write docker-compose file to simplify the command we need to execute. In this case the content of a docker-compose.yml file looks like:
```
version: '3'
services: 
  react-image:
    build: .
    ports:
      - "3000:3000"
    volumes:
      - /app/node_modules
      - .:/app
```

###Running a command in a already running container
`docker exec -it <container id> <command>`

###`docker attach` command
This attaches the stdin, stdout, stderr to the container, meaning we can type input commands to the container.

###Using nginx
nginx is going to serve our static files in production environment.

###Implementing multi-step builds
```
FROM node:alpine as builder
WORKDIR '/app'
COPY package.json .
RUN npm install
COPY . .
RUN npm run build

FROM nginx
COPY --from=builder /app/build /usr/share/nginx/html
```

##Services overview
In this section we use github + travis + aws to deploy our application.
The Dockerfile.dev and the Dockerfile is the same as above.

The .travis.yml content
```
sudo: required
services:
  - docker

before_install:
  - docker build -t mihalytari/react-image -f Dockerfile.dev .

script:
  - docker run mihalytari/react-image npm run test -- --coverage

deploy:
  provider: elasticbeanstalk
  region: "us-east-2"
  app: "react-image"
  env: "ReactImage-env"
  bucket_name: "elasticbeanstalk-us-east-2-151932697359"
  bucket_path: "react-image"
  on:
    branch: master
  access_key_id: $AWS_ACCESS_KEY
  secret_access_key:
    secure: "$AWS_SECRET_KEY"
```

Dockerfile's content
```
FROM node:alpine as builder
WORKDIR '/app'
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

FROM nginx
EXPOSE 80
COPY --from=builder /app/build /usr/share/nginx/html
```

Docker-compose content
```
version: '3'
services: 
  react-image:
    build: 
      context: .
      dockerfile: Dockerfile.dev
    ports:
      - "3000:3000"
    volumes:
      - /app/node_modules
      - .:/app
  react-image-tests:
    build: 
      context: .
      dockerfile: Dockerfile.dev
    volumes:
      - /app/node_modules
      - .:/app
    command: ["npm", "run", "test"]
```
