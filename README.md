# ud-linux-server
* Server-Address/Webhost: http://35.157.68.204/
* SSH-Port: 2200
* Graders-PW: udacity
## Configuration Steps
### Update Server Software
* `sudo apt-get update`
* `sudo apt-get full-upgrade`
* `sudo apt-get autoremove`
* `sudo reboot`
### Change SSH Port
* Configure Lightsail Firewall to enable traffic on port 2200
* Update the port in `/etc/ssh/sshd_config`
* Restart the SSH-Server with `sudo service ssh restart`
### UFW Config
* Default deny any incoming traffic `sudo ufw default deny incoming`
* Enable SSH (Port 220), NTP and HTTP
    * `sudo ufw allow 2200/tcp`
    * `sudo ufw allow http`
    * `sudo ufw allow ntp`
* Enable UFW with `sudo ufw enable`
### Create Grader's Account
* Create account with `sudo useradd -m grader`
* Grant sudo with `sudo usermod -a -G sudo grader`
* Create ssh-keypair (on the client!) with `ssh-keygen`
* Create key directory in grader's home dir with `sudo mkdir /home/grader/.ssh/`
* Create authorized_keys file `sudo touch /home/grader/.ssh/authorized_keys`
* Adapt the permissions
    * `sudo chmod 700 /home/grader/.ssh`
    * `sudo chown grader:grader /home/grader/.ssh`
    * `sudo chmod 644 /home/grader/.ssh/authorized_keys`
    * `sudo chown grader:grader /home/grader/.ssh/authorized_keys`
* Put the public key into the created file
* Restart SSH with `sudo service ssh restart`
* Disable PW login in `/etc/ssh/ssh_config`
* Disable Root ligin in `/etc/ssh/ssh_config`
### Set Timezone to UTC
* `sudo timedatectl set-timezone Etc/UTC`
### Install and configure Apache
* `sudo apt-get install apache2`
* Install dependencies:
    * `sudo apt-get install python-sqlalchemy`
    * `sudo apt-get install python-flask`
    * `sudo apt-get install python-oauth2client`
    * `sudo apt-get install python-httplib2`
    * `sudo apt-get install python-requests`
    * `sudo apt-get install python-psycopg2`
* Install wsgi module `sudo apt-get install libapache2-mod-wsgi`
* Adapt apache2 config to serve wsgi application. In `/etc/apache2/sites-available/000-default.conf` add `WSGIScriptAlias / /var/www/html/myapp.wsgi`
### Install postgres
* `sudo apt-get install postgresql`
* Make sure that postgres does not allow remote connections. In `/etc/postgresql/9.5/main/postgresql.conf` listen_addresses needs to be either commented out or set to localhost. UFW also blocks all connections on that port.
* Create user item_catalog_dbuser with `sudo -u postgres createuser item_catalog_dbuser`
* Create database with `sudo -u postgres createdb item_catalog`
* Create a password for the user with `sudo -u postgres item_catalog_dbuser`
    * Open psql with `sudo -u postgres psql`
    * Alter password with `psql=# alter user item_catalog_dbuser with encrypted password '<password>';`
    * Limit rights for user and db with `GRANT ALL PRIVILEGES ON DATABASE item_catalog TO item_catalog_dbuser;`
### WSGI-Configuration
* In the web server's root dir checkout the repo with `git clone https://github.com/dkarnutsch/ud-item-catalog`
* Create a file `myapp.wsgi` containing the call to the project.py file.
```
import sys
sys.path.insert(0, '/var/www/html/ud-item-catalog')
    
from project import app as application
```
* reload apache with `sudo service apache2 restart`

## Resources
* Flask and WSGI: http://flask.pocoo.org/docs/0.12/deploying/mod_wsgi/
* PSQL: https://medium.com/coding-blocks/creating-user-database-and-adding-access-on-postgresql-8bfcd2f4a91e
* Server-Timezone: https://www.server-world.info/en/note?os=Ubuntu_16.04&p=timezone
* The rest is based on the course in udacity.
