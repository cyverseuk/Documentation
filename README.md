#CyverseUK App Documentation

###**_this page is under construction and is NOT complete_**

###Creating a Docker Image

To create a Docker Image you will need to download and install <a href=https://www.docker.com/products/overview>Docker</a> for your distribution.
The suggestion is to follow the tutorial available on the website to get started.

For CyverseUK Docker images we will start with a Linux distribution (`FROM` command), ideally the suitable one that provides the smallest base image.  
The `LABEL` field provides the image metadata, in our case software/package`.version`.  
The `USER` will be `root` by default.  
The `RUN` instruction execute the following commands installing the wanted software and all its dependencies. As suggested by the official Docker documentation the best practice is to write all the commands in the same `RUN` instruction (this is also true for any other instruction), to keep the number of layers to a minimum.  
The `WORKDIR` instruction sets the working directory (`/data/` for my images).

If needed the following instructions may be found:
`ADD/COPY` to add file/data/software to the image. Ideally the source will be a link or repository publicly available.
`ENV` set environmental variables.
`MANTAINER` reports author of the image.
`ENTRYPOINT` may provide a default command to run when starting a new container.

###Run a Container

If running a container locally we often want to run it in interactive mode:
```docker run -ti <image_name>```
If the interactive mode is not needed don't use the `-i` option.  
In case the image is not available locally, Docker will try to dowload it from <a href=https://hub.docker.com/>DockerHub</a>.

To use data available on the local machine we may need to use a volume. The `-v <source_dir>:<image_dir>` option given by command line will mount `source_dir` in the host to `image_dir` in the docker container. Any change will affect the host directory too.

###More on Docker
######_useful commands and tricks_

Remove orpahned volumes from Docker:  
```sudo docker volume ls -f dangling=true | awk '{print $2}' | tail -n +2 | xargs sudo docker volume rm```  
Remove all containers:  
```docker ps -a | awk '{print $1}' | tail -n +2 | xargs docker rm```  
Remove dangling images (i.e. untagged): (to avoid errors due to images being used by containers, remove the containers first)  
```docker images -qf dangling=true | xargs docker rmi```   
Remove dangling images AND the first container that is using them, if any: (may need to be run more than once)
```docker images -qf dangling=true | xargs docker rmi 2>&1 | awk '$1=="Error" {print$NF}' | xargs docker rm```

<hr>

###Registering an App

###Running an App
