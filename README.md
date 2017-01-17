LINUX SERVER CONFIGURATION

IP Address: 35.160.116.120

SSH Port : 2200

URL to access Book Catalog Applicaiton: http://ec2-35-160-116-120.us-west-2.compute.amazonaws.com

SETUP:

1. Create a new user named grader                  sudo adduser grader

2. Give the grader the permission to sudo          usermod -aG sudo grader

3. Create SSH-KEY                                  ssh-keygen

4. Installing a public key
    Login as grader
    Go to home directory                            mkdir .ssh
                                                    touch .ssh/authorized_keys
    In local machine                                cd .ssh
                                                    cat .ssh/linuxcourse.pub

    Copy key
                                                    nano .ssh/authorized_keys
    Paste content and save it
                                                    chmod 770 .ssh
                                                    chmod 644 .ssh/authorized_keys
    Login as grader
                                                    sudo nano /etc/ssh/sshd-config
    Change password authentication from yes to no
                                                    sudo service ssh restart

4. Update all currently installed packages          sudo apt-get update
                                                    sudo apt-get upgrade

5. Configure the local timezone to UTC              sudo dpkg-reconfigure tzdata
                                                    When GUI shows up select NONE OF THE ABOVE
                                                    then select UTC

6. Change the SSH port from 22 to 2200              cd /etc/ssh
                                                    vi sshd_config
                                                    change port from 22 to 2200


7. Configure the Uncomplicated Firewall (UFW)
   to only allow incoming connections for           sudo ufw status
   SSH (port 2200)                                  sudo ufw allow 2200
   HTTP (port 80)                                   sudo ufw allow 80
   NTP (port 123)                                   sudo ufw allow 123
                                                    sudo ufw enable
                                                    sudo ufw status

8. Login as root                                    ssh -i ~/.ssh/udacity_key.rsa root@35.160.116.120 -p 2200

9. Login as grader(user)                            ssh grader@35.160.116.120 -p 2200 -i ~/.ssh/myKey

10.Installing Apache                                sudo apt-get install apache2

11.Install mod_wsgi                                 sudo apt-get install python-setuptools libapache2-mod-wsgi
                                                    sudo service apache2 restart

12.Install and configure git                        sudo apt-get install git
                                                    git config --global user.name "YOUR NAME"
                                                    git config --global user.email "YOUR EMAIL ADDRESS"

13.Deploy Flask application on Ubuntu               sudo apt-get install libapache2-mod-wsgi python-dev
   Enable mod_wsgi                                  sudo a2enmod wsgi
   Move to the www directory                        cd /var/www
   Setup a directory for the app, e.g. catalog      sudo mkdir catalog
                                                    cd catalog
                                                    sudo mkdir catalog
                                                    cd catalog
                                                    sudo mkdir static templates
   Create the file that will contain                sudo nano __init__.py
   the flask application logic

   Paste in the following code                      from flask import Flask
                                                    app = Flask(__name__)
                                                    @app.route("/")
                                                    def hello():
                                                        return "Veni vidi vici!!"
                                                    if __name__ == "__main__":
                                                        app.run()

14.Install Flask                                    sudo apt-get install python-pip
                                                    sudo pip install virtualenv
                                                    sudo virtualenv venv
                                                    sudo chmod -R 777 venv
                                                    source venv/bin/activate
                                                    pip install Flask
                                                    python __init__.py
                                                    deactivate

15.Configure and Enable a New Virtual Host#         sudo nano /etc/apache2/sites-available/catalog.conf
    Paste in the following lines of code:
                    <VirtualHost *:80>
                        ServerName PUBLIC-IP-ADDRESS
                        ServerAdmin admin@PUBLIC-IP-ADDRESS
                        WSGIScriptAlias / /var/www/catalog/catalog.wsgi
                        <Directory /var/www/catalog/catalog/>
                            Order allow,deny
                            Allow from all
                        </Directory>
                        Alias /static /var/www/catalog/catalog/static
                        <Directory /var/www/catalog/catalog/static/>
                            Order allow,deny
                            Allow from all
                        </Directory>
                        ErrorLog ${APACHE_LOG_DIR}/error.log
                        LogLevel warn
                        CustomLog ${APACHE_LOG_DIR}/access.log combined
                    </VirtualHost>

    Enable Virtual Host                             sudo a2ensite catalog

16.Create wsgi file                                 cd /var/www/catalog
                                                    sudo vim catalog.wsgi
        Paste in the following lines of code:
                    #!/usr/bin/python
                    import sys
                    import logging
                    logging.basicConfig(stream=sys.stderr)
                    sys.path.insert(0,"/var/www/catalog/")

                    from catalog import app as application
                    application.secret_key = 'Add your secret key'

    Restart Apache                                  sudo service apache2 restart

17. Clone GitHub repository and make it web inaccessible
        git clone https://github.com/ravalig/Book-Catalog.git
    Move all the content to /var/www/catalog/catalog/ directory

18. Make the GitHub repository inaccessible         cd /var/www/catalog/
                                                    sudo vim .htaccess
    Paste the following                             RedirectMatch 404 /\.git

19.  Install needed modules & packages              source venv/bin/activate
                                                    pip install httplib2
                                                    pip install requests
                                                    sudo pip install flask-seasurf
                                                    sudo pip install --upgrade oauth2client
                                                    sudo pip install sqlalchemy
                                                    sudo apt-get install python-psycopg2
                                                    sudo pip install Pillow
                                                    sudo pip install python-resize-image

20. Install and configure PostgreSQL                sudo apt-get install postgresql postgresql-contrib
    Check that no remote connections                sudo vim /etc/postgresql/9.3/main/pg_hba.conf
    are allowed (default)
    Open the database setup file                    sudo vim database_setup.py
    Change the engine                               engine = create_engine('postgresql://
                                                             catalog:Udacity123@localhost/catalog')
    Change the same line in application.py
    Rename application.py                           mv application.py __init__.py
    Create needed linux user for psql               sudo adduser catalog
    Change to default user postgres                 sudo su - postgres
    Connect to the system                           psql
    Add postgre user with password                  CREATE USER catalog WITH PASSWORD 'Udacity123';
                                                    ALTER USER catalog CREATEDB;
    List current roles and their attributes         \du
    Create database                                 CREATE DATABASE catalog WITH OWNER catalog;
    Connect to the database catalog                 \c catalog
    Revoke all rights                               REVOKE ALL ON SCHEMA public FROM public;
    Grant only access to the catalog role           GRANT ALL ON SCHEMA public TO catalog;
    Exit out of PostgreSQl and the postgres user    \q
                                                    exit
    Create postgreSQL database schema               python database_setup.py

21. Open the Apache configuration files
    for the web app                                 sudo vim /etc/apache2/sites-available/catalog.conf
    Paste in the following line below ServerAdmin:
        ServerAlias   ec2-35-160-116-120.us-west-2.compute.amazonaws.com
    Enable the virtual host                         sudo a2ensite catalog

22. Run application                                 sudo service apache2 restart
    Access the application here:
        http://ec2-35-160-116-120.us-west-2.compute.amazonaws.com

23. Checking the errors                             sudo tail -20 /var/log/apache2/error.log


References:

                Udacity Forums
                Stack Overflow
                https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
                https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-14-04
                http://blog.udacity.com/2015/03/step-by-step-guide-install-lamp-linux-apache-mysql-python-ubuntu.html

