.. _config:

*************
Configuration
*************

Generally we at least have a folder on the host containing all the configuration, using a volume binding from the container.

This configuration often comes with a working example.

The reason to do this is to avoid rebuilding the image every time the configuration needs to be changed.

Mostly we use this structure::

    example
        etc
            start.sh
        run.sh
        run.ps1

The folder etc contains configurations and sometimes also the logs. The logs are mapped to this folder to make it
easier to read the logs on the host without needing to ssh to the container.

For more information about run.sh and run.ps1 view :ref:`run`.

start.sh is used when the server/software can be started in different ways depending on how you want your server to run.
