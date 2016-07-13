#CyverseUK App Documentation

###**_this page is under construction and is NOT complete_**

Notes and Documentation created for <a href=http://cyverseuk.org/>CyverseUK</a> at <a href=http://www.earlham.ac.uk/>Earlham Institute</a>.  
App for CyverseUK are dockerized, allowing the use of a standard environment and version control of packages and softwares used.

###Creating a Docker Image

To create a Docker Image you will need to download and install <a href=https://www.docker.com/products/overview>Docker</a> for your distribution.
The suggestion is to follow the tutorial available on the website to get started.

For CyverseUK Docker images we will start with a Linux distribution (`FROM` command), ideally the suitable one that provides the smallest base image. For CyverseUK images the convention is to specify the tag of the base image too, to provide the user with a completely standard container.  
The instruction to build the image are written in a DockerFile.  
The `LABEL` field provides the image metadata, in CyverseUK software/package`.version`.  
The `USER` will be `root` by default for CyverseUK Docker images.  
The `RUN` instruction execute the following commands installing the wanted software and all its dependencies. As suggested by the official Docker documentation the best practice is to write all the commands in the same `RUN` instruction (this is also true for any other instruction) separated by `&&`, to keep the number of layers to a minimum. Note that the building process is NOT interactive and the user is not able to answer the prompt, so use `-y` or `-yy` to run `apt-get update` and `apt-get install`. It is also possible to set `ENV DEBIAN_FRONTEND=noninteractive` to disable the prompt.  
The `WORKDIR` instruction sets the working directory (`/data/` for my images).

If needed the following instructions may be found:
`ADD/COPY` to add file/data/software to the image. Ideally the source will be a link or repository publicly available. The difference between the two instructions is that the former can extract files and open URLs (so in CyverseUK will be preferred: however `ADD` DO NOT extract from an URL, the extraction will have to be explicitly performed in a second time).   
`ENV` set environmental variables. Note that it supports a few standard bash modifiers as the following:  
```bash
${variable:-word}
${variable:+word}
```  
`MANTAINER` is the author of the image.  
`ENTRYPOINT` may provide a default command to run when starting a new container making the docker image an executable.

###Run a Container

If running a container locally we often want to run it in interactive mode:  
```docker run -ti <image_name>```   
If the interactive mode is not needed don't use the `-i` option.  
In case the image is not available locally, Docker will try to download it from <a href=https://hub.docker.com/>DockerHub</a>.

To use data available on our local machine we may need to use a volume. The `-v <source_dir>:<image_dir>` option given by command line will mount `source_dir` in the host to `image_dir` in the docker container. Any change will affect the host directory too.  
It is possible to stop and keep using the same container in a second time as:  
```
docker start <container_name>
docker attach <container_name>
```

###DockerHub and Automated Build

To make an image publicly available this needs to be uploaded in DockerHub. You have to create an account and follow the official documentation. To summarize use the following command:  
```docker tag <image_ID> <DockerHub_username/image_name[:tag]>```  
`<image_ID>` can be easily determined with `docker images`. Note that <DockerHub_username/image_name> needs to be manually created in DockerHub prior to the above command to be run.
CyverseUK Docker images can be found under the <a href=https://hub.docker.com/u/cyverseuk/>cyverseuk</a> organization. We are using automated build, that allows to trigger a new build every time the linked GitHub repository is updated.  
Another useful feature of the automated build is to publicly display the DockerFile, allowing the user to know exactly how the image was built and what to expect from a container that is running it. GitHub `README.md` file is made into the Docker image long description.
Each image can be provided at build time with a tag (default one is `latest`). To do so type `docker build -t image_name[:tag] path/to/Dockerfile`.  

For CyverseUk images when there is a known change in the image, a new build with the date as tag will be manually triggered to keep track of the different versions. **Remember to save the new tag before triggering and save again `:latest` as default!!** (also note that any change in the linked GitHub repository will trigger a new built, not just a  change in the DockerFile)

###More on Docker
#####_useful commands and tricks_

