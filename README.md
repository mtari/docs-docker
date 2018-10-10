#Docker 

##Installation
1. First we need to create an account on https://www.docker.com/
2. Download the client.
3. Install the client.
4. Login

##First run
Run command: 
```
docker run hello-world
```

This command is going to download and start hello-world image.

##Definitions
###Image
An image contains 2 things:
- start command
- section of the storage (codebase, config, dev environment, etc).

###Container
As a simple description: Container is an instance of image.
Contains the system and access hardware of a computer and an image.

##Commands
* `docker run <image-name>`
* `docker run <image-name> <command>`

  for example: `docker run busybox echo hi`
  
  This is going to start the image and run the echo command as a start command.