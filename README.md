# Linux Server Configuration

A baseline installation of Linux on AWS Lightsail configured to host one of my python web app project.

### Pubic IP

```
http://34.244.19.210/
```

### Grader SSH Details

The public key is located in this repo. You can SSH into the server using the below:

```
ssh -i <private_key> grader@34.244.19.210 -p 2200
```

And enter the private key passphrase

```
<passphrase>
```
## Deployment Process
### Initial Configuration

1. Sign up for AWS and create a Lightsail instance.
2. Connect to the instance from local PC using public/private keys created by lightsail.
3. Create a "grader" user and provide this user with sudo access.
4. Create a public/private key pair (on local machine) for this user and add the public key to the lightsail authorized_keys file.
5. Update all installed packages and change the SSH port from 22 to 2200 - Replicate this on the lightsail webapp.
6. Created a snapshot of the instance
7. Implemented UFW settings restricting connections on the following ports: SSH (port 2200), HTTP (port 80), and NTP (port 123).

### Environment Set Up
1. Installed Apache, mod_wsgi & python-dev
    ```
    sudo apt-get install apache2
    sudo apt-get install libapache2-mod-wsgi
    sudo apt-get install python-dev
    ```
2. Enter the /var/www and create a project directory
    ```
    cd /var/www
    sudo mkdir ecommerce
    ```
3. Enter project dir and clone web app repo
    ```
    cd ecommerce
    sudo git clone https://github.com/oreilm49/ecommerce-flask
    ```
4. Create Environment: install pip & virtualenv, then create a virtual environment.
    ```
    sudo apt-get install python-pip
    sudo pip install virtualenv
    sudo virtualenv ecommerce
    source ecommerce/bin/activate
    ```
5. Install flask
    ```
    sudo pip install Flask
    ```
### Configure virtual host
1. Create the VirtualHost file
    ```
    sudo nano /etc/apache2/sites-available/ecommerce.conf
    ```
2. Add the following content then save and close the file
    ```
    <VirtualHost *:80>
            ServerName 34.244.19.210
            ServerAdmin mark@markoreilly.xyz
            WSGIScriptAlias / /var/www/ecommerce/ecommerce.wsgi
            <Directory
            /var/www/ecommerce/ecommerce-flask>
                    Order allow,deny
                    Allow from all
            </Directory>
            Alias /static
            /var/www/ecommerce/ecommerce-flask/static
            <Directory
            /var/www/ecommerce/ecommerce-flask/static/>
                    Order allow,deny
                    Allow from all
            </Directory>
            ErrorLog ${APACHE_LOG_DIR}/error.log
            LogLevel warn
            CustomLog ${APACHE_LOG_DIR}/access.log combined
    </VirtualHost>
    ```
3. Enable the virtualhost
    ```
    sudo a2ensite ecommerce
    ```
### Create the .wsgi File
1. Move to /var/www/ecommerce dir and create the wsgi file
    ```
    sudo nano ecommerce.wsgi
    ```
2. Add the following content
    ```
    #!/usr/bin/python
    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0,"/var/www/ecommerce/ecommerce-flask/")

    from server import app as application
    application.secret_key = 'super_secret_key'
    ```
### Restart Apache
    ```
    sudo service apache2 restart
    ```
## Third party software

- Ubuntu        -   operating system
- Python        -   runtime
- PIP           -   package manager
- Apache        -   web server
- Postgresql    -   database