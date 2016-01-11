---
layout: post
title: Install CKAN On Centos 7
---

# Install CKAN On Centos 7

## Background

CKAN document, release 2.6.0a, gives three options to install CKAN.

* Installing CKAN from package.
* Installing CKAN from source.
* Install using Docker.

In this post, I will discuss the 2nd option, Installing CKAN from source, as it is more generic in many aspects.
Please refer to CKAN document for details.

This post will show you how to install CKAN in a corporation production environment.

The OS is Centos 7 on a vmware virtual machine.

## Install Centos 7

I use the image CentOS-7-x86_64-Everything-1503-01.iso to install Centos 7.
Set up network proxy and date/time properly.

In /etc/sudoers add the ckan user for sudo. 
 
```
	ckan-user   ALL(=ALL)   ALL
```

In /etc/yum.conf, add the following to enable proxy for yum.

```
    proxy=http://proxy_server:port_number
    proxy_username=username
    proxy_password=password
```


## Install the required packages

CKAN document says that if you are using a Debian-based operating system (such as Ubuntu) install the required packages with this command:
```
	sudo apt-get install python-dev postgresql libpq-dev python-pip python-virtualenv git-core solr-jetty 
```

In my case, I need to install the packages by myself on Centos 7.

| Package     | Description                                        |
|-------------|:--------------------------------------------------:|
| Python      | The Python programming language, v2.6 or 2.7       |
| PostgreSQL  | The PostgreSQL database system, v8.4 or newer      |
| libpq       | The C programmer’s interface to PostgreSQL         |
| pip         | A tool for installing and managing Python packages |
| virtualenv  | The virtual Python environment builder             |
| Git         | A distributed version control system               |
| Apache Solr | A search platform                                  |
| Jetty       | An HTTP server (used for Solr).                    |
| OpenJDK 6   | JDK The Java Development Kit                       |

### Python

Centos 7 comes with Python 2.7. Check with commend below.
```
	python --version
```

### PostgreSql

```
	sudo yum install postgresql-server postgresql-contrib
	sudo postgresql-setup initdb
```

Update file /var/lib/pgsql/data/pg_hba.conf
```
	sudo vi /var/lib/pgsql/data/pg_hba.conf
```
Change 
```
	host    all             all             127.0.0.1/32            ident
	host    all             all             ::1/128                 ident
```

to
```
	host    all             all             127.0.0.1/32            md5
	host    all             all             ::1/128                 md5
```

This allows Postgresql to authenticate with password.

Start Postgresql.
```
	sudo systemctl start postgresql
	sudo systemctl enable postgresql
```

Sudo as user postgre.
```
	sudo -i -u postgres
```

Now, I can psql.
```
	psql
	\q
```

### libpq
```
	yum install postgresql-devel
```

This will install /usr/lib64/libpq.so.

### pip
Install pip after proxy.
```
	sudo rpm --httpproxy http://<proxy server>:<proxy port>@<proxy user>:<proxy password> -iUvh http://dl.fedoraproject.org/pub/epel/7/x86_64/e/epel-release-7-5.noarch.rpm
```

In /etc/yum.repos.d/epel.repo, uncomment baseurl.
```
	sudo yum -y install python-pip
```

### virtualenv
```
	sudo yum install python-virtualenv
```

### Git
```
	sudo yum install git-core
```

### Apache Solr and Jetty
```
	sudo adduser solr
	cd /opt
	sudo curl --proxy http://10.150.17.10:3128 --proxy-user operyhg:Qyy2004Hyf -O http://www.trieuvan.com/apache/lucene/solr/5.4.0/solr-5.4.0.tgz
	sudo tar zxvf solr-5.4.0.tgz
	sudo cp /opt/solr-5.4.0/bin/install_solr_service.sh .
	sudo rm -rf solr-5.4.0
	sudo ./install_solr_service.sh solr-5.4.0.tgz
```

Then enable solr auto-start at reboot.
```
	sudo chkconfig --add solr
	sudo chkconfig | grep solr
```

