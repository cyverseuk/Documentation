#CyverseUK App Documentation

###**_this page is under construction and is NOT complete_**

Notes and Documentation created for <a href=http://cyverseuk.org/>CyverseUK</a> at <a href=http://www.earlham.ac.uk/>Earlham Institute</a>.  
App for CyverseUK are dockerized, allowing the use of a standard environment and version control of packages and softwares used.

###Very short Docker overview

###Creating a Docker Image

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

###Build a Docker image

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
CyverseUK Docker images can be found under the <a href=https://hub.docker.com/u/cyverseuk/>cyverseuk</a> organization. We are using automated build, that allows to trigger a new build every time the linked GitHub repository is updated.  
Another useful feature of the automated build is to publicly display the DockerFile, allowing the user to know exactly how the image was built and what to expect from a container that is running it. GitHub `README.md` file is made into the Docker image long description.

For CyverseUK images when there is a change in the image, a new build with the same tag as the github release will be triggered to keep track of the different versions. At the same time also an update of the `:latest` tag is triggered (you need to manually add a rule for this to happen, it's not done automatically).

###Run a Container

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

###More on Docker
#####_useful commands and tricks_

When writing a Dockerfile it is worth noticing the `source` command is not available as the default interpreter is `/bin/sh` (and not `/bin/bash`). A possible solution is to use the following command:  
```
/bin/bash -c "source <whatever_needs_to_be_sourced>"
```

See all existing containers:  
```
docker ps -a
```   
Remove orphaned volumes from Docker:  
```
sudo docker volume ls -f dangling=true | awk '{print $2}' | tail -n +2 | xargs sudo docker volume rm
```  
Remove all containers:  
```
docker ps -a | awk '{print $1}' | tail -n +2 | xargs docker rm
```  
To avoid the accumulation of containers is also possible to run docker with the `--rm` option, that remove the container after the execution.  
Remove dangling images (i.e. untagged): (to avoid errors due to images being used by containers, remove the containers first)  
```
docker images -qf dangling=true | xargs docker rmi
```   
Remove dangling images AND the first container that is using them, if any: (may need to be run more than once)  
```
docker images -qf dangling=true | xargs docker rmi 2>&1 | awk '$1=="Error" {print$NF}' | xargs docker rm
```  
To avoid running the above command multiple times I wrote <a href=https://raw.githubusercontent.com/aliceminotto/EarlhamInstitute_scripts/master/rmi_docker.sh>this script</a> (should work, no guarantees).   
See the number of layers:  
```
docker history <image_name> | tail -n +2 | wc -l
```  
See the image size:  
```
docker images <image_name> | tail -n +2 | awk '{print$(NF-1)" "$NF}'
```  

Other instructions than the ones listed here are available: `EXPOSE`, `VOLUME`, `STOPSIGNAL`, `CMD`, `ONBUILD`, `HEALTHCHECK`. These are usually not required for our purposes, but you can find more informations in the official Docker Documentation.

For previous docker versions <a href=https://imagelayers.io/>ImageLayers.io</a> used to provide the user with a number of functionalities. Badges were available to clearly display the number of layers and the size of the image (this can be very useful to know before downloading the image and running a container if time/resources are a limiting factor). We restored only this last feature with a bash script (<a href=https://github.com/aliceminotto/ImageInfo>ImageInfo</a>) that uses <a href=http://shields.io/>shields.io</a>.  

About the use of Docker universe on HTCondor: the use of volumes (or Data Volumes Containers) is not enabled (yet????) (would require give permissions to specific folders, also is not clear if it mounts volume as only read -ok- or read and write -not so ok-), to get the same result we need to use `transfer_input_files` as from next section.  

**IMPORTANT:** You may encounter problems when trying to build a Docker image or connect to internet from inside a container if you are on a local network. From the Docker Documentation:  
>...all localhost addresses on the host are unreachable from the container's network.  

To make it work:
* find out your DNS address  
  `nmcli dev list iface em1 | grep IP4.DNS | awk '{print $2}'`  
* option 1 (easier and preferred): build the image and run the container with `--dns=<you_DNS_adress>`.  
* option 2: in the `RUN` instruction re-write the `/etc/resolv.conf` file to list your DNS as nameserver.

<hr>

##Registering an App

To write and register an App I suggest reading <a href=https://github.com/cyverseuk/cyverseuk-util/blob/master/app_tutorial/agaveapps.md>this tutorial</a>.  
Note that it's not possible to register an App if there is already one with the same name. You can delete the previous one (if there was an error), or change the version number (if you need to make an updated version).  

Also, it's not possible (of course) to run an App in Cyverse interactively. To run multiple commands in a Docker container we need the following syntax:  
```
docker run <image_name[:tag]> /bin/bash -c "command1;command2...;".
```  
`/bin/bash` is not strictly necessary, but, depending on the base image, bash may not be the default shell: adding it to command line takes care of this problem.  

**IMPORTANT UPDATE**: in Docker version 1.12 the `SHELL` instruction was added. This allows the default shell used for the shell form of commands to be overridden (at build time too). Use it as follows:  
`SHELL ["/bin/bash", "-c"]`

The HPC is using HTCondor, so the JSON file alone is not enough to run the app: a `wrapper.sh` script (to handle the variables and determine the actual command to be run) and a `HTCondorSubmit.htc` script are needed. The wrapper script has to perform all the checks that the Agave API doesn't support (mutually inclusive or exclusive parameters for example). It may be useful to use the wrapper script to delete any new files that is not needed from the working directory, to avoid it to be transferred back to the DE.  
The HTCondorSubmit.htc file will be in the following form:
```
universe                = docker
docker_image            = <image_name>[:tag]
executable              = <command>
should_transfer_files   = YES
arguments               = <arguments_for_executable>
transfer_input_files    = <files to transfer separated by commas>
transfer_output_files   = <files/folder to transfer separated by commas>
when_to_transfer_output = ON_EXIT
request_memory          = 100G
```  
`transfer_output_files` is not needed if the output are in the working directory. Of course this HTCondor submit is generated by the wrapper.sh since we can't know in advance arguments and inputs files.  
If transferring executables, make sure to restore the right permissions in the script (e.g `chmod u+x <file_name>`).  
It's also possible that the Docker image has to be updated giving 777 permissions to scripts because of how condor handle docker.  

A good idea is to create, when possible, all the output files in a subdirectory (e.g. `output`) of the working directory, so that the transfer is easier.  

The App, after being made public with (this step has to be performed by a tenant admin)  
```
apps-pems-update -v -u <username> -p ALL <app_name>-<version>
```  
can be found in the <a href=https://de.iplantcollaborative.org/de/>DE</a>, under Apps>High-Performance Computing. The App interface is automatically generated based on the submitted JSON file.

#####Additional notes on the JSON file

Following the introductory part the JSON file lists inputs and parameters. A good documentation about the available fields and their usage can be found <a href=http://agaveapi.co/documentation/tutorials/app-management-tutorial/app-inputs-and-parameters-tutorial/>here</a>.  
For the application (if you wish to publish it) to display a proper information window in the Discovery Environment, the following fields need to be present in the JSON file: `help_URI`, `datePublished`, `author`, `longDescription`.  
In the `ontology` field a list of IRI for topic and operation branches of the <a href=http://www.ebi.ac.uk/ols/ontologies/edam>EDAM ontology</a> has to be specified to properly categorize the App.  
If `details.showArgument` (boolean) is set to `true`, it will pass `details.argument` before the value (e.g. if we want to pass to command line `--kmer 31`). Note that the argument is put before the value without spaces (so usually we want to add one in the string)!!  
`value.validator` can supply a check on the format of the submitted value as a <a href=http://perldoc.perl.org/perlre.html>perl formatted regular expression</a>. (**pay particular attention to the escapes**) Example case: JSON `value.type` doesn't provide a distinction between integers and floating point, but just `number`. To check the input is an integer we may use `"validator": "^\\d*$"` (or `"^[0-9]+$"` to avoid the escapes). The same field also allow to accept just even/odd numbers, set a maximum value, etc.  
We usually don't want the user to work in a folder that is not the set working directory, so if the program run by the App has a `--output_directory` option (or similar) we may want to add a validator to be sure that the string doesn't start with '/', or just hide it and give a default name (e.g. `ouput`).

**IMPORTANT**:  
```json
"value": {
    ...
    "visible": false,
    "default": "default_value",
    ...
}
```  
is NOT supported. The default value must be provided in the wrapper script if we don't want the user to be able to change it.  

###Running an App

Again see a full tutorial <a href=https://github.com/cyverseuk/cyverseuk-util/blob/master/app_tutorial/agaveapps.md>here</a>.


