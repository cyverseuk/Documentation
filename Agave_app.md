## Register and run apps with Agave API

Index:
* <a href="#registering">Registering an App</a>
    * <a href="#json">Additional notes on the JSON file</a>
    * <a href="#general">Docker integration</a>
    * <a href="#condor">Condor integration</a>
* <a href="#publishing">Publishing an App</a>
* <a href="#running">Running an App from command line</a>

### <dev id="registering">Registering an App</dev>

To write and register an App I suggest reading <a href=https://github.com/cyverseuk/cyverseuk-util/blob/master/app_tutorial/agaveapps.md>this tutorial</a>.  
Note that it's not possible to register an App if there is already one with the same name. You can delete the previous one (if there was an error), or change the version number (if you need to make an updated version).  

The wrapper script should perform all the checks that the Agave API doesn't support (mutually inclusive or exclusive parameters for example), and ideally return the proper error before running the Docker container. It may be useful to use the wrapper script to delete any new files that is not needed from the working directory, to avoid them to be archived.  
In our case there is some additional logic in the wrapper scripts to allow some automatatic tasks in the virtual machines to perform as expected and to integrate the system with the webapp (<a href="http://cyverseuk.herokuapp.com/">CyVerseUk</a>). You don't usually have to worry about this.  

##### <div id="json">Additional notes on the JSON file</div>

Following the introductory part the JSON file lists inputs and parameters. A good documentation about the available fields and their usage can be found <a href=http://agaveapi.co/documentation/tutorials/app-management-tutorial/app-inputs-and-parameters-tutorial/>here</a>.  
For the application (if you wish to publish it) to display a proper information window in the Discovery Environment, the following fields need to be present in the JSON file: `help_URI`, `datePublished`, `author`, `longDescription`.  
In the `ontology` field a list of IRI for topic and operation branches of the <a href=http://www.ebi.ac.uk/ols/ontologies/edam>EDAM ontology</a> has to be specified to properly categorize the App.  

May you encounter some problems registering your application, I'd suggest first checking the JSON file is valid. A good way to do this is to copy-paste it to <a href="https://togo.agaveapi.co/app/#/apps/new">AgaveToGo</a>.  

If `details.showArgument` (boolean) is set to `true`, it will pass `details.argument` before the value (e.g. if we want to pass to command line `--kmer 31`). Note that the argument is put before the value without spaces (so usually we want to add one in the string!!).  
`value.validator` can supply a check on the format of the submitted value as a <a href=http://perldoc.perl.org/perlre.html>perl formatted regular expression</a>. (**pay particular attention to the escapes**)  
Example case: JSON `value.type` doesn't provide a distinction between integers and floating point, but just `number`. To check the input is an integer we may use `"validator": "^\\d*$"` (or `"^[0-9]+$"` to avoid the escapes). The same field also allow to accept just even/odd numbers, set a maximum value, etc.  
*Also note that it may be useful to define numerical variables as strings providing the right validator if we don't want to define a default value, because both the Discovery Environment and the CyVerseUk web interface will pass 0 otherwise.*  
We usually don't want the user to work in a folder that is not the set working directory, so if the program run by the App has a `--output_directory` option (or similar) we may want to add a validator to be sure that the string doesn't start with '/', or just hide it and give a default name (e.g. `output`, this will also make the wrapper script easier to write and maintain).

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

#### <dev id="general">Docker integration</dev>

It's not possible to run an App in CyVerse interactively. Therefore to run multiple commands in a Docker container we need the following syntax in the `wrapper.sh` script:  
```
docker run <image_name[:tag]> /bin/bash -c "command1;command2...;".
```  
`/bin/bash` is not strictly necessary, but, depending on the base image, bash may not be the default shell: adding it to command line takes care of this problem.  

**IMPORTANT UPDATE**: in Docker version 1.12 the `SHELL` instruction was added. This allows the default shell used for the shell form of commands to be overridden (at build time too-so it may make the built a bit slower). Use it as follows:  
`SHELL ["/bin/bash", "-c"]`

#### <div id="condor">Condor integration</div>

The HPC on CyVerseUk infrastructure is using HTCondor scheduler, so the `wrapper.sh` is not enough to run the app, but a `HTCondorSubmit.htc` script is needed as well.  
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
This HTCondor submit has to be generated by the `wrapper.sh` since we can't know in advance arguments and inputs files.  
`transfer_output_files` is not needed if the output is in the working directory. A good idea is to create, when possible, all the output files in a subdirectory (e.g. `output`) of the working directory, so that the transfer is easier.  
If transferring executables in `transfer_input_files`, make sure to restore the right permissions in the wrapper script (e.g. `chmod u+x <file_name>`).  
It's also possible that the Docker image has to be updated giving 777 permissions to scripts because of how Condor handle Docker.  

###<div id="publishing">Publishing an App</div>

The App, after being made public with (this step has to be performed by a tenant admin, so please contact them if you have a ready-to-publish application):  
```
apps-pems-update -v -u <username> -p ALL <app_name>-<version>
```  
can be found both in the <a href=https://de.iplantcollaborative.org/de/>DE</a>, under Apps>High-Performance Computing, and in the <a href="https://cyverseuk.herokuapp.com/">CyVerseUk</a> web interface. The App interface is automatically generated based on the submitted JSON file.

###<div id="running">Running an App from command line</div>

Again see a full tutorial <a href=https://github.com/cyverseuk/cyverseuk-util/blob/master/app_tutorial/agaveapps.md>here</a>.

<hr/>

