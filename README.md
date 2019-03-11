**IP Address:** 13.234.116.153

**APP URL:** http://13.234.116.153.xip.io/

**SSH command:** `ssh -i Pemfile.pem -p 2200 grader@13.234.116.153`

---
## 1. Connect to AWS Lighstail instance
Connect using `ssh -i Lightsail.pem ubuntu@xxx.xxx.xxx.xxx` where xxx.xxx.xxx.xxx is the public IP address.

## 2. Configure ports and firewall
### 2.1. Change SSH port from 22 to 2200
- Run `sudo vim /etc/ssh/sshd_config` to edit the file. Change port `22` to `2200`
- Restart the ssh service by running `sudo service ssh restart`

### 2.2. Configure UFW firewall
- Run `sudo ufw default deny incoming` to block all incoming connections
- Run `sudo ufw default allow outgoing` to allow outgoing connections
- Run `sudo ufw allow 123/udp` to allow NTP on port `123`
- Run `sudo ufw allow ssh` to allow SSH on port `2200`
- Run `sudo ufw deny 22` to block the default SSH port `22`
- Run `sudo ufw allow 2200/tcp` so that the new SSH port works
- Run `sudo ufw enable` to enable UFW firewall

Configure Lightsail instance to update SSH and NTP ports. 
Reconnect to the instance using `ssh -i Lightsail.pem -p 2200 ubuntu@xxx.xxx.xxx.xxx`

## 3. Create a new user 'grader'
This user has sudo rights.

- Create a user using `sudo adduser grader`. Give a password and fill required details
- Create `/etc/sudoers.d/grader` file and add `grader ALL=(ALL) NOPASSWD:ALL` to give it sudo rights
- Create a folder `~/.ssh` and give it permission `700`
- Generate new SSH key-pair by running `ssh-keygen -t rsa`
- Create a new file `~/.ssh/authorized_keys` and give it permission `600`
- Copy the contents of `~/.ssh/id_rsa.pub` inside `~/.ssh/authorized_keys`
- Copy the contents of `~/.ssh/id_rsa` to a file in your local machine and save it as `grader.pem`
- Delete both the `~/.ssh/id_rsa.pub` and `~/.ssh/id_rsa` files
- Disconnect from the instance and reconnect as grader by running `ssh -i grader.pem -p 2200 grader@xxx.xxx.xxx.xxx`

## 4. Install and configure Apache server
Run `sudo apt-get install apache2` to install Apache server. It will be running on port 80 by default. Confirm by visiting the public address of the instance via browser.

The instructions on [this site](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps) was followed exactly to configure Apache.

The Apache config file `/etc/apache2/sites-available/FlaskApp.conf` looks as:

```XML
<VirtualHost *:80>
		ServerName http://13.234.116.153.xip.io/
		ServerAdmin siddharth.thevaril@gmail.com
		WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi
		<Directory /var/www/FlaskApp/FlaskApp/>
			Order allow,deny
			Allow from all
		</Directory>
		Alias /static /var/www/FlaskApp/FlaskApp/static
		<Directory /var/www/FlaskApp/FlaskApp/static/>
			Order allow,deny
			Allow from all
		</Directory>
		ErrorLog ${APACHE_LOG_DIR}/error.log
		LogLevel warn
		CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

## 5. Install python and its dependencies
Install the following:
- Run `sudo apt-get install python python-pip` to install python and pip
- Run `sudo pip install --upgrade pip` to upgrade pip
- Run `sudo apt-get install flask packaging oauth2client passlib flask-httpauth psycopg2-binary requests` to install necessary python modules

## 6. Install Postgres
**Run the below commands by activating virtual environment**
- Run `sudo apt-get install postgresql` to install Postgres
- Switch user to postgres using `sudo su - postgres`
- Run `psql` to connect to Postgres via its terminal
- Create a role using `CREATE ROLE grader WITH LOGIN;`
- Give role the authority to create a database by running `ALTER ROLE catalog CREATEDB;`
- Give the role grader a password by running `\password grader`. Then run `exit` to return to the previous user
- Run `createdb catalog` to create a database. This will be owned by `grader`

## 7. Setup the Catalog Project
- Run `git clone https://github.com/Sidsector9/FSND-Item-Catalog.git .` inside `/var/www/FlaskApp/FlaskApp`
- Rename the file `server-catalog.py` to `__init__.py`
- Delete all the files ending in `.pyc`
- Replace `app.run(host='0.0.0.0', port=5000)` to `app.run()` inside `__init__.py`
- Search-replace `credentials.json` to `/var/www/FlaskApp/FlaskApp/credentials.json` in all the files
- Search-replace `sqlite:///catalog.db` to `postgresql://grader:<PASSWORD>@localhost/catalog` in all the files

## 8. Fill up the category table
**Only Items can be added, edited and deleted, categories can't.**
- Run `sudo su - postgres`
- Run `psql`
- Run `\c catalog` to connect to the catalog database
- Run `INSERT INTO category (id, name, slug) VALUES (val1, val2, val3);` to fill up the category

Articles referred:
- [Connect to a Postgres database](https://stackoverflow.com/questions/3949876/how-to-switch-databases-in-psql)
- [Insert values in the table](http://www.postgresqltutorial.com/postgresql-insert/)
