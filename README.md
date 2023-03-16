# XNAT-1.8.7-on-Debian-10
This is just an how-to installation guide for XNAT-1.8.7 on Debian 10

The following "setup-guide" is a list of steps i had to do to get XNAT running on my machine. Its a combination of the official XNAT install-documentation and a lot of trial-and-error which was accompanied by intensive web research!

The fastest way to get a working XNAT server is to follow the described steps - of the official documentation - for using it in a virtual machine together with __Vagrant.__ This is really straight forward! All necessary steps are explaint here: https://wiki.xnat.org/documentation/getting-started-with-xnat/running-xnat-in-a-vagrant-virtual-machine

But in my case of using XNAT in a reasearch project, with the task of administering the underlying server and the XNAT-Web application over a long period of time, i decided to setup XNAT from scratch.

Of course I tried to follow the official XNAT documentation for this, but honestly, who wants to read all that!? Sorry XNAT-guys but your documentation didn't work for me. I never heared about __tomcat__ before i startet with XNAT. OK, maybe the installation instructions are not written for someone like me, but for someone who knows! Never mind let's go:

The installation steps are here: https://wiki.xnat.org/documentation/getting-started-with-xnat/xnat-installation-guide

## 1) Prerequisites
+ __Choosing the right OS__

I thought an up-to-date Debian would be a good idea for sure. No it was not! After much back and forth and bugs here and there and Java-8 and Debian-11 along with Tomcat-9. I can no longer say what errors I had! But somehow "no one wanted to talk to the other". I read some forums and one of them said XNAT and Debian 11 doesn't work that way! I don't know if that's actually true, but it gave me hope that it wasn't just me!

So I decided to switch to __Debian 10.__ I also created a (sudo)user named __xnat.__

+ __Core XNAT dependencies__

Ok what else is needed?

  1) __Java 8__ JRE or JDK (XNAT will not run with later versions of Java)

 Here I followed the steps of the inst.-guide:

```bash
wget -qO - https://adoptopenjdk.jfrog.io/adoptopenjdk/api/gpg/key/public | sudo apt-key add -
sudo add-apt-repository --yes https://adoptopenjdk.jfrog.io/adoptopenjdk/deb/
sudo apt update
sudo apt --yes install adoptopenjdk-8-hotspot
```
Java is then located in: 

```bash
/usr/lib/jvm/adoptopenjdk-8-hotspot-amd64
```

  2) Apache Tomcat 8.5 or 9.0 (recommended; 7.0 supported but requires special configuration)

Here I installed __Tomcat 9.__ The Installation and configuration is based on the following website: https://linuxize.com/post/how-to-install-tomcat-9-on-debian-10/
Skip the java part in the description of the link, we already have java-8 installed.

Create a Tomcat user:
```basch
sudo useradd -m -U -d /opt/tomcat -s /bin/false tomcat
```
Download Tomcat:
```bash
cd /tmp
wget https://www-eu.apache.org/dist/tomcat/tomcat-9/v9.0.73/bin/apache-tomcat-9.0.73.tar.gz
```
(ver. 9.0.27 as describebed in the link above isn´t available, so use Ver. 9.0.73 instead)

  3) PostgreSQL 10 or later

## 2) Setting-up PostgreSQL
      
