# Catalog_Server
Udacity project 5 (linux server)

## The project
Build an ubuntu linux server and run the catalog application from project 3 on it.

## The process
1.	Launch your Virtual Machine with your Udacity account. Please note that upon graduation from the program your free Amazon AWS instance will no longer be available.
    1.	Download public key
    2.	Ssh into server
2.	Create a new user named grader
    1.	sudo adduser grader
3.	Give the grader the permission to sudo
    1.	Visudo edit sudoers section to include: grader (ALL)=(ALL…
4.	Update all currently installed packages
    1.	sudo apt-get update / upgrade
5.	Change the SSH port from 22 to 2200
    1.	Edit the file located /etc/ssh/sshd_config
    2.	Find where the port 22 is and change to 2200
6.	Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
    1.	Sudo ufw default deny incoming
    2.	Sudo ufw default allow outgoing
    3.	Sudo ufw allow ssh
    4.	Sudo ufw allow 2200/tcp
    5.	Sudo ufw allow http / https / ntp
7.	Configure the local timezone to UTC
    1.	Sudo dpkg-reconfigure tzdata
8.	Install and configure Apache to serve a Python mod wsgi application
    1.	Sudo apt-get install apache2
    2.	Sudo apt-get install libapache2-mod-wsgi
    3.	At this point I ran into an error: Could not reliably determine the server’s fully qualified domain name….
    4.	echo "ServerName localhost" | sudo tee /etc/apache2/conf-available/fqdn.conf  sudo a2enconf fqdn
    5.	credit: http://askubuntu.com/questions/256013/could-not-reliably-determine-the-servers-fully-qualified-domain-name
    6.	Next build a wsgi app in the /var/www/html/myapp.wsgi file
9.	Install and configure PostgreSQL:
    1.	Sudo apt-get install postgresql
At this point I decided to take a break and check on my website. I visited:
http://ec2-MY-PUBLIC-IP-ADDRESS.us-west-2.compute.amazonaws.com/
10.	Do not allow remote connections
    1.	PSQL remote access is denied by default, but can be turned on
11.	Create a new user named catalog that has limited permissions to your catalog application database
    1.	To create a new user switch users to postgres (su – postgre)
    2.	 createuser -P -d -l catalog;
        1.	–P forces password
        2.	–d gives database creation rights
        3.	–l allows login
        4.	catalog is the user name
    3.	Once created, I will open psql by entering ‘psql’ and create a database that is owned by the catalog user (create database mycat owner catalog) (\l will list databases) (\du will list users and attributes) (\q will quit psql)
12.	Install git, clone and setup your Catalog App project (from your GitHub repository from earlier in the Nanodegree program) so that it functions correctly when visiting your server’s IP address in a browser. Remember to set this up appropriately so that your .git directory is not publicly accessible via a browser!
    1.	sudo apt-get install git
    2.	git clone https://github.com/git/git
    3.	git config --global user.name "Nicolas H"
    4.	git clone https://github.com/Afkupuz/catalog
  
In order to get the app running on the server I needed to make some adjustments. Firstly, the app was originally using MySQL via sqlalchemy. I needed to have a user configured in psql that has permissions to access the database and edit it.
I followed https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps for extra help
Instead of __init__.py I changed my project.py to init.py
Then I had to change the database to use psql and provide a user name, password and database to use:
	engine = create_engine('postgresql+psycopg2://USERNAME:PASSWORD@localhost/DATABASENAME)
This has to be changed in both my database_setup and my project.py files.
Once done, I was able to run ‘python project.py’ and I was given a series of errors that dealt with missing modules that I installed using pip
Now I have the project running:
 * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
 * Restarting with stat
 * Debugger is active!
 * Debugger pin code: 261-080-655 
And I can’t access localhost port 5000 on the server, SO
I checked sudo a2ensite catalog, this returned that catalog was already running. This means that I correctly inserted my catalog.conf file into the sites-available folder in apache2 directory
I checked sudo a2enconf catalog, this returned that no catalog.conf was there. So I copied the file catalog.conf from the sites-available to the conf-available and ran it again. (note, as soon as a2enconf catalog was working, my main site went down, to bring it back I put sudo a2disconf catalog and restarted)
It still wasn’t working so I checked /var/log/apache2/error.log for what was going on. It showed that there was an issue with python.eggs. To fix this I created a directory mkdir /var/www/.python-eggs and make it accessible to all with chmod 777 /var/www/.python-eggs.

Now that everything is arranged, I visited the website and IT WORKS!!! BUT the CSS returns a 404 and the Oauth2 doesn’t work.

To fix the OAuth2 I had to open the options in my original google app engine project and add the new website as an authorized user. But the website is not finding the secrets folder so I needed to add the full path to the file in the code.

Facebook login doesn’t work either… I had to log in to facebook developer and change my apps URL to the new one.

CSS wasn’t working because in the .conf file I had the path …/catalog/catalog/static but I moved things so it needed to be …/catalog/static





You may also find this Getting Started Guide helpful when working on Project 5.


ssh -i ~/.ssh/udacity_key.rsa root@52.36.143.65 -p 2200


