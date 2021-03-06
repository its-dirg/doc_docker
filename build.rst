***********
Build image
***********

For building we use a script called build.sh. We use it to simplifying the build process, specially if you run boot2docker.

For the script to work you need to setup your Dockerfile confguration in a folder called dockerfiles.


dockerfiles
===========

In the docker project we always have a folder called dockerfiles with usually this structure::

    dockerfiles
        build.sh
        build.ps1
        Dockerfile
        requirements.txt
        start.sh

build.sh
--------
Responsible for containing the name of the repository/image and simplifying the use of docker and boot2docker.


Example of the script::

    #!/bin/bash

    #Get's the directory for the script build.sh
    dir=$(dirname `which $0`)

    #Name of the repository/image
    repository="itsdirg/pefim_sp"

    # Check if running on mac
    if [ $(uname) = "Darwin" ]; then

        # Check so the boot2docker vm is running
        if [ $(boot2docker status) != "running" ]; then
            boot2docker start
        fi
        #Initiate boot2docker, so you can run docker commands.
        $(boot2docker shellinit)
    else
        # if running on linux
        if [ $(id -u) -ne 0 ]; then
            sudo="sudo"
        fi
    fi
    #Remove the docker image before building a new.
    ${sudo} docker rmi -f ${repository}
    #Build a new docker image
    ${sudo} docker build -t=${repository} ${dir}

build.ps1
---------
Powershell script version of build.sh for windows users.::


    Write-Host "Start build"
    #Name of the repository/image
    $repository = "itsdirg/pefim_sp"

    $ssh_path = "c:\Program Files (x86)\Git\bin"
    IF (!($Env:Path | Select-String -SimpleMatch $ssh_path)){
        $Env:Path = "${Env:Path};$ssh_path"
    }

    # Check so the boot2docker vm is running
    IF ($(Boot2Docker status) -ne "running"){
        $(Boot2Docker start)
    }

    #Initiate boot2docker, so you can run docker commands.
    IF ($Env:DOCKER_HOST.length -eq 0){
        boot2docker shellinit | Invoke-Expression
    }

    #Remove windows CR (avoid file issues between windows and linux/unix
    $files = gci ../ *.* -File -rec | where {! $_.ps1} | %{$_.FullName}
    foreach ($file in $files){
        sed -i 's/\r//' ${file}
    }

    #Remove the docker image before building a new.
    docker rmi -f "${repository}" 2>&1> $null;
    #Build a new docker image
    docker build -t="${repository}" .

    Write-Host "End build"


Dockerfile
----------
You have to read the documentation of docker, we will not explain how you write a Dockerfile.

We will bind one or more volumes for configurations files, logs and other resources that should we want to keep/change on the host.::

    #DIRG base image (only used for some projects)
    FROM itsdirg/dirg_base

    MAINTAINER DIRG <dirg@its.umu.se>

    VOLUME ["/opt/my_project_folder/etc"]

    ADD requirements.txt /opt/my_project_folder/requirements.txt

    RUN apt-get update
    RUN apt-get install -y --no-install-recommends\
            libsasl2-dev \
            libldap2-dev \
            libssl-dev \
            xmlsec1
    RUN apt-get clean
    RUN rm -rf /var/lib/apt/lists/*

    RUN pip install --upgrade pip
    RUN pip install -r /opt/pefim/requirements.txt
    #The project to be installed
    RUN pip install git+https://github.com/its-dirg/my_project.git#egg=my_project

    ADD start.sh /start.sh

    WORKDIR /

    CMD ["bash", "/start.sh"]

requirements.txt
----------------
This is specific for our python projects and we use it to install the dependencies we have to other python project.


start.sh
--------
This script is responsible for starting the server/software in the docker container. This script often runs a start.sh
script that is added to the volume folder on the host, since how the server is started often depends on how you want
 to configure it.






