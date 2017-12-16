# Udacity Project 6 - Configure Linux Server

Configure a baseline instance of a Linux server to host web applications. Includes instructions on installing updates, security measures, and installing web and database servers.

IP address: 50.112.217.129
URL: http://ec2-50-112-217-129.us-west-2.compute.amazonaws.com/
SSH port: 2200

Connect using `ssh grader@50.112.217.129 -i Grader_key -p 2200`

## Connect to server

1. Download the private key "Udacity_key.pem"
2. Place the private key into desired location on your computer ex. directory called ".ssh" using the command in your terminal

`$ mv ~/Downloads/Udacity_key.pem ~/.ssh/`

3. cd into the .ssh directory and change the permission of the private key so only you can view it using the command

`$ chmod 600 Udacity_key.pem` 

4. Connect to the server instance using the following command while inside the .ssh directory. If you are not in the directory the pem file is in, simply replace "Udacity_key.pem" to the include the full path to the location of that file.

`$ ssh -i Udacity_key.pem ubuntu@50.112.217.129`

You may see a message saying "The authenticity of host '50.112.217.129 (50.112.217.129)' can't be established. Are you sure you want to continue connecting (yes/no)?" Type yes and you should be connected as the ubuntu user.

## Update packages
1. Update all currently installed packages using first the command 

`$ sudo apt-get update`

This will make your system aware of all the software available and the latest version numbers. "apt-get" is the main interface for package related functionality. You can run `$ man apt-get` to see all it can do.

2. Now run the command

`$ sudo apt-get upgrade`

This command will do the actual updating of the installed packages.

3. Since we are on the topic of package management, run the command below to install the finger package for user management in the next section.

`$ sudo apt-get install finger` 

## Create new user
1. Run the following command to create a new user named grader

`$ sudo adduser grader`

It will prompt you to create a password and enter some values such as full name, etc.

2. Create a new file in the sudoers directory using

`$ sudo nano /etc/sudoers.d/grader`

3. Add the line "grader ALL=(ALL:ALL) ALL" to the text file to give the user sudo access.

4. You can confirm the user was created using the command 

`$ finger grader`

5. You can switch to the grader user from ubuntu user using the command 

`$ su - grader`

It will prompt you for the password you created when making the grader user.

6. Confirm that grader has sudo permissions by running 

`$ sudo -l`

If you see a message like the one below, it means grader has sudo permissions

'Matching Defaults entries for grader on
    ip-XX-XX-XX-XX.ec2.internal:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User grader may run the following commands on
	ip-XX-XX-XX-XX.ec2.internal:
    (ALL : ALL) ALL'


## Set up SSH keys for grader user
1. Open a new terminal tab or window in which you are NOT connected to the server. Run the following command to generate the key pair for grader.

`$ ssh-keygen`

It will ask you where to save the key. You can save it into "/Users/[user]/.ssh/[keyname]" where [user] is your local user for your computer and [keyname] is whatever you want to name the key being generated. This will create 2 files named: [keyname] and [keyname].pub. I named mine "Grader_key".

2. Now place the public key into the server. Go back to the other terminal tab/window where you are logged in as grader. 

3. Create a directory called ".ssh" using the command 

`$ mkdir ssh`

4. Create a file within that directory called "authorized_keys" using the command 

`$ touch .ssh/authorized_keys`

5. Now copy the contents of the [keyname].pub file from your local terminal window thats NOT connected to the server. You can view the contents by using the following command. Be sure to cd into the local ".ssh" folder that contains the 2 files created from "ssh-keygen".

'cat [keyname].pub'

6. In the terminal window connected to the server as grader, run the command 

`$ nano .ssh/authorized_keys`

This will open up that file to edit. Paste the contents of [keyname].pub from your local terminal window into this file and save. 

7. Set up permissions for the ssh directory and authorized_keys file in your connected grader terminal using the commands 

`chmod 700 .ssh`
`chmod 644 .ssh/authorized_keys`

8. You should now be able to connect to the server instance as grader using the command

`$ ssh grader@50.112.217.129 -i Grader_key` 

Where [Grader_key] should be replaced with whatever you named your key when you ran the "ssh-keygen" command and created a keypair. It will prompt you for the passphrase that you entered when generating the keypair as well.

## Disable ssh login for root user
1. Run the command

`sudo nano /etc/ssh/sshd_config`

Change "PermitRootLogin prohibit-password" to "PermitRootLogin no"

2. Run the following command for it to take effect.

