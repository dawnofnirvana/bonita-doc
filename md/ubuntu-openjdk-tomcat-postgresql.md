# Custom Bonita Platform Setup

How to set up a Bonita Platform from scratch in a specific environment (Ubuntu, Openjdk, Tomcat, PostgreSQL).

## Operating System

Install a Ubuntu Linux distribution.
Simply follow a standard Ubuntu installation procedure. You won't need any specific settings here.

Note: make sure you got a text editor installed such as `nano`. In order to install it: `sudo apt-get install nano`

Note: we use `aptitude` command line tool to install packages such as OpenJDK or PostgreSQL. You can also use different tool such
as `apt-get` or a graphical package manager such as `synaptic`.

## Java Virtual Machine

To install the OpenJDK JVM you need to install the `openjdk-8-jre-headless` package:

* Run the following command line: `sudo aptitude install openjdk-8-jre-headless`
* If needed, type your Ubuntu user password
* Type _**Enter**_ to confirm that you want to continue installation

To check that Java is correctly setup in a console/terminal, type: `java -version`.
You should get a similar message to the one below:  
```
openjdk version "1.8.0_111"
OpenJDK Runtime Environment (build 1.8.0_111-8u111-b14-2ubuntu0.16.04.2-b14)
OpenJDK 64-Bit Server VM (build 25.111-b14, mixed mode)
```

## Database

### Install

To install the PostgreSQL 9.3 you need to install the `postgres-9.3` package:

* Run the following command line: `sudo aptitude install postgresql-9.3`
* If needed, type your Ubuntu user password
* Type _**Enter**_ to confirm that you want to continue installation

### Create databases and user

You now need to create: a new PostgreSQL user and two new databases for Bonita, and grant the user the appropriate privileges to access
the databases. In a terminal or console, type following commands:

* Switch to a shell for `postgres` user account: `sudo -u postgres -i`
* To create a new database user named `bonita_db_user` type: `createuser -P bonita_db_user`
* Enter (twice) the password for the newly created database user. E.g.: `bonita_db_password`
* Create a new database (for Bonita Engine) named `bonita_db` and give ownership to `bonita_db_user`: `createdb -O bonita_db_user bonita_db`
* Create another database (for BDM) named `bonita_bdm` and give ownership to `bonita_db_user`: `createdb -O bonita_db_user bonita_bdm`
* Exit `postgres` session, type command: `exit`

You can now verify that user and database are correctly created. In a terminal or console type following commands:

* Start PostgreSQL client program by typing: `psql -d bonita_db -h 127.0.0.1 -U bonita_db_user`
* Type the PostgreSQL user password. E.g.: bonita\_db\_password
* You can run test query that should return `1`: `SELECT 1;`
* Now you can quit PostgreSQL client, type: `\q`

### Enable prepared transactions

Prepared transactions is disabled by default in PostgreSQL. You need to enable it:

* Edit the `postgresql.conf` configuration file: `sudo nano /etc/postgresql/9.3/main/postgresql.conf`
* Look for the line with `max_prepared_transactions` (line 117)
* Uncomment the line: remove `#` character
* Change the value from `zero` to `100`
* Save modification and quit text editor
* Restart PostgreSQL server: `sudo service postgresql restart`

## Application Server

### Install Tomcat 8.5

Run the following command lines:

* `sudo aptitude install curl`
* `cd /tmp/`
* `wget http://apache.mirrors.ionfish.org/tomcat/tomcat-8/v8.5.31/bin/apache-tomcat-8.5.31.tar.gz`
* `sudo tar xzvf apache-tomcat-8.5.31.tar.gz -C /usr/share/tomcat8 --strip-components=1`

Tomcat 8.5 should now be installed on your computer. Now you should enable tomcat to be run as a service

* `sudo nano /etc/systemd/system/tomcat.service`
* Copy the following code into the file
```
[Unit]
Description=Apache Tomcat Web Application Container
After=network.target

[Service]
Type=forking

Environment=JAVA_HOME=Your Java Home
Environment=CATALINA_PID=/usr/share/tomcat/temp/tomcat.pid
Environment=CATALINA_HOME=/usr/share/tomcat
Environment=CATALINA_BASE=/usr/share/tomcat
Environment='CATALINA_OPTS=-Xms512M -Xmx1024M -server -XX:+UseParallelGC'
Environment='JAVA_OPTS=-Djava.awt.headless=true -Djava.security.egd=file:/dev/./urandom'

ExecStart=/usr/share/tomcat/bin/startup.sh
ExecStop=/usr/share/tomcat/bin/shutdown.sh

User=yourUser
Group=yourGroup
UMask=0007
RestartSec=10
Restart=always

[Install]
WantedBy=multi-user.target
```

