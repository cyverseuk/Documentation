Index:
* <a href="#overview">Very short Docker Overview</a>
* <a href="#creating_image">Creating a Docker Image</a>
* <a href="#build_image">Build a Docker Image</a>
* <a href="#run">Run a Container</a>
* <a href="#more">More on Docker</a>

###<div id="overview">Very short Docker Overview</div>  

Docker containers are similar to lightweight virtual machines, but have a different architecture and are organized in layers.  
For our purposes here we care about them to be lightweight and to provide the user with a virtually isolated environment that includes the application and all its dependencies. Working with Docker containers also enable portability between systems and allow us to provide users with an additional service, in case they don't wish to use Agave applications.  

###<div id="creating_image">Creating a Docker Image</div>

To create a Docker Image you will need to download and install <a href=https://www.docker.com/products/overview>Docker</a> for your distribution.
The suggestion is to follow the tutorial available on the website to get started.

For CyverseUK Docker images we will start with a Linux distribution (`FROM` command), ideally the suitable one that provides the smallest base image. For CyverseUK images the convention is to specify the tag of the base image too, to provide the user with a completely standard container.  
The instructions to build the image are written in a DockerFile.  
The `LABEL` field provides the image metadata, in CyverseUK software/package`.version` (note that we are *not* currently respecting the guideline of prefixing each label key with the reverse DNS notation of the cyverse domain). A list of labels and additional informations can be retrieved with ```docker inspect <image_name>```.  
The `USER` will be `root` by default for CyverseUK Docker images.  
The `RUN` instruction executes the following commands installing the wanted software and all its dependencies. As suggested by the official Docker documentation, the best practice is to write all the commands in the same `RUN` instruction (this is also true for any other instruction) separated by `&&`, to keep the number of layers to a minimum. Note that the building process is NOT interactive and the user is not able to answer the prompt, so use `-y` or `-yy` to run `apt-get update` and `apt-get install`. It is also possible to set `ARG DEBIAN_FRONTEND=noninteractive` to disable the prompt (`ARG` instruction set a variable _just_ at build time).  
The `WORKDIR` instruction sets the working directory (`/data/` for my images).

If needed the following instructions may be also present:
`ADD/COPY` to add file/data/software to the image. Ideally the source will be a link or repository publicly available. The difference between the two instructions is that the former can extract files and open URLs (so in CyverseUK will be preferred: however `ADD` DO NOT extract from an URL, the extraction will have to be explicitly performed in a second time). Also may worth to note that the official documentation recommends, when possible, to avoid `ADD` and use `wget` or `curl`.   
`ENV` set environmental variables. Note that it supports a few standard bash modifiers as the following:  
```bash
${variable:-word}
${variable:+word}
```  
`MANTAINER` is the author of the image. Note that in the meanwhile `MAINTAINER` have been deprecated, so from now on it will be listed as a key-value pair in `LABEL`.  
`ENTRYPOINT` may provide a default command to run when starting a new container making the docker image an executable.

###<div id="build_image">Build a Docker Image</div>