See all existing containers:  
```docker ps -a```   
Remove orphaned volumes from Docker:  
```sudo docker volume ls -f dangling=true | awk '{print $2}' | tail -n +2 | xargs sudo docker volume rm```  
Remove all containers:  
```docker ps -a | awk '{print $1}' | tail -n +2 | xargs docker rm```  
To avoid the accumulation of containers is also possible to run docker with the `--rm` option, that remove the container after the execution.  
Remove dangling images (i.e. untagged): (to avoid errors due to images being used by containers, remove the containers first)  
```docker images -qf dangling=true | xargs docker rmi```   
Remove dangling images AND the first container that is using them, if any: (may need to be run more than once)  
```docker images -qf dangling=true | xargs docker rmi 2>&1 | awk '$1=="Error" {print$NF}' | xargs docker rm```  
See the number of layers:  
```docker history <image_name> | tail -n +2 | wc -l```  
See the image size:  
```docker images <image_name> | tail -n +2 | awk '{print$(NF-1)" "$NF}'```

For previous docker versions <a href=https://imagelayers.io/>ImageLayers.io</a> used to provide the user with a number of functionalities. Badges were available to clearly display the number of layers and the size of the image (this can be very useful to know before downloading the image and running a container if time/resources are a limiting factor). We restored only this last feature with a bash script (<a href=https://github.com/aliceminotto/ImageInfo>ImageInfo</a>) that uses <a href=http://shields.io/>shields.io</a>.  

About the use of Docker universe on HTCondor: the use of volumes (or Data Volumes Containers) is not enabled (yet????) (would require give permissions to specific folders, also is not clear if it mounts volume as only read -ok- or read and write -not so ok-), to get the same result we need to use `transfer_input_files` as from next section.

<hr>

###Registering an App

To write and register an App I suggest reading <a href=https://github.com/cyverseuk/cyverseuk-util/blob/master/app_tutorial/agaveapps.md>this tutorial</a>.  
Note that it's not possible to register an App if there is already one with the same name. You can delete the previous one (if there was an error), or change the version number (if you need to make an updated version).  

Also, it's not possible (of course) to run an App in Cyverse interactively. To run multiple commands in a Docker container we need the following syntax:  
```docker run <image_name[:tag]> /bin/bash -c "command1;command2...;".```  
`/bin/bash` is not strictly necessary, but, depending on the base image, bash may not be the default shell: adding it to command line takes care of this problem.  

The HPC is using HTCondor, so the JSON file alone is not enough to run the app: a wrapper.sh script (to handle the variables and determine the actual command to be run) and a HTCondorSubmit.htc script are needed. The wrapper script has to perform all the checks that the Agave API doesn't support (mutually inclusive or exclusive parameters for example). It may be useful to use the wrapper script to delete any new files that is not needed from the working directory, to avoid it to be transferred back to the DE.  
The HTCondorSubmit.htc file will be in the following form:
```
universe                = docker
docker_image            = <image_name>
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

A good idea is to create, when possible, all the output files in a subdirectory (e.g. `output`) of the working directory, so that the transfer is easier.  

The App, after being made public with  
```apps-pems-update -v -u <username> -p ALL <app_name>-<version>```  
can be found in the <a href=https://de.iplantcollaborative.org/de/>DE</a>, under Apps>High-Performance Computing. The App interface is automatically generated based on the submitted JSON file.

#####Additional notes on the JSON file

Following the introductory part the JSON file lists inputs and parameters. A good documentation about the available fields and their usage can be found <a href=http://agaveapi.co/documentation/tutorials/app-management-tutorial/app-inputs-and-parameters-tutorial/>here</a>.  
In the `ontology` field a list of IRI for topic and operation branches of the <a href=http://www.ebi.ac.uk/ols/ontologies/edam>EDAM ontology</a> has to be specified to properly categorize the App.  
If `details.showArgument` (boolean) is set to `true`, it will pass `details.argument` before the value (e.g. if we want to pass to command line `--kmer 31`). Note that the argument is put before the value without spaces (so usually we want to add one in the string)!!  
`value.validator` can supply a check on the format of the submitted value as a <a href=http://perldoc.perl.org/perlre.html>perl formatted regular expression</a>. (**pay particular attention to the escapes**) Example case: JSON `value.type` doesn't provide a distinction between integers and floating point, but just `number`. To check the input is an integer we may use `"validator": "^\d*$"`. The same field also allow to accept just even/odd numbers, set a maximum value, etc.  
We usually don't want the user to work in a folder that is not the set working directory, so if the program run by the App has a `--output_directory` option (or similar) we may want to add a validator to be sure that the string doesn't start with '/', or just hide it and give a default name (e.g. `ouput`).

IMPORTANT:  
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