* Save and close the file
* Reload the systemd daemon: `sudo systemctl daemon-reload`

In case of problems with this part refer to the documentations pages [How to Install and Configure Apache Tomcat 8.5 on Ubuntu 16.04](https://www.howtoforge.com/tutorial/how-to-install-apache-tomcat-8-5-on-ubuntu-16-04/)

### Add JDBC driver

You need to include JDBC driver in Tomcat classpath:

* Change to Tomcat libraries directory: `cd /usr/share/tomcat8/lib`
* Install `wget` tool in order to be able to download JDBC driver: `sudo aptitude install wget`
* Download the JDBC driver: `sudo wget http://jdbc.postgresql.org/download/postgresql-9.3-1102.jdbc4.jar`

## Bonita Platform

### Download and unzip the Bonita deploy bundle

Download the Bonita deploy bundle from the [Customer Portal](https://customer.bonitasoft.com/)
(Subscription editions) or get the [Community edtion](http://www.bonitasoft.com/downloads-v2). Instructions
below will be given for Bonita Subscription Pack. You can easily adapt files and folder names to your edition.

* Go to Bonitasoft [Customer Portal](https://customer.bonitasoft.com/)
* In **Download** menu, click on _**Request a download**_
* Select your version and click on _**Access download page**_ button
* On the download page, go to the **Deploying Server Components** section
* Click on _**Download BonitaSubscription-x.y.z-deploy.zip**_ link. If your server only has a terminal available
you can copy the link and use `wget` to download the file or use SSH with `scp` command to copy
the file from another computer.
* Make sure that the `BonitaSubscription-x.y.z-deploy.zip` is located in your home folder (e.g. `/home/osuser`).
If you type `cd ~ && ls` you should see the file listed.
* Make sure the `unzip` command is installed: `sudo aptitude install unzip`
* Unzip the deploy bundle: `unzip -q BonitaSubscription-x.y.z-deploy.zip`

Finally, make sure that the user that runs the Tomcat server, is the owner of all Bonita "home" files:

* Change folders and files ownership: `sudo chown -R tomcat8:tomcat8 /opt/bonita`

### JVM system variables

To define JVM system properties, you need to use a new file named `setenv.sh`:

* Copy the file from deploy bundle to Tomcat `bin` folder: `sudo cp ~/BonitaSubscription-x.y.z-deploy/Tomcat-8.5.z/bin/setenv.sh /usr/share/tomcat8/bin/`, where "x.y.z" stands for your current product version.
* Make the file executable: `sudo chmod +x /usr/share/tomcat8/bin/setenv.sh`
* Edit `setenv.sh` file: `sudo nano /usr/share/tomcat8/bin/setenv.sh`
* Change `sysprop.bonita.db.vendor` from `h2` to `postgres`
* Change `btm.root` from `${CATALINA_HOME}` to `/opt/bonita/btm`
* Change `bitronix.tm.configuration` from `${CATALINA_HOME}/conf/bitronix-config.properties` to `/opt/bonita/btm/conf/bitronix-config.properties`

### Add extra libraries to Tomcat

Bonita needs extra libraries such as Bitronix, in order to run on Tomcat:

* Change to the Deploy bundle Tomcat lib folder: `cd ~/BonitaSubscription-x.y.z-deploy/Tomcat-8.5.z/lib`, where "y.z" stands for the last digits of the product version
* Copy the libraries (.jar files) from the Deploy bundle to your Tomcat folder: `sudo cp btm-2.1.3.jar btm-tomcat55-lifecycle-2.1.3.jar jta-1.1.jar slf4j-api-1.6.1.jar slf4j-jdk14-1.6.1.jar /usr/share/tomcat8/lib/` (carefully check and replace with the product versions you currently have)

### Configure Bonita to use PostgreSQL

You need to configure the data source for Bonita Engine.

Warning: make sure you stop Tomcat before performing following operations: `sudo service tomcat8 stop`

* Create new folders in order to store Bitronix files: `sudo mkdir -p /opt/bonita/btm/conf && sudo mkdir /opt/bonita/btm/work`
* Set the ownership of the Bitronix folder: `sudo chown -R tomcat8:tomcat8 /opt/bonita/btm`
* Copy the Bitronix configuration files to `/opt/bonita/btm/conf` folder: `sudo cp ~/BonitaSubscription-x.y.z-deploy/Tomcat-8.5.z/conf/bitronix-* /opt/bonita/btm/conf/`
* Edit `bitronix-resources.properties` file, comment (using `#`) h2 section, uncomment PostgreSQL example
and update the username, password and DB name ("bonita" in the URL property) to match your configuration (e.g.
`bonita_db_user`, `bonita_db_password` and `bonita_db`): `sudo nano /opt/bonita/btm/conf/bitronix-resources.properties`
* Also in `bitronix-resources.properties` update the section for `resource.ds2` (BDM data source) and set the value for the BDM data base (e.g. `bonita_db_user`, `bonita_db_password` and `bonita_bdm`)
* Save and quit: `CTRL+X, Y, ENTER`
* Copy the `bonita.xml` file (Bonita web app context configuration): `sudo cp ~/BonitaSubscription-x.y.z-deploy/Tomcat-8.5.z/conf/Catalina/localhost/bonita.xml /etc/tomcat8/Catalina/localhost/`
* Edit the `bonita.xml` file by commenting the h2 datasource configuration (using ),
uncomment PostgreSQL example and update username, password and DB name (bonita in the URL property) to match your
configuration (e.g. `bonita_db_user`, `bonita_db_password` and `bonita_db`): `sudo nano /etc/tomcat8/Catalina/localhost/bonita.xml`
* Also in `bonita.xml` file update data base configuration for BDM to match your configuration (e.g. `bonita_db_user`, `bonita_db_password` and `bonita_bdm`)
* Save and quit: `CTRL+X, Y, ENTER`
* Copy and overwrite `logging.properties` file: `sudo cp ~/BonitaSubscription-x.y.z-deploy/Tomcat-8.5.z/conf/logging.properties /etc/tomcat8/logging.properties`
* Copy and overwrite `context.xml` file: `sudo cp ~/BonitaSubscription-x.y.z-deploy/Tomcat-8.5.z/conf/context.xml /etc/tomcat8/context.xml`
* Copy and overwrite `server.xml` file: `sudo cp ~/BonitaSubscription-x.y.z-deploy/Tomcat-8.5.z/conf/server.xml /etc/tomcat8/server.xml`
* Edit `server.xml` (`sudo nano /etc/tomcat8/server.xml`) and comment out h2 listener line
* Fix ownership on the copied files: `sudo chown -R root:tomcat8 /etc/tomcat8`

### License

If you run the Subscription Pack version, you will need a license:

* Generate the key in order to get a license:
  * Change the current directory to license generation scripts folder: `cd ~/BonitaSubscription-x.y.z-deploy/request_key_utils-x.y-z`
  * Make the license generation script executable: `chmod u+x generateRequestKey.sh`
  * Run the script: `./generateRequestKey.sh`
  * For `License type:` enter `1` to select `1 - Case counter license.` (*Note*: If your subscription type is cpu based, please refer to the [knowledge-base](https://customer.bonitasoft.com/knowledgebase) in the customer portal)
  * You will get a license key number that you can copy. Make sure that you keep the brackets. If the key is separated by a linebreak, remove it and put the key on a single line.
* Connect to Bonitasoft [Customer Portal](https://customer.bonitasoft.com/)
* Go to Licenses \> **Request a license**
* Fill in the license request forms
* You should receive the license file by email
* Copy the license file to the Bonita "home" folder: `sudo cp BonitaSubscription-x.y-Your_Name-ServerName-YYYYMMDD-YYYYMMDD.lic /opt/bonita/bonita_home-x.y.z/server/licenses/`
* Change folders and files ownership: `sudo chown -R tomcat8:tomcat8 /opt/bonita`

### Deployment

Deploy the Bonita web application:

Copy `bonita.war` to Tomcat `webapps` folder: `sudo cp ~/BonitaSubscription-x.y.z-deploy/Tomcat-8.5.z/webapps/bonita.war /var/lib/tomcat8/webapps/`

Take care to set the proper owner: `sudo chown tomcat8:tomcat8 /var/lib/tomcat8/webapps/bonita.war`

Start Tomcat: `sudo service tomcat8 start`

### First connection

You can access the Bonita Portal using your web browser, just type the following URL `http://<your_server_hostname>:8080/bonita` (your\_server\_hostname can be either an IP address or a name).  
You can log in using the tenant administrator login: `install` and password: `install`.  
The first step is to create at least one user and add it to "administrator" and "user" profiles.