The easier way to build a Docker image once written the Dockerfile is to run the following command:
```
docker build -t image_name[:tag] path/to/Dockerfile
```
Each image can be provided at build time with a tag (default one is `latest`). (it's a good idea to have one Dockerfile per folder, hence you can run the previous command in ```.```)

####DockerHub and Automated Build

To make an image publicly available this needs to be uploaded in DockerHub. You have to create an account and follow the official documentation. To summarize use the following command:  
```
docker tag <image_ID> <DockerHub_username/image_name[:tag]>
```  
`<image_ID>` can be easily determined with `docker images`. Note that <DockerHub_username/image_name> needs to be manually created in DockerHub prior to the above command to be run.
CyverseUK Docker images can be found under the <a href=https://hub.docker.com/u/cyverseuk/>cyverseuk</a> organization.  
![dockerhub view of cyverse organization](https://raw.githubusercontent.com/cyverseuk/Documentation/master/media/dockerhub_view.png)  
We are using automated build, that allows to trigger a new build every time the linked GitHub repository is updated.  
Another useful feature of the automated build is to publicly display the DockerFile, allowing the user to know exactly how the image was built and what to expect from a container that is running it. GitHub `README.md` file is made into the Docker image long description.  
![view of a dockerfile available publicly on dockerhub](https://github.com/cyverseuk/Documentation/blob/master/media/dockerfile_ex.png?raw=true)  

For CyverseUK images when there is a change in the image, a new build with the same tag as the github release will be triggered to keep track of the different versions. At the same time also an update of the `:latest` tag is triggered (you need to manually add a rule for this to happen, it's not done automatically).

###<div id="run">Run a Container</div>

If running a container locally we often want to run it in interactive mode:  
```
docker run -ti <image_name>
```   
If the interactive mode is not needed don't use the `-i` option.  
In case the image is not available locally, Docker will try to download it from the register <a href=https://hub.docker.com/>DockerHub</a>.

To use data available on our local machine we may need to use a volume. The `-v <source_dir>:<image_dir>` option given by command line will mount `source_dir` in the host to `image_dir` in the docker container. Any change will affect the host directory too.  
It is possible to stop and keep using the same container in a second time as:  
```
docker start <container_name>
docker attach <container_name>
```

###<div id="more">More on Docker</div>
#####_useful commands and tricks_

* When writing a Dockerfile it is worth noticing the `source` command is not available as the default interpreter is `/bin/sh` (and not `/bin/bash`). A possible solution is to use the following command:  
  ```
  /bin/bash -c "source <whatever_needs_to_be_sourced>"
  ```

* See all existing containers:  
  ```
  docker ps -a
  ```   
* Remove orphaned volumes from Docker:  
  ```
  sudo docker volume ls -f dangling=true | awk '{print $2}' | tail -n +2 | xargs sudo docker volume rm
  ```  
* Remove all containers:  
  ```
  docker ps -a | awk '{print $1}' | tail -n +2 | xargs docker rm
  ```  
  To avoid accumulating containers it's also possible to run docker with the `--rm` option, that remove the container after the execution.  
* Remove dangling images (i.e. untagged): (to avoid errors due to images being used by containers, remove the containers first)  
  ```
  docker images -qf dangling=true | xargs docker rmi
  ```   
* Remove dangling images AND the first container that is using them, if any: (may need to be run more than once)  
  ```
  docker images -qf dangling=true | xargs docker rmi 2>&1 | awk '$1=="Error" {print$NF}' | xargs docker rm
  ```  
  To avoid running the above command multiple times I wrote <a href=https://raw.githubusercontent.com/aliceminotto/EarlhamInstitute_scripts/master/rmi_docker.sh>this script</a> (should work, no guarantees).   
* See the number of layers:  
  ```
  docker history <image_name> | tail -n +2 | wc -l
  ```  
* See the image size:  
  ```
  docker images <image_name> | tail -n +2 | awk '{print$(NF-1)" "$NF}'
  ```  

Other instructions than the ones listed here are available: `EXPOSE`, `VOLUME`, `STOPSIGNAL`, `CMD`, `ONBUILD`, `HEALTHCHECK`. These are usually not required for our purposes, but you can find more informations in the official Docker Documentation.

For previous docker versions <a href=https://imagelayers.io/>ImageLayers.io</a> used to provide the user with a number of functionalities. Badges were available to clearly display the number of layers and the size of the image (this can be very useful to know before downloading the image and running a container if time/resources are a limiting factor). We restored only this last feature with a bash script (<a href=https://github.com/aliceminotto/ImageInfo>ImageInfo</a>) that uses <a href=http://shields.io/>shields.io</a>.  

**IMPORTANT:** You may encounter problems when trying to build a Docker image or connect to internet from inside a container if you are on a local network. From the Docker Documentation:  
>...all localhost addresses on the host are unreachable from the container's network.  

To make it work:
* find out your DNS address  
  `nmcli dev list iface em1 | grep IP4.DNS | awk '{print $2}'`  
* *option 1 (easier and preferred)*: build the image and run the container with `--dns=<you_DNS_adress>`.  
* *option 2*: in the `RUN` instruction re-write the `/etc/resolv.conf` file to list your DNS as nameserver.  

**About the use of Docker universe on HTCondor**: the use of volumes (or Data Volumes Containers) is not enabled (yet????) (would require give permissions to specific folders, also is not clear if it mounts volume as only read -ok- or read and write -not so ok-), to get the same result we need to use `transfer_input_files` as from next section.  
It's also possible that the Docker image has to be updated giving 777 permissions to scripts because of how Condor handle Docker. 

<hr>