`sudo service ssh restart`

You should now only be able to ssh in using grader `$ ssh grader@50.112.217.129 -i Grader_key`. Note: you will not be able to connect to your instance through the AWS console anymore.

## Change SSH port and configure the Uncomplicated Firewall

1. Run the command 

`sudo nano /etc/ssh/sshd_config`

Change the line that says "Port 22" to read "Port 2200" and save.

2. Restart SSH using the command

`sudo service ssh restart`

3. Configure the firewall in the terminal to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123) using the commands

`sudo ufw default deny incoming` to block everything coming in
`sudo ufw default allow outgoing` to allow everything outgoing
`sudo ufw allow 2200/tcp` to allow ssh
`sudo ufw allow 80/tcp` to allow HTTP
`sudo ufw allow 123/udp`to allow NTP
`sudo ufw deny 22` to deny port 22 since we want connection through 2200
`sudo ufw enable` to enable firewall
`sudo ufw status` to check which ports are open and see if ufw is active

4. Go to https://lightsail.aws.amazon.com/ls/webapp/home/instances and click on your instance. Click the Networking tab and you should see a section called "Firewall". Click "Add another" and create rules:

Custom | TCP | 2200
Custom | UDP | 123

Delete the rule for SSH | TCP | 22 to disable port 22

You should now be able to ssh in through only port 2200 instead of port 22 using `$ ssh grader@50.112.217.129 -i Grader_key -p 2200` 

## Configure the local timezone to UTC
1. Run the command and choose UTC

`sudo dpkg-reconfigure tzdata` 

## Install Apache

1. Run the command 

`sudo apt-get install apache2`

2. Use the command below to start the web server

`sudo service apache2 start`

3. Check that apache is working by visiting 50.112.217.129 in your browser. You should see a page titled "Apache2 Ubuntu Default Page"

## Install mod_wsgi

1. Run the command 

`sudo apt-get install libapache2-mod-wsgi python-dev`

2. Enable mod_wsgi with  the command

`sudo a2enmod wsgi`

## Install PostgrSQL

1. Install PostgreSQL with the command

`sudo apt-get install postgresql postgresql-contrib`

2. Open the config file and check that only connections from the local host are allowed using the command 

`sudo nano /etc/postgresql/9.3/main/pg_hba.conf`

The file should look like this with the comments removed

`local   all             postgres                                peer
local   all             all                                     peer
host    all             all             127.0.0.1/32            md5
host    all             all             ::1/128                 md5`

## Create a DB user

1. Change to the Linux user prostgres created by PostgreSQL during installation using the command 

`sudo su - postgres`

2. Connect to psql using 

`psql`

3. Create a user named "catalog" using the command

`CREATE ROLE catalog WITH LOGIN;`

4. Allow user to create databases with the command

`ALTER ROLE catalog CREATEDB;`

5. Give the catalog user a password using the command

`\password catalog`

6. Check that the user was created using the command `\du`. It should return a table users catalog and postgres

7. Create a database called "catalog" using the command 

`CREATE DATABASE catalog WITH OWNER catalog;`

You can use the command `\c catalog` to connect to the catalog db

8. Check that the database was created using the command `\l`

9. Exit psql using `\q` and switch back to the grader user by running `exit`

## Install Git and clone project

1. Install git using the command 

`sudo apt-get install git`

2. Now change directories to /var/www using the command

`cd /var/www`

3. Create a directory named "catalog" here using the command 

`sudo mkdir catalog`

4. Change the owner of the catalog directory to grader using the command 

`sudo chown -R grader:grader catalog`

5. Now change directories into the directory catalog using

 `cd catalog`

6. Clone your project from github into this directory using

`git clone https://github.com/cecilenguyen/CatalogApp.git`

This will clone your files into a folder named whatever your repo name is. In this case, my files are in a folder named "CatalogApp".

7. Change directories into "CatalogApp" and change the name of the file app.py to __init__.py using the commands

`cd CatalogApp`
`mv app.py __init__.py`

8. Edit the last line of the __init__.py file from `app.run(host='0.0.0.0', port=8000)` to `app.run()` using the command

`sudo nano __init__.py`

9. Edit the database_setup.py file to use PostgreSQL instead of SQLite

`sudo nano database_setup.py`

Change the line that reads `engine = create_engine('sqlite:///catalog.db')` to be `engine = create_engine('postgresql://catalog:PASSWORD_FOR_DATABASE@localhost/catalog')`

