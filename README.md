#CyverseUK App Documentation

###**_this page is under construction and is NOT complete_**

###Creating a Docker Image

To create a Docker Image you will need to download and install <a href=https://www.docker.com/products/overview>Docker</a> for your distribution.
The suggestion is to follow the tutorial available on the website to get started.

For CyverseUK Docker images we will start with a Linux distribution (`FROM` command), ideally the suitable one that provides the smallest base image.
The `LABEL` field provides the image metadata, in our case software/package`.version`.
The `USER` will be `root` by default.
The `RUN` instruction execute the following commands installing the wanted software and all its dependencies.
The `WORKDIR` instruction 

If needed the following instructions may be found:
`ADD/COPY` to add file/data/software to the image. Ideally the source will be a link or repository publicly available.
`ENV` set environmental variables.
`MANTAINER` reports author of the image.

###Run a Container

###More on Docker
######_useful commands and tricks_

<hr>

###Registering an App

###Running an App
