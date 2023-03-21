The following explains how to use XNAT´s Container-Service with the goal to run automatic pipelines like DICOM to BIDS conversion and the integration of MRI-Data quality assessment with the tool MRI-QC (https://mriqc.readthedocs.io/en/latest).

#__1) Docker-Installation on Debian 10__

To install Docker I followed the steps from https://linuxize.com/post/how-to-install-and-use-docker-on-debian-10/

>Install the packages necessary to add a new repository over HTTPS:
>```bash
>sudo apt update
>sudo apt install apt-transport-https ca-certificates curl software-properties-common gnupg2
>```
>Import the repository’s GPG key using the following curl command :
>```bash
>curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
>```
>On success, the command will return OK.
>
>Add the stable Docker APT repository to your system’s software repository list:
>```bash
>sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/debian $(lsb_release -cs) stable"
>```
>$(lsb_release -cs) will return the name of the Debian distribution . In this case, that is buster.
>
>Update the apt package list and install the latest version of Docker CE (Community Edition):
>```bash
>sudo apt update
>sudo apt install docker-ce
>```
>Once the installation is completed the Docker service will start automatically. To verify it type in:
>
>```bash
>sudo systemctl status docker
>```





#__2) Installation of Container PlugIn for XNAT__ 
