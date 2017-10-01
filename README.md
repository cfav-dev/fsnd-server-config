# Linux Server Configuration
As part of the Udacity Full Stack Nanodegree, this *README* describes steps taken to configure a linux server instance running on a virtual machine. A Digital Ocean Droplet running *Ubuntu 16.04.3 x64* was created for this project. 


## Access Details
* IP Address: 128.199.229.152
* SSH port: 2200


## Initial Server Setup
*most of these steps are taken [here](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-16-04)*
1. We first login remotely by typing `ssh root@128.199.229.152`.
2. System software is updated: `apt-get update` then `apt-get upgrade` 
3. We create a new user: `adduser grader`. Enter new UNIX password.
4. New user is given root priveleges: `usermod -aG sudo grader`
5. To add public key authentication, we first run `ssh-keygen` in our **local computer** to generate a key pair. Default file paths are accepted. We enter a passphrase if we want.
6. Run `cat ~/.ssh/id_rsa.pub` and copy contents to clipboard.
7. Back in our **server**, we switch to our newly created user: `su - grader`
8. Create .ssh folder and restrict permissions: `mkdir .ssh` then `chmod 700 ~/.ssh`
9. Create *authorized_keys*: `nano ~/.ssh/authorized_keys`
10. Paste public key then save.
11. Disable password authentication if needed. I chose SSH keys during Droplet creation so sshd_config is already setup for this.
12. Login from local computer: `ssh grader@128.199.229.152`
13. Configure SSH: `sudo nano /etc/ssh/sshd_config`
14. Exit and login again remotely: `ssh grader@128.199.229.152 -p 2200`
15. *ufw* is configured to deny incoming and allow outgoing by default: `sudo ufw default deny incoming` and `sudo ufw default allow outgoing`
16. *ufw* allow ports 2200, 80 (http), and 123 (ntp): `sudo ufw allow *port number*`
17. enable firewall: `sudo ufw enable`
18. Try logging in again through SSH

## Web Application Setup
*reference [here](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)*
1. Install Apache: `sudo apt-get install apache2`
2. Install mod-wsgi: `sudo apt-get install libapache2-mod-wsgi python-dev`
3. Enable it: `sudo a2enmod wsgi`
4. In **local computer**, navigate to *vagrant* directory then copy *Item Catalog Project* files to server: `scp -P 2200 -r catalog/ grader@128.199.229.152:/home/grader/`. I chose this option because I did not setup a GitHub repo for the previous project.
5. Make folder for Flask apps: `sudo mkdir /var/www/html/FlaskApps`
6. Copy app to created folder: `sudo cp -r ~/catalog /var/www/html/FlaskApps` and rename main program code to *\_\_init.py\_\_*
7. Install pip: `sudo apt-get install python-pip`
8. Install python packages in a virtual environment:
  * `sudo pip install virtualenv`
  * `sudo virtualenv venv`
  * `source venv/bin/activate`
  * `sudo pip install Flask`
  * `sudo pip install SQLAlchemy`
  * `sudo pip install --upgrade google-api-python-client`
  * `sudo pip install requests`
  * `sudo python __init__.py` to test
  * `deactivate`
9. Configure and Enable a New Virtual Host
  * `sudo nano /etc/apache2/sites-available/Catalog.conf`
```
  <VirtualHost *:80>
        ServerName 128.199.229.152
        ServerAdmin admin@mywebsite.com
        WSGIScriptAlias / /var/www/html/FlaskApps/flaskapp.wsgi
        <Directory /var/www/html/FlaskApp/catalog/>
                Order allow,deny
                Allow from all
        </Directory>
        Alias /static /var/www/html/FlaskApps/catalog/static
        <Directory /var/www/html/FlaskApps/catalog/static/>
                Order allow,deny
                Allow from all
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/access.log combined
	</VirtualHost>
```
  * `sudo a2ensite Catalog`
10. Create .wsgi file: `sudo nano /var/www/html/FlaskApps/flaskapp.wsgi`
11. Restart database: `sudo service apache2 restart`
12. Type `128.199.229.152` in browser to access web app
13. Errors are diagnosed with the command: `sudo tail /var/log/apache2/error.log`
14. Change relative paths to absolute paths: database and client_secrets.json
15. Add public IP address to OAuth 2.0 Client Javascript Origins in https://console.developers.google.com/apis
16. Add this to db to prevent threading error: `/path/to/db?check_same_thread=False`
17. Change ownership of db file and its directory to prevent write errors. Answer found [here](https://serverfault.com/questions/57596/why-do-i-get-sqlite-error-unable-to-open-database-file)
  * `chown www-data *db_file*`
  * `chown www-data /var/www/html/FlaskApps/catalog`
18. Restart apache and retry.