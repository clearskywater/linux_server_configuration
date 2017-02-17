# Linux Server Configuration Project

# IP address: http://35.161.133.72

# Port: 2200

# URL: http://ec2-35-161-133-72.us-west-2.compute.amazonaws.com

# Server login: grader
# Password: 123456

# 1.launch virtual machine
    a.install VirtualBox
	  b.install Vagrant
    c.create a new folder for the project
	  d.open a new terminal and go to the new folder
	  e.vagrant init ubuntu/trusty64
	  f.vagrant up
	  g.vagrant ssh

# 2.ssh into the server with udacity account
    a.download and save private key as udacity_key.rsa
	  b.copy udacity_key.rsa to ~/.ssh/
	  c.chmod 600 ~/.ssh/udacity_key.rsa
	  d.ssh -i ~/.ssh/udacity_key.rsa root@35.161.133.72
		e.chmod 600 ~/.ssh/grader_key.rsa
		f.ssh -i ~/.ssh/grader_key.rsa grader@35.161.133.72 -p 2200

# 3.create a new user named grader
    a.adduser grader

# 4.give the grader the permission to sudo
    a.change directory to /ect/sudoers.d
	  b.create a new file: grader
	  c.in the new file above add the content: grader ALL=(ALL) NOPASSWD:ALL
    d.create a ssh key for grader
      1).run this command in the local terminal: ssh-keygen
      2).in the server terminal run the command: su grader
	    3).mkdir ~/.ssh
	    4).change directory to ~/.ssh
	    5).create a new file: authorized_keys
	    6).copy the content of .pub file generated in step a into authorized_keys
	    7).chmod 700 ~/.ssh
	    8).chmod 644 ~/.ssh/authorized_keys

# 5.update all currently installed packages
	  a.sudo apt-get update
	  b.sudo apt-get upgrade

# 6.change the SSH port from 22 to 2200
    a.open /etc/hosts and add this information: 127.0.1.1 ip-10-20-47-41 
	  b.open /etc/ssh/sshd-config
	  c.change "Port 22" to "Port 2200"
	  d.Change "PermitRootLogin without-password" to "PermitRootLogin no"
	  e.sudo service sshd restart

# 7.configure the uncomplicated firewall
    a.sudo ufw allow 2200/tcp
    b.sudo ufw allow www
    c.sudo ufw allow 123/udp
    d.sudo ufw enable		

# 8.configure the local timezone to UTC
    a.udo dpkg-reconfigure tzdata
    b.select UTC timezone

# 9.install and configure Apache to serve a Python mod_wsgi application
    a.sudo apt-get install apache2
    b.sudo apt-get install libapache2-mod-wsgi 	
		c.sudo apt-get install python-dev
		d.sudo apt-get install libpq-dev

# 10.install and configure PostgreSQL
    a.sudo apt-get install postgresql		
		b.sudo su - postgres
		c.psql
		d.CREATE USER catalog WITH PASSWORD '123456';
		e.ALTER USER catalog CREATEDB;
		f.CREATE DATABASE catalog WITH OWNER catalog;
		g.\c catalog
		h.REVOKE ALL ON SCHEMA public FROM public;
		i.GRANT ALL ON SCHEMA public TO catalog;
		j.quit with Control + d

# 11. install git, clone and setup catalog app project
    a.sudo apt-get install git
    b.cd /var/www
    c.git clone https://github.com/clearskywater/catalog.git		
		d.cd catalog/.git
		e.create .htaccess file and add this content:
		  Order allow,deny
			Deny from all
	  f.go to catalog directory
    g.rename application.py to catalog.py
		h.sudo apt-get install python-pip
    i.sudo pip install flask httplib2 requests oauth2client sqlalchemy psycopg2
	  j.create catalog.wsgi and add this content:
      #!/usr/bin/python
      import sys
      import logging
      logging.basicConfig(stream=sys.stderr)
      sys.path.insert(0,"/var/www/catalog")
      
      from catalog import app as application
	    application.secret_key = '123456'		
	  k.add the following content to /etc/apache2/sites-available/catalog.conf

      <VirtualHost *:80>
          ServerName 35.161.133.72
          ServerAlias ec2-35-161-133-72.us-west-2.compute.amazonaws.com
          ServerAdmin admin@ec2-35-161-133-72.us-west-2.compute.amazonaws.com
          WSGIScriptAlias / /var/www/catalog/catalog.wsgi
          <Directory /var/www/catalog/>
              Order allow,deny
              Allow from all
          </Directory>
          Alias /static /var/www/catalog/static
          <Directory /var/www/catalog/static/>
              Order allow,deny
              Allow from all
          </Directory>
          ErrorLog ${APACHE_LOG_DIR}/error.log
          LogLevel warn
          CustomLog ${APACHE_LOG_DIR}/access.log combined
      </VirtualHost>

    l.sudo a2ensite catalog
	  m.open catalog.py and add full path for .json files
		n.change app oauth path to:
	  	http://ec2-35-161-133-72.us-west-2.compute.amazonaws.com 
	  o.open catalog.py and database_setup.py. Chane the database path to:
		  postgresql://catalog:password@localhost/catalog
		p.sudo service apache2 restart

# 12. summary of software installed
    		
    apache2
    libapache2-mod-wsgi 	
		python-dev
		libpq-dev
		postgresql
		git
		python-pip
		flask 
		httplib2 
		requests 
		oauth2client 
		sqlalchemy 
		psycopg2
