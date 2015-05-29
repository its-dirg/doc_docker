.. _run:

*******************
Running a container
*******************

To make it easier and quicker to run a container we have created run.sh/run.ps1 script to automatically bind the
configuration to the container and solve the boot2docker issues for the user.

run.sh
======

Both run script are examples with port forwarding for boot2docker (not used for linux offcourse). If you don't want
port forwarding you can use the docker IP instead.

Add this flag to docker run command::

    -e HOST_IP=$(boot2docker ip)

You can get the ip by running $(boot2docker ip) in the terminal.

Run script for linux and OS X that uses port forwarding to the virutal environment for boot2docker.::

    #!/bin/bash
    #
    # Use this line to start your container if you want to debug it:
    #   DOCKERARGS="--entrypoint /bin/bash" bash -x ./run.sh

    #Path to return to after the script is executed. Please improve! :)
    c_path=$(pwd)
    #Folder for the script.
    cddir=$(dirname `which $0`)
    #Go to the script folder.
    cd $cddir

    # Name of config file
    conf=service_conf.py

    # Image name
    image=itsdirg/pefim_sp

    # Name of container
    name=pefim_sp

    # Relative path to volume
    volume=etc

    #This is usable for most of DIRG applications.
    #Finds the server port in the configuration file to be forwarded to the container.
    #For boot2docker it's also used to forward the port to the virtual environment.
    port=$(cat ${volume}/${conf} | grep PORT | grep -v "#" | head -1 | sed 's/[^0-9]//g')

    dir=$(pwd)

    #Fix so docker works on Cent OS and Redhat. :)
    centos_or_redhat=$(cat /etc/centos-release 2>/dev/null | wc -l)
    if [ ${centos_or_redhat} = 1 ]; then
        $(chcon -Rt svirt_sandbox_file_t ${dir}/${volume})
    fi

    # Check if running on mac
    if [ $(uname) = "Darwin" ]; then

        #Forward the server port to the virtual environment.
        port_check=$(netstat -an | grep ${port} | wc -l)
        port_b2d=$(VBoxManage showvminfo "boot2docker-vm" --details | grep ${port} | wc -l)

        if [ ${port_b2d} = 0 ]; then
            if [ ${port_check} = 0 ]; then
                port_b2d=1
                VBoxManage controlvm "boot2docker-vm" natpf1 "${name},tcp,127.0.0.1,${port},,${port}"
            else
                echo "Port: " ${port} " is already used! Change port in the files " ${conf}
                exit 1
            fi
        fi

        # Check so the boot2docker vm is running
        if [ $(boot2docker status) != "running" ]; then
            boot2docker start
        fi
        $(boot2docker shellinit)
    else
        # if running on linux
        if [ $(id -u) -ne 0 ]; then
            sudo="sudo"
        fi
    fi

    if ${sudo} docker ps | awk '{print $NF}' | grep -qx ${name}; then
        echo "$0: Docker container with name $name already running. Press enter to restart it, or ctrl+c to abort."
        read foo
        #Kill the container if it already is running.
        ${sudo} docker kill ${name}
    fi
    #Removing container if it exists.
    $sudo docker rm ${name} > /dev/null 2> /dev/null

    #Finally we are running the container. :)
    ${sudo} docker run --rm=true \
        --name ${name} \
        --hostname localhost \
        -v ${dir}/${volume}:/opt/pefim/etc \
        -p ${port}:${port} \
        $DOCKERARGS \
        -i -t \
        ${image}

    #Delete the boot2docker port forwarding
    if [ $(uname) = "Darwin" ] && [ ${port_b2d} = 1 ]; then
        VBoxManage controlvm "boot2docker-vm" natpf1 delete "${name}"
    fi

    #Return to starting path. Please improve if you can. :)
    cd $c_path




Run script for windows that uses port forwarding to the virutal environment for boot2docker.::

    param([switch] $debug)

    Write-Host "Start script"

    #Remove windows CR (avoid file issues between windows and linux/unix.
    $files = gci ./etc *.* -File -rec | where {! $_.ps1} | %{$_.FullName}
    foreach ($file in $files){
        sed -i 's/\r//' ${file}
    }

    # Name of config file
    $conf="service_conf.py"

    # Image name
    $image="itsdirg/pefim_sp"

    # Name of container
    $name="pefim_sp"

    # relative path to volume
    $volume="etc"

    $dir = Convert-Path .
    $dir = $dir -replace "C:", "/c"
    $dir = $dir -replace "\\", "/"

    #This is usable for most of DIRG applications.
    #Finds the server port in the configuration file to be forwarded to the container.
    #For boot2docker it's also used to forward the port to the virtual environment.
    $port = cat ${volume}/${conf} | grep PORT | head -1 | sed 's/[^0-9]//g'

    $ssh_path = "c:\Program Files (x86)\Git\bin"

    #Setting up the port forward.
    [int]$port_check=[convert]::ToInt32($(netstat -an | grep ${port} | wc -l))
    [int]$port_b2d=[convert]::ToInt32($(& "C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" showvminfo "boot2docker-vm" --details | grep ${port} | wc -l))
    IF ($port_b2d -eq 0){
        IF ($port_check -eq 0 ){
            $port_b2d=1
            & "C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" controlvm "boot2docker-vm" natpf1 "${name},tcp,127.0.0.1,${port},,${port}"
        }
        ELSE{
            Write-Host "Port: ${port} is already used! Change port in the file ${conf}"
            Exit
        }
    }

    IF (!($Env:Path | Select-String -SimpleMatch $ssh_path)){
        $Env:Path = "${Env:Path};$ssh_path"
    }

    #Make sure that boot2docker vm is running.
    IF ($(Boot2Docker status) -ne "running"){
        $(Boot2Docker start)
    }

    #Initiate boot2docker, so you can run docker commands.
    IF ($Env:DOCKER_HOST.length -eq 0){
        boot2docker shellinit | Invoke-Expression
    }

    #Kill the container if it's running.
    IF ($(docker ps | %{ $_.Split(' ')[0];} | grep ${name} | wc -l) -eq 1){
        docker kill ${name}
    }
    #Delete the container if it exists.
    docker rm ${name} 2>&1> $null;

    mkdir ./etc/logs 2>&1> $null;
    mkdir ./etc/db 2>&1> $null;

    $debug_args = ""

    IF ($debug){
        docker run --rm=true --name ${name} --hostname localhost -v ${dir}/${volume}:/opt/pefim/etc -p ${port}:${port} --entrypoint /bin/bash -i -t ${image}
    }
    ELSE{
        docker run --rm=true --name ${name} --hostname localhost -v ${dir}/${volume}:/opt/pefim/etc -p ${port}:${port} -i -t ${image}
    }

    #Removes all port forwarding to the virtual machine.
    IF ($port_b2d -eq 1){
        # delete port forwarding
        & "C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" controlvm "boot2docker-vm" natpf1 delete "${name}"
    }
    Write-Host "End script"
