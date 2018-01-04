# linux-project

IP: 18.217.1.39
SSH: 2200

URL: http://ec2-18-217-1-39.us-east-2.compute.amazonaws.com/



# Summary

1. *Update all currently installed packages.*

```bash
sudo apt update    # update pakage reference
sudo apt upgrade   # install the available updates
```


2. *Change the SSH port from **22** to **2200**. Make sure to configure the Lightsail firewall to allow it.*

Install `PuTTY`.

Download private key `LightsailDefaultPrivateKey-us-east-2.pem` from 
https://lightsail.aws.amazon.com/ls/webapp/account/keys .

Using `PuTTYgen`, change format of the private key and make a public key.

Configure and save the connection to Amazon Lightsail instance.

Open session to Lightsail with `PuTTY`.

```bash
$ sudo nano /etc/ssh/sshd_config    # change port number, PermitRootLogin no
$ sudo service ssh restart
```
Update information in `PuTTY` connection session.



3. *Edit the Firewall.*

```bash
# note
$ sudo ufw default deny incoming
$ sudo ufw default allow outgoing

$ sudo ufw deny ssh    # ssh = 22
$ sudo ufw allow 2200/tcp    # ssh
$ sudo ufw allow www    # www = 80
$ sudo ufw allow ntp    # ntp = 123

$ sudo ufw enable
$ sudo ufw status
```

Add custom rule on Amazon Firewall: `Custom	TCP	2200`



4. *Give `grader` access.*

```bash
$ sudo adduser grader    # Udacity Linux Project Grader
$ sudo nano /etc/sudoers.d/grader    #grader ALL=(ALL) NOPASSWD:ALL
$ sudo su grader

grader@ip:/home/ubuntu$ sudo apt-get install apache2
grader@ip:/home/ubuntu$ sudo apt-get install postgresql

grader@ip:/home/ubuntu$ cd
grader@ip:~$ mkdir .ssh
grader@ip:~$ nano .ssh/authorized_keys    # contents of the public key
grader@ip:~$ chmod 700 .ssh
grader@ip:~$ chmod 644 .ssh/authorized_keys
```

5. *Apache*

The configuration layout for an Apache2 web server installation on Ubuntu systems is as follows:

```
/etc/apache2/
|-- apache2.conf
|       `--  ports.conf
|-- mods-enabled
|       |-- *.load
|       `-- *.conf
|-- conf-enabled
|       `-- *.conf
|-- sites-enabled
|       `-- *.conf
```

`sudo service apache2 status`



6. Mod_WSGI

```bash
$ sudo apt-get install libapache2-mod-wsgi
$ sudo nano /etc/apache2/sites-enabled/000-default.conf
```

```
<virtualHost *>
    ...
    WSGIScriptAlias / /var/www/html/myapp.wsgi
</VirtualHost>
```

```bash
$ sudo apache2ctl restart
$ sudo nano /var/www/html/myapp.wsgi
```

```python
def application(environ, start_response):
    status = '200 OK'
    output = 'Hello Udacity!'

    response_headers = [('Content-type', 'text/plain'), ('Content-Length', str(len(output)))]
    start_response(status, response_headers)

    return [output]
```



7. Postgres

```sql
grader@ip:~$ sudo -u postgres psql    # logs you in as the psql superuser
postgres=# CREATE USER grader CREATEDB;
postgres=# CREATE USER catalog;
postgres=# \q
grader@ip:~$ psql -d postgres    # logs you in to the psql database
postgres=> CREATE DATABASE grader;
postgres=> \q

grader@ip:~$ sudo -u postgres psql
postgres=# GRANT ALL PRIVILEGES ON DATABASE grader TO grader;
postgres=# \q
```



8. Project Deployment

download project from git repository: `git clone [URL_PATH]`

```bash
grader@ip:~$ mkdir projects
grader@ip:~$ cd projects
grader@ip:~/projects$ git clone https://github.com/okochepasova/restaurant-item-catalog.git
grader@ip:~/projects$ cd restaurant-item-catalog
grader@ip:~/projects/restaurant-item-catalog$ nano myapp.wsgi
```

```python
import sys
sys.path.insert(0, '/home/grader/projects/restaurant-item-catalog')

import configuration
configuration.setup_lightsail()

from project import app as application
application.secret_key = configuration.secret_key

```

```bash
$ sudo apt-get install python-pip
$ sudo su root
# cd
# pip install --upgrade pip
# pip install flask
# pip install sqlalchemy
# pip install oauth2client
# pip install requests
# pip install psycopg2
# ^D
$ sudo nano /etc/apache2/sites-enabled/000-default.conf
```

```
<virtualHost *>
    <Directory /home/grader/projects/restaurant-item-catalog>
        Require all granted
    </Directory>

    WSGIDaemonProcess localhost user=grader group=grader home=/home/grader/projects/restaurant-item-catalog
    WSGIScriptAlias / /home/grader/projects/restaurant-item-catalog/myapp.wsgi
    ...
    #WSGIScriptAlias / /var/www/html/myapp.wsgi
</VirtualHost>
```

```bash
$ psql < catalog_for_postgress.sql
$ sudo service apache2 restart
```



9. Notes


I've edited my `item catalog` project to make it work on the amazon server, and I'm not 
including all the changes I've made in the steps.

