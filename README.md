# Dockerizing Virtuoso

To simplify deployment, Virtuoso Open Source Edition can be run in a [Docker](https://www.docker.com) container.

## Building a VOS Base Image

Two VOS images are available:

* A base image which provides a compiled, installed VOS binary.
* A deployment image, built from the base image, which starts a VOS instance on the specified ports using the given VOS virtuoso.ini configuration file and, optionally, an existing Virtuoso database.

To build the base image:

    docker build -f Dockerfile.vos_base -t openlink/vos_base:v0 .
    
Dockerfile.vos_base builds VOS from the VOS GitHub sources, as described in the [VOS Wiki](http://virtuoso.openlinksw.com/dataspace/doc/dav/wiki/Main/VOSUbuntuNotes#Building%20from%20Upstream%20Source), and installs VOS in /opt/virtuoso-opensource.

## Building a VOS Deployment Image

To build the deployment image:

    docker build -f Dockerfile.vos -t openlink/vos:v0 .
    
The image created by Dockerfile.vos runs Virtuoso in the foreground and assumes Virtuoso listens for HTTP connections on port 8890 and SQL connections on port 1111. 

## Running VOS within Docker

### Initializing a new database

In order to retain changes to the Virtuoso database, the database should be held in the host file system. The database location on the host should reflect the installation directory used by the base image. Create directory /opt/virtuoso-opensource/var/lib/virtuoso/db in the host file system and provide a virtuoso.ini configuration file.

    sudo mkdir -p /opt/virtuoso-opensource/var/lib/virtuoso/db
    sudo cp ./virtuoso.ini.template /opt/virtuoso-opensource/var/lib/virtuoso/db/virtuoso.ini
    
virtuoso.ini.template assumes the installation directory is /opt/virtuoso-opensource, with the Virtuoso HTTP server listening on port 8890 and SQL client connections made through port 1111.

### Starting the VOS Container

Start a VOS container by running:

    sudo docker run -v /opt/virtuoso-opensource/var/lib/virtuoso/db:/opt/virtuoso-opensource/var/lib/virtuoso/db -t -p 1111:1111 -p 8890:8890 -i openlink/vos:v0
    
If the db directory contains only a virtuoso.ini file, a new database will be created when the container is started for the first time. All subsequent changes to the database will be persisted to the host file system.

### Using an existing database

If the db directory in the host file system contains an existing Virtuoso database, that database will be used by the container. Again, all subsequent changes to the database will be persisted to the host file system.

### Installing the Cartridges VAD

The Virtuoso Sponger and its associated transformers are distributed as a VAD (Virtuoso Application Distribution). When a new database instance is created, VOS will automatically install the Virtuoso Conductor UI. However it will not automatically install a Cartridges VAD.

A variant of the standard Cartridges VAD, containing Fusepool P3 specific transformers and dataset specific XSLT stylesheets for sample FP3 datasets is available for [download](http://opldownload.s3.amazonaws.com/uda/vad-packages/fusepool-p3/cartridges_dav.vad). This VAD is downloaded as part of the VOS base image build and included in the image at /opt/virtuoso-opensource/share/virtuoso/vad. After initializing a VOS instance, the FP3-specific cartridges VAD can be installed by logging into the Virtuoso Conductor (`http://{container_host}:8890/conductor`) and navigating to the 'System Admin' > 'Packages' tab and clicking on the 'Install' link. Once installed, enable the appropriate transformers through the 'Linked Data' > 'Sponger' > 'Extractor Cartridges' panel.
