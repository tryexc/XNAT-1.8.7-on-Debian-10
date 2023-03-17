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
Java is then located at: 

```bash
/usr/lib/jvm/adoptopenjdk-8-hotspot-amd64
```

  2) Apache Tomcat 8.5 or 9.0 (recommended; 7.0 supported but requires special configuration)

Here I installed __Tomcat 9.__ The Installation and configuration is based on the following website: https://linuxize.com/post/how-to-install-tomcat-9-on-debian-10/
Skip the java part in the description, we already have java-8 installed.

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

Then extract the file and move it to ```/opt/tomcat```:

```bash 
tar -xf apache-tomcat-9.0.73.tar.gz
```
I also renamed the file because i don´t like "*.*" in folder- or filenames:
```basch
sudo mv apache-tomcat-9.0.73 /opt/tomcat/apache-tomcat-9073
```

In the next steps we are going to configure the tomcat service. Therefore we have to define a few folder-paths. In case we are switching the tomcat version later, it is better to wor with a fixed folder name. Therfore we are using a symbolic-link:

```bash
sudo ln -s /opt/tomcat/apache-tomcat-9073 /opt/tomcat/latest
```

In case of updating Tomcat to a newer version we just have to change the link ```/opt/tomcat/latest``` to the new version.

Now we are changing the ownership of ```/opt/tomcat``` and make the ```**/bin/*.sh``` files executable.
Here we are using our xnat user not the tomcat user. Yes we generated this at the beginning, but we will no longer use it.

```bash
sudo chown -R xnat: /opt/tomcat
sudo sh -c 'chmod +x /opt/tomcat/latest/bin/*.sh'
```

OK, let´s configure tomcat! Therefore we have to generate a ```tomcat.service``` file in ```/etc/systemd/system```.
I do ist with nano directly in the terminal:

```bash
sudo nano /etc/systemd/system/tomcat.service
````

Use the following definitions:

```service 
[Unit]
Description=Tomcat 9.0 servlet container
After=network.target

[Service]
Type=forking

User=xnat
Group=xnat

PrivateTmp=yes
AmbientCapabilities=CAP_NET_BIND_SERVICE
NoNewPrivileges=true
CacheDirectory=tomcat9
CacheDirectoryMode=750
ProtectSystem=strict

ReadWritePaths=/opt/tomcat/latest/conf/Catalina/
ReadWritePaths=/opt/tomcat/latest/webapps/
ReadWritePaths=/opt/tomcat/latest/logs/
ReadWritePaths=/opt/tomcat/latest/temp/
ReadWritePaths=/opt/tomcat/latest/work/Catalina/localhost/
ReadWritePaths=/data/xnat/

Environment="JAVA_HOME=/usr/lib/jvm/adoptopenjdk-8-hotspot-amd64"
Environment="JAVA_JRE=/usr/lib/jvm/adoptopenjdk-8-hotspot-amd64/jre"
Environment="JAVA_OPTS=-Djava.security.egd=file:///dev/urandom"
Environment="CATALINA_BASE=/opt/tomcat/latest"
Environment="CATALINA_HOME=/opt/tomcat/latest"
Environment="CATALINA_PID=/opt/tomcat/latest/temp/tomcat.pid"
Environment="CATALINA_OPTS=-Xms512M -Xmx1024M -server -XX:+UseParallelGC -Dxnat.home=/data/xnat/home"

ExecStart=/opt/tomcat/latest/bin/startup.sh
ExecStop=/opt/tomcat/latest/bin/shutdown.sh

[Install]
WantedBy=multi-user.target
```

Note:
Here you see why the use of a symbolic link is a good idea because when you change tomcat. you don´t have to change the service definition. Two more things: You have to change the ```JAVA_HOME and JAVA_JRE``` to the above generated java-8 path. And we defined the xnat home directory in ```CATALINA_OPTS``` we don´t generated the home directories in ```/data/xnat/``` yet. We will do ist later! 

Every time we change the service file we have to reload it:
(Stop tomcat before if it is running: ```sudo systemctl start tomcat``` )

```bash
sudo systemctl daemon-reload
sudo systemctl start tomcat
```

I also defined the tomcat service user as follows (Not sure if it is really necessary):
I added a file ```tomcat9.conf``` in ```/opt/tomcat/latest/conf```.

```bash
sudo nano /opt/tomcat/latest/conf/tomcat9.conf
```

with the following content:

```service
# Run Tomcat as this user ID. Not setting this or leaving it blank will use the
# default of tomcat9.
TOMCAT9_USER=xnat

# Run Tomcat as this group ID. Not setting this or leaving it blank will use
# the default of tomcat9.
TOMCAT9_GROUP=xnat
```
Note: If you updating tomcat you have to do it again for the new tomcat version!

Check:

```bash
sudo systemctl status tomcat
```

If no errors (maybe at this stage there is an error or warning b.c. we don´t generate the xnat home directory!), enable autostart at boot time:

```bash
sudo systemctl enable tomcat
```
Done.... Thats it with tomcat. 

Here again the basic tomcat actions:

```basch
sudo systemctl stop tomcat
sudo systemctl daemon-reload
sudo systemctl start tomcat
sudo systemctl status tomcat

sudo systemctl restart tomcat
```

  3) PostgreSQL 10 or later
  
Now we will focus to Database installation. We ant use the latest version of PostgreSQL.
Here I use the descripton you´ll find here: https://linuxize.com/post/how-to-install-postgresql-on-debian-10/

First update the packages:

```bash
sudo apt update
```

Then install PostgreQSL:

```bash
sudo apt install postgresql postgresql-contrib
```

Last check the version:

```bash
sudo -u postgres psql -c "SELECT version();"
```

Done, quick and easy!


## 2) Setting-up PostgreSQL

Now we are starting to configure our XNAT database. Here I go back to the official XNAT-manual. You will find it here: https://wiki.xnat.org/documentation/getting-started-with-xnat/xnat-installation-guide/configuring-postgresql-for-xnat

The steps are:

1: Create XNAT's database user

2: Configuring database access

3: Configuring TCP/IP Listeners -> I don´t do that!

4: (Optional) Managing database access by user and host  -> I don´t do that!

So focus on step 1 and 2:

```bash
$ sudo su - postgres 
postgres:postgres$ createuser --createdb xnat
postgres:postgres$ createdb --owner=xnat xnat;
postgres:postgres$ psql
psql (12.6)
Type "help" for help.

postgres=# \password xnat
Enter new password:
Enter it again:
postgres=# \q
postgres:~$ logout
$
```

Restart PostgreSQL:

```bash
sudo systemctl restart postgresql
```

Puh, that was also no magic!

## 3) Set Up XNAT_HOME and the File System Structure

Now we are generating all necessary folder:

a) XNAT_HOME:

```bash
sudo mkdir -p /data/xnat/home/config
sudo mkdir /data/xnat/home/logs
sudo mkdir /data/xnat/home/plugins
sudo mkdir /data/xnat/home/work
````

b) XNAT data:

```bash
sudo mkdir /data/xnat/archive
sudo mkdir /data/xnat/build
sudo mkdir /data/xnat/cache
sudo mkdir /data/xnat/fileStore
sudo mkdir /data/xnat/ftp
sudo mkdir /data/xnat/inbox
sudo mkdir /data/xnat/prearchive
```

c) Grant ownership to the Tomcat service user:

```bash
sudo chown -R xnat:xnat /data
```

## 4) Configure XNAT for Initial Startup and install Web App

a) Starting with creating a XNAT initial-file in ```/data/xnat/home/config```















