`Server Address`
IP address : http://18.218.123.239/
app url : http://18.218.123.239/
ssh port : `2200`


## Softwares installed
1) git
2) libapache2-mod-wsgi
3) postgresql
4) apache2
5) flask
6) other dependencies of flask project


### Configuration steps
### Create an instance with Amazon Lightsail
1. Sign in to [Amazon Lightsail](https://amazonlightsail.com) using an Amazon Web Services account

2. Follow the 'Create an instance' link

3. Choose the 'OS Only' and 'Ubuntu 16.04 LTS' options

4. Choose a payment plan

5. Give the instance a unique name and click 'Create'

6. Wait for the instance to start up


### Connect to the instance on a local machine
Note: While Amazon Lightsail provides a broswer-based connection method, this will no longer work once the SSH port is changed (see below). The following steps outline how to connect to the instance via the Terminal program on Mac OS machines (this can also be done on a Windows machine with a program such as [PuTTY](http://www.putty.org)).

1. Download the instance's private key by navigating to the Amazon Lightsail 'Account page'

2. Click on 'Download default key'

3. A file called LightsailDefaultPrivateKey.pem or LightsailDefaultPrivateKey-YOUR-AWS-REGION.pem will be downloaded; open this in a text editor

4. Copy the text and put it in a file called lightrail_key.rsa in the local ~/.ssh/ directory

5. Run `chmod 600 ~/.ssh/lightrail_key.rsa`

6. Log in with the following command: `ssh -i ~/.ssh/lightrail_key.rsa ubuntu@18.218.123.239`, where XX.XX.XX.XX is the public IP address of the instance (note that Lightsail will not allow someone to log in as `root`; `ubuntu` is the default user for Lightsail instances)


### Upgrade currently installed packages
1. Notify the system of what package updates are available by running `sudo apt-get update`

2. Download available package updates by running `sudo apt-get upgrade`

## firewall configuration (UFW)
1) default ssh port changed to 2200
2) ports allowed are 123(ntp), 80(http), 2200(ssh)
3) default outgoing allowed & incoming dennied.
4) Run `sudo ufw status` to check which ports are open and to see if the ufw is active; if done correctly, it should look like this:

	```
	To                         Action      From
	--                         ------      ----
	22                         DENY        Anywhere
	2200/tcp                   ALLOW       Anywhere
	80/tcp                     ALLOW       Anywhere
	123/udp                    ALLOW       Anywhere
	22 (v6)                    DENY        Anywhere (v6)
	2200/tcp (v6)              ALLOW       Anywhere (v6)
	80/tcp (v6)                ALLOW       Anywhere (v6)
	123/udp (v6)               ALLOW       Anywhere (v6)
	```

4. Update the external (Amazon Lightsail) firewall on the browser by clicking on the 'Manage' option, then the 'Networking' tab, and then changing the firewall configuration to match the internal firewall settings above (only ports `80`(TCP), `123`(UDP), and `2200`(TCP) should be allowed; make sure to deny the default port `22`)

5. Now, to login, open up the Terminal and run:

	`ssh -i ~/.ssh/lightrail_key.rsa -p 2200 ubuntu@XX.XX.XX.XX`, where XX.XX.XX.XX is the public IP address of the instance i.e 18.218.123.239 here.

Note: As mentioned above, connecting to the instance through a browser now no longer works; this is because Lightsail's browser-based SSH access only works through port `22`, which is now denied.


### Create a new user named `grader`
1. Run `sudo adduser grader`

2. Enter in a new UNIX password (twice) when prompted

3. Fill out information for the new `grader` user

4. To switch to the `grader` user, run `su - grader`, and enter the password


### Give `grader` user sudo permissions
1. Run `sudo visudo`

2. Search for a line that looks like this:

	`root    ALL=(ALL:ALL) ALL`

3. Add the following line below this one:

	`grader	   ALL=(ALL:ALL) ALL`

4. Save and close the visudo file

### Allow `grader` to log in to the virtual machine
1. Run `ssh-keygen` on the local machine

2. Choose a file name for the key pair (such as grader_key)

3. Enter in a passphrase twice (two files will be generated; the second one will end in .pub)

4. Log in to the virtual machine

5. Switch to `grader`'s home directory, and create a new directory called `.ssh` (run `mkdir .ssh`)

6. Run `touch .ssh/authorized_keys`

7. On the local machine, run `cat ~/.ssh/insert-name-of-file.pub`

8. Copy the contents of the file, and paste them in the .ssh/authorized_keys file on the virtual machine

9. Run `chmod 700 .ssh` on the virtual machine

10. Run `chmod 644 .ssh/authorized_keys` on the virtual machine

11. Make sure key-based authentication is forced (log in as `grader`, open the `/etc/ssh/sshd_config` file, and find the line that says, '# Change to no to disable tunnelled clear text passwords'; if the next line says, `PasswordAuthentication yes`, change the 'yes' to 'no'; save and exit the file; run `sudo service ssh restart`)

