# Project-01: Setup Web Application Stack on Local Machine 

In this project we will setup the Web Application Stack on the local machine using, this stack includes following tools and services.

- NGINX: This will be act as load balancer and forward user requests to the Apache Tomcat application server.

- Apache Tomcat: This will be our application server to deploy the java web application.

- RabbitMQ: Used as message broker for the application.

- Memcached: Used to cache data from the MySQL database server for faster user performance. 

- MySQL: Database server for the application.


## Pre-requisite 

- Oracle Virtual Box (for Windows)/ VMware Fusion Tech (for MacOS)
- Vagrant (for VM automation)
- Vagrant Plugin (vagrant-hostmanager/vagrant-vbguest/vagrant-vmware-desktop) 
- Git
- JDK8
- Maven 
- IDE (VSCode)


## Architecture 

![GitHub Light](./snaps/web-app-stack-local.png)

## Step 1: Setup Virtual Machines


Clone the repository

```
git clone https://github.com/vijaylondhe/vprofile-project.git
```

```
cd vagrant/Manual_provisioning_MacOSM1
```

Run the virtual machines

```
vagrant up
```

Note: This will bring up all the virtual machines listed in Vagrantfile, execute the below command to see the status of virtual machines 

```
vagrant status
```

![GitHub Light](./snaps/vagrant_status.png)


### Setup VM for MySQL 


Login into MySQL Virtual Machine:

```
vagrant ssh db01
```

Check host file is automatically updated by host-manager vagrant plugin.

```
cat /etc/hosts
```

![GitHub Light](./snaps/vagrant_db01_host_file.png)


Update the repository and install MariaDB package

```
sudo yum update -y
```

```
sudo yum install epel-release -y 
```

```
sudo yum install git mariadb-server -y

```

Start and Enable MariaDB server

```
sudo systemctl start mariadb
sudo systemctl enable mariadb
```

Run the database installation script 

```
sudo mysql_secure_installation
```

- [ ] *Set user password: admin123*

- [ ] *Remove Anonymous User? Y*

- [ ] *Disallow root login remotely? Y*

- [ ] *Remove test database and access to it? Y*

- [ ] *Reload privilege tables now? Y*


Set Database and user details 

```
sudo mysql -u root -padmin123
```

```
mysql> create database accounts;
mysql> grant all privileges on accounts.* TO 'admin'@'db01' identified by 'admin123';
mysql> FLUSH PRIVILEGES;
mysql> exit;
```

Download the source code and initialize the database

```
git clone https://github.com/vijaylondhe/vprofile-project.git
cd /home/vagrant/vprofile-project
sudo mysql -u root -padmin123 accounts < src/main/resources/db_backup.sql
sudo mysql -u root -padmin123 accounts
mysql> show tables;
```

Restart the database service

```
sudo systemctl restart mariadb
```

Configure firewall for mariadb database on port 3306

```
sudo systemctl start firewalld
sudo systemctl enable firewalld
sudo firewall-cmd --get-active-zones
sudo firewall-cmd --zone=public --add-port=3306/tcp --permanent 
sudo firewall-cmd --reload
sudo systemctl restart mariadb
```


### Setup VM for Memcached

Login to mc01 virtual machine and install memcached package.

```
vagrant ssh mc01
sudo yum update -y 
sudo yum install memcached -y
```

Start and enable the service 

```
sudo systemctl start memcached
sudo systemctl enable memcached
sudo systemctl status memcached
```

Configure firewall for memcached port 11211

```
sudo firewall-cmd --add-port=11211/tcp --permanent
sudo firewall-cmd --reload
```

Make the changes in memcached configuration file

```
sudo sed -i 's/OPTIONS="-l 127.0.0.1"/OPTIONS=""/' /etc/sysconfig/memcached
```

Restart and configure the memcached service 

```
sudo systemctl restart memcached 
sudo memcached -p 11211 -U 11111 -u memcache -d
```

### Setup VM for RabbitMQ

Login to rmq01 instance and install the required packages

```
vagrant ssh rmq01
sudo yum update -y
```

Disable SELINUX on the virual machine 

```
sudo sed -i 's/SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config
sudo setenforce 0
```

Install dependencies and rabbitmq-server package

```
curl -s https://packagecloud.io/install/repositories/rabbitmq/erlang/script.rpm.sh | sudo bash
sudo yum clean all
sudo yum makecache
sudo yum install erlang -y
sudo yum install rabbitmq-server -y
```

Start and enable the service 

```
sudo systemctl start rabbitmq-server
sudo systemctl enable rabbitmq-server
sudo systemctl status rabbitmq-server
```

Make configuration changes in configuration file

```
sudo sh -c 'echo "[{rabbit, [{loopback_users, []}]}]." > /etc/rabbitmq/rabbitmq.config'
sudo rabbitmqctl add_user test test
sudo rabbitmqctl set_user_tags test administrator
```

Configure firewall for rabbitmq-server port 

```
sudo firewall-cmd --add-port=5671/tcp --permanent 
sudo firewall-cmd --add-port=5672/tcp --permanent # firewall-cmd --reload
sudo firewall-cmd --zone=public --add-port=25672/tcp --permanent    #This port is used for inter-node communication.
sudo firewall-cmd --reload
sudo systemctl restart rabbitmq-server
sudo reboot
```