Set up firewall.
```
	sudo firewall-cmd --zone=public --add-port=8983/tcp --permanent
	sudo firewall-cmd --reload
```

Useful commands for firewall.
```
	systemctl disable/stop/start/status firewalld
```

Useful commands for solr.
```
	sudo service solr restart
	sudo service solr stop
	sudo service solr start
```

### OpenJDK 7
```
	sudo yum -y install java-1.7.0-openjdk
	sudo yum -y install java-1.7.0-openjdk-devel
```

## Install CKAN into a Python virtual environment

### Preparation
Under ckan user,  run the following.
```
	mkdir -p ~/ckan/lib
	sudo ln -s ~/ckan/lib /usr/lib/ckan
	mkdir -p ~/ckan/etc
	sudo ln -s ~/ckan/etc /etc/ckan
```

### Create Python virtual environment (virtualenv)
Under ckan user,  run the following.
```
	sudo mkdir -p /usr/lib/ckan/default
	sudo chown `whoami` /usr/lib/ckan/default
	virtualenv --no-site-packages /usr/lib/ckan/default
	. /usr/lib/ckan/default/bin/activate
```

### Install CKAN source code into virtualenv
Here I set up NTLMAPS for proxy, then point git to that proxy (http.proxy A113361.ad.internal.osr.nsw.gov.au:5865).
```
	git config --global http.proxy A113361.ad.internal.osr.nsw.gov.au:5865
	git config --global https.proxy A113361.ad.internal.osr.nsw.gov.au:5865
	git config --global http.sslVerify false
	git config --global https.sslVerify false
```

The .gitconfig file looks like
```
	[http]
		proxy = A113361.ad.internal.osr.nsw.gov.au:5865
		sslVerify = false
	[https]
		proxy = A113361.ad.internal.osr.nsw.gov.au:5865
		sslVerify = false
```

For CKAN 2.5.1, do
```
	pip install --proxy http://user:passwd@proxy-server:proxy-port -e `git+https://github.com/ckan/ckan.git@ckan-2.5.1#egg=ckan'
```
For CKAN latest dev version, do
```
	pip install --proxy http://user:passwd@proxy-server:proxy-port -e 'git+https://github.com/ckan/ckan.git#egg=ckan'
```

### Install Python modules for CKAN in virtualenv
```
	pip install --proxy http://user:passwd@proxy-server:proxy-port  -r /usr/lib/ckan/default/src/ckan/requirements.txt
```

For some reason (many discussions in google), the command above may not work for all the requirements. My workaound is to split requirements.txt into 
small chunks and repeat the command.

### Reactivate virtualenv
```
	deactivate
	. /usr/lib/ckan/default/bin/activate
```

## Setup PostgreSQL database
List existing database:
```
	sudo -u postgres psql -l
```
Check that the encoding of databases is UTF8.

Create a new database user ckan_default.
```
	sudo -u postgres createuser -S -D -R -P ckan_default
```

Create a new PostgreSQL database ckan_default, owned by user ckan_default.
```
	sudo -u postgres createdb -O ckan_default ckan_default -E utf-8
```

Note: If PostgreSQL is run on a separate server, you will need to edit postgresql.conf and pg_hba.conf. 

Uncomment the listen_addresses parameter and specify a comma-separated list of IP addresses of the network interfaces
PostgreSQL should listen on or '*' to listen on all interfaces. For example,
```
	listen_addresses = 'localhost,192.168.1.21'
```

Add a line similar to the line below to the bottom of pg_hba.conf to allow the machine running Apache to connect to
PostgreSQL. Please change the IP address as desired according to your network settings.
```
	host all all 192.168.1.22/32 md5
```

## Create CKAN config file
Create a directory to contain the site's config files:
```
	sudo mkdir -p /etc/ckan/default
	sudo chown -R `whoami` /etc/ckan/
```

Create the CKAN config file:
```
	paster make-config ckan /etc/ckan/default/development.ini
```