#CyverseUK App Documentation

###**_this page is under construction and is NOT complete_**

Notes and Documentation created for <a href=http://cyverseuk.org/>CyverseUK</a> at <a href=http://www.earlham.ac.uk/>Earlham Institute</a>.  
App for CyverseUK are dockerized, allowing the use of a standard environment and version control of packages and softwares used.

###Creating a Docker Image

To create a Docker Image you will need to download and install <a href=https://www.docker.com/products/overview>Docker</a> for your distribution.
The suggestion is to follow the tutorial available on the website to get started.

For CyverseUK Docker images we will start with a Linux distribution (`FROM` command), ideally the suitable one that provides the smallest base image. For CyverseUK images the convention is to specify the tag of the base image too, to provide the user with a completly standard container.  
The instruction to build the image are written in a DockerFile.  
The `LABEL` field provides the image metadata, in CyverseUK software/package`.version`.  
The `USER` will be `root` by default for CyverseUK Docker images.  
The `RUN` instruction execute the following commands installing the wanted software and all its dependencies. As suggested by the official Docker documentation the best practice is to write all the commands in the same `RUN` instruction (this is also true for any other instruction) separated by `&&`, to keep the number of layers to a minimum. Note that the building process is NOT interactive and the user is not able to answer the prompt, so use `-y` or `-yy` to run `apt-get update` and `apt-get install`. It is also possible to set `ENV DEBIAN_FRONTEND=noninteractive` to disable the prompt.  
The `WORKDIR` instruction sets the working directory (`/data/` for my images).

If needed the following instructions may be found:
`ADD/COPY` to add file/data/software to the image. Ideally the source will be a link or repository publicly available. The difference between the two instructions is that the former can extract files and open URLs (so in CyverseUK will be preferred: however `ADD` DO NOT extract from an URL, the extraction will have to be explicitly permformed in a second time.   
`ENV` set environmental variables.  
`MANTAINER` reports author of the image.  
`ENTRYPOINT` may provide a default command to run when starting a new container making the docker image an executable.

###Run a Container

If running a container locally we often want to run it in interactive mode:
```docker run -ti <image_name>```
If the interactive mode is not needed don't use the `-i` option.  
In case the image is not available locally, Docker will try to download it from <a href=https://hub.docker.com/>DockerHub</a>.

To use data available on the local machine we may need to use a volume. The `-v <source_dir>:<image_dir>` option given by command line will mount `source_dir` in the host to `image_dir` in the docker container. Any change will affect the host directory too.  
It is possible to stop and keep using the same container in a second time as:  
```docker start <container_name>
docker attach <container_name>```

###DockerHub and Automated Build

To make an image publicly available this needs to be uploaded in DockerHub. You will have to create an account and follow the official documentation. To summarize use the following command:  
```docker tag <image_ID> <DockerHub_username/image_name[:tag]>```  
`<image_ID>` can be easily determined with `docker images`. Note that <DockerHub_username/image_name> needs to be manually created in DockerHub prior to the above command to be run.
For CyverseUK Docker images we are using automated build, that allows to trigger a new build every time the linked GitHub repository is updated.  
Another useful feature of the automated build is publicing displayed the DockerFile, allowing the user to know exactly how the image was built and what to expect from a container that is running it.
Each image can be provided at build time with a tag (deafult one is `latest`). To do so type `docker build -t image_name:tag path/to/Dockerfile`.  

For CyverseUk images when there is a known change in the image, a new build with the date as tag will be manually triggered to keep track of the different versions.

###More on Docker
######_useful commands and tricks_

See all existing containers:  
```docker ps -a```
Remove orpahned volumes from Docker:  
```sudo docker volume ls -f dangling=true | awk '{print $2}' | tail -n +2 | xargs sudo docker volume rm```  
Remove all containers:  
```docker ps -a | awk '{print $1}' | tail -n +2 | xargs docker rm```  
To avoid the accumulation of containers is also possible to run docker with the `--rm` option, that remove the container when the execution ends.  
Remove dangling images (i.e. untagged): (to avoid errors due to images being used by containers, remove the containers first)  
```docker images -qf dangling=true | xargs docker rmi```   
Remove dangling images AND the first container that is using them, if any: (may need to be run more than once)  
```docker images -qf dangling=true | xargs docker rmi 2>&1 | awk '$1=="Error" {print$NF}' | xargs docker rm```

For previous docker versions <a href=https://imagelayers.io/>ImageLayers.io</a> used to provide the user with dieffrent functionalities. Badges were available to clearly displayed the size and the number of layers of the image (this can be very useful to know before downloading the image and running a container if time/resources are a limit factor). We restored only this last feature with a bash script (<a href=https://github.com/aliceminotto/ImageInfo>ImageInfo</a>) that uses <a href=http://shields.io/>shields.io</a>.

<hr>

###Registering an App

To write and register an App I suggest to read <a href=https://github.com/cyverseuk/cyverseuk-util/blob/master/app_tutorial/agaveapps.md>this tutorial</a>.  
Note that it's not possible to register an App if there is already one with the same name. You can delete the previous one, or change the version number.  

Also, it's not possible (of course) to run an App in Cyverse interactively. To run a Docker container and give it multiple commands we need the following synthax:  
```docker run <image_name[:tag]> /bin/bash -c "command1;command2...;".```  
`/bin/bash` is not strictly necessary, but depending on the base image bash may not be the default shell: adding it to command line would remove this problem.

#####Additional notes on the JSON file

Following the introductory part the JSON file lists inputs and parameters. A good documentation about the available fields and their usage can be found <a href=http://agaveapi.co/documentation/tutorials/app-management-tutorial/app-inputs-and-parameters-tutorial/>here</a>.

###Running an App