where "PASSWORD_FOR_DATABASE" is the password you created for the catalog db and the IP address is changed accordingly.

10. Make the same edit above for the "create_engine" lines in the files __init__.py and database_init.py

## Update Google Authorization

1. Go to the Google developers console and create a new OAuth Client ID

2. Add http://ec2-50-112-217-129.us-west-2.compute.amazonaws.com/ and http://50.112.217.129 as authorized Javascript origins

3. Add http://ec2-50-112-217-129.us-west-2.compute.amazonaws.com and http://ec2-50-112-217-129.us-west-2.compute.amazonaws.com/gconnect
as authorized redirect URIs

4. Download the client_secrets JSON 

5. Replace the contents of the existing client_secrets.json file that is in /var/www/catalog/CatalogApp with the contents of the newly downloaded client_secrets.json file using the following commands

`cd /var/www/catalog/CatalogApp`
`nano client_secrets.json`

6. Update the templates/login.html file in the CatalogApp directory with the new client ID from the Google OAuth you just created on the line that says "data-clientid=XXXXX" using the commands

`cd templates`
`nano login.html`

7. Edit the file path in the __init__.py file to read 'client_secrets.json' to 'var/www/catalog/CatalogApp/client_secrets.json'. Do so with the command

`nano __init__.py`

You may need to change directories `cd ..` to get back to the CatalogApp directory where the __init__.py file is from within the templates directory.

There are 2 instances in the file where you will have to change to the full file path: 

1. Right below the imports and DBSessions you will find the line 

`CLIENT_ID = json.loads(
    open('client_secrets.json', 'r').read())['web']['client_id']`

2. In the gconnect route, you will find the line 

`oauth_flow = flow_from_clientsecrets('client_secrets.json', scope='')`

## Set up virtual environment and dependencies

1. Change directories back outside of /var/www/catalog/CatalogApp using the command `cd ..` until you reach just "$/"

2. Install pip 

`sudo apt-get install python-pip`

3. Install virtualenv 

`sudo apt-get install python-virtualenv`

4. Change directories into /var/www/catalog/CatalogApp and run the command 

`/var/www/catalog/CatalogApp`
`virtualenv catalogEnv`

where "catalogEnv" is whatever you want to name your temporary environment

5. Activate the environment created using the command

`. catalogEnv/bin/activate`

6. Once it's activated, you should see "(catalogEnv)" in front of your logged in linux username

7. Install project dependencies within the virtual environment

`pip install httplib2`

`pip install requests`

`pip install --upgrade oauth2client`

`pip install sqlalchemy`

`pip install flask`

`sudo apt-get install libpq-dev` (Note: this will install to the global evironment)

`pip install psycopg2`

8. Run the "database_init.py" file to populate the database using the command

`python database_init.py`

9. Run the "__init__.py" file to test that all environmental variables are correctly installed using `python __init__.py`. You should get a message like 

`* Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
 * Restarting with stat
 * Debugger is active!
 * Debugger PIN: 298-931-904`

9. Once it is running correctly, CTRL+C to quit and deactivate the virtual environment by running `deactivate`

## Configure Apache to serve the app

1. Create a catalog.wsgi file in /var/www/catalog using the command 

`touch /var/www/catalog/catalog.wsgi`
`nano /var/www/catalog/catalog.wsgi`

2. Add the following to the file 

3. Create a file "catalog.conf" in /etc/apache2/sites-available/ using the commands

`touch /etc/apache2/sites-available/catalog.conf`
`nano /etc/apache2/sites-available/catalog.conf`

5. Paste the following into catalog.conf

`<VirtualHost *:80>
		ServerName 50.112.217.129
		WSGIScriptAlias / /var/www/catalog/catalog.wsgi
		<Directory /var/www/catalog/catalogApp/>
			Order allow,deny
			Allow from all
			Options -Indexes
		</Directory>
		Alias /static /var/www/catalog/CatalogApp/static
		<Directory /var/www/catalog/CatalogApp/static/>
			Order allow,deny
			Allow from all
			Options -Indexes
		</Directory>
		ErrorLog ${APACHE_LOG_DIR}/error.log
		LogLevel warn
		CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>`

6. Disable the default Apache site using

`sudo a2dissite 000-default.conf`

7. Enable the catalog app virtual host using 

`sudo a2ensite catalog.conf`

8. Reload Apache using

`sudo service apache2 reload`

Your application should now be served on http://50.112.217.129/ and http://ec2-50-112-217-129.us-west-2.compute.amazonaws.com/



