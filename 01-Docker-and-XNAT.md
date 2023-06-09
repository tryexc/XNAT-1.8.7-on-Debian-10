The following explains how to use XNAT´s Container-Service with the goal to run automatic pipelines like DICOM to BIDS conversion and the integration of MRI-Data quality assessment with the tool MRI-QC (https://mriqc.readthedocs.io/en/latest).

# __1) Docker-Installation on Debian 10__

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

At this stage docker is just executable with sudo. Let´s enroll our xnat user to the docker group. The group "docker" should already exists, if not create one:

```bash
sudo groupadd docker
sudo usermod -aG docker xnat
```

Set the the right permissions:

```bash
sudo chown root:docker /var/run/docker.sock
sudo chown -R root:docker /var/run/docker
```

Test:

```bash
docker info
```

# __2) Installation of Container PlugIn for XNAT__ 

First download the XNAT-container-PlugIn:
https://wiki.xnat.org/container-service/container-service-122978848.html

Next copy the "jar" file to the XNAT-Home directory and restart tomcat:
```bash
cd ~/Downloads
sudo mv container-service-3.3.0-fat.jar /Data/xnat/home/plugins
sudo systemctl restart tomcat
```
Now we should be able to pull docker images! Navigate on XNAT web-page to: Administer -> Plugin Settings

![image](https://user-images.githubusercontent.com/88328518/226909623-ce55e176-6ddf-481d-8485-3f2ff16fff74.png)