12. Log in as the grader using the following command:

	`ssh -i ~/.ssh/grader_key -p 2200 grader@18.218.123.239`

Note that a pop-up window will ask for `grader`'s password.
Enter the passphrase that you entered for grader user.


### Configure the local timezone to UTC
1. Run `sudo dpkg-reconfigure tzdata`, and follow the instructions (UTC is under the 'None of the above' category)

2. Test to make sure the timezone is configured correctly by running`date`


### Install and configure Apache
1. Run `sudo apt-get install apache2` to install Apache

2. Check to make sure it worked by using the public IP of the Amazon Lightsail instance as as a URL in a browser; if Apache is working correctly, a page with the title 'Apache2 Ubuntu Default Page' should load


### Install mod_wsgi
1. Install the mod_wsgi package (which is a tool that allows Apache to serve Flask applications) along with python-dev (a package with header files required when building Python extensions); use the following command:

	`sudo apt-get install libapache2-mod-wsgi python-dev`

1. Make sure mod_wsgi is enabled by running `sudo a2enmod wsgi`


### Install PostgreSQL and make sure PostgreSQL is not allowing remote connections
1. Install PostgreSQL by running `sudo apt-get install postgresql`

### Make sure Python is installed
Python should already be installed on a machine running Ubuntu 16.04. To verify, simply run `python`.

### Create a new PostgreSQL user named `catalog` with limited permissions
1. PostgreSQL creates a Linux user with the name `postgres` during installation; switch to this user by running `sudo su - postgres` (for security reasons, it is important to only use the `postgres` user for accessing the PostgreSQL software)

2. Connect to psql (the terminal for interacting with PostgreSQL) by running `psql`

3. Create the `catalog` user by running `CREATE ROLE catalog WITH LOGIN;`

4. Next, give the `catalog` user the ability to create databases: `ALTER ROLE catalog CREATEDB;`

5. Finally, give the `catalog` user a password by running `\password catalog`

6. Check to make sure the `catalog` user was created by running `\du`; a table of sorts will be returned, and it should look like this:

	```
					   List of roles
	 Role name |                         Attributes                         | Member of
	-----------+------------------------------------------------------------+-----------
	 catalog   | Create DB                                                  | {}
	 postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
	```

7. Exit psql by running `\q`

8. Switch back to the `ubuntu` user by running `exit`


### Create a Linux user called `catalog` and a new PostgreSQL database
1. Create a new Linux user called `catalog`:

	- run `sudo adduser catalog`
	- enter in a new UNIX password (twice) when prompted
	- fill out information for `catalog`

1. Give the `catalog` user sudo permissions:

	- run `sudo visudo`
	- search for a line that looks like this: `root    ALL=(ALL:ALL) ALL`
	- add the following line below this one: `catalog    ALL=(ALL:ALL) ALL`
	- save and close the visudo file

2. While logged in as `catalog`, create a database called catalog by running `createdb catalog`

3. Run `psql` and then run `\l` to see that the new database has been created

4. Switch back to the `ubuntu` user by running `exit`


### Install git and clone the catalog project
1. Run `sudo apt-get install git`

1. Create a directory called 'catalog' in the /var/www/ directory

1. Change to the 'catalog' directory, and clone the catalog project:

	`sudo git clone https://github.com/AbhishekSuri/catalog.git

### Write a .wsgi file
1. Apache serves Flask applications by using a .wsgi file; create a file called project_server.wsgi in /var/www/catalog/

2. Add the following to the file:

   import sys
   sys.path.insert(0, '/var/www/catalog/catalog')

   from dbp import app as application
   application.secret_key = 'My_secret_key'
	```
### Now, this wsgi file needs to be enabled.
1. Add the following lines to the /etc/apache2/sites-enabled/000-default.conf file.

WSGIDaemonProcess dbp
WSGIScriptAlias / /var/www/catalog/project_server.wsgi

<Directory catalog>
    WSGIProcessGroup project_server
    WSGIApplicationGroup %{GLOBAL}
    Order deny,allow
    Allow from all
</Directory>

2. Resart Apache: `sudo service apache2 restart`


### Switch the database in the application from SQLite to PostgreSQL
1. Replace line 20 in `dbp.py`, line 7 in `db_populate.py`, and line 7 in `dbp.py` with the following:

	engine = create_engine('postgresql://catalog:password@localhost/catalog')

2. Run `sudo service apache2 reload`  

### Set up the database schema and populate the database

1. Then run `python db_populate.py`

2. Resart Apache again: `sudo service apache2 restart`

3. Now open up a browser and check to make sure the app is working by going to http://18.218.123.239/ or http://ec2-18-218-123-239.us-east-2.compute.amazonaws.com .
