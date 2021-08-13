# Airflow Cluster Installation on Ubuntu 20.04

This repository is a guide for Apache Airflow installation on Ubuntu 20.04 with cluster mode.

## Step 1 - Install Anaconda

You can find a step by step guide on how to install anaconda here [https://github.com/ozgunakin/anaconda-installation-on-ubuntu20.04](https://github.com/ozgunakin/anaconda-installation-on-ubuntu20.04)

## Step 2 - Create a Conda Environment

Creating an environment is important to easily manage pip packages.

```text
conda create --name airflowenv

conda activate airflowenv
```

## Step 3 - Create an Airflow Directory \(Optional\)

Actually, you don't need to create a directory for airflow, because airflow automatically creates it under your home directory. In our case, we want to store airflow files line dags, configs, and PID's in another location\(/data/airflow\). That is why we are creating a directory for it.

```text
mkdir /data/airflow
```

## Step 4 - Install Necessary Packages

```text
sudo apt-get install libmysqlclient-dev
pip install mysqlclient
sudo apt-get -y install gcc
pip install celery==4.4.7
pip install flower==0.9.4
```

## Step 5 - Install Airflow

We will install airflow 2.1.0 using conda-forge to our airflowenv.

```text
conda install -c conda-forge airflow=2.1.0
```

## Step 6 - Configure Environment Variables

We need to set AIRFLOW\_HOME and AIRFLOW\_CONFIG paths in .bashrc file of our user because we don't want to use the default home directory.

```text
nano ~/.bashrc
```

* [x] Enter following lines into .bashrc file and save the file
* export AIRFLOW\_HOME=/data/airflow
* export AIRFLOW\_CONFIG=/data/airflow/airflow.cfg

```text
source ~/.bashrc
```

## Step  7 - Install MySQL \(If you don't already have it\)

You can find a step by step guide on how to install MySQL here [https://github.com/ozgunakin/mysql-installation-on-ubuntu20.04](https://github.com/ozgunakin/mysql-installation-on-ubuntu20.04)

## Step 8 - Create an Airflow User in MySQL

You need to create an airflow database and a user who is permitted to write on this table.

* [x] Create an airflow database

```text
sudo mysql

CREATE DATABASE airflow;
```

* [x] Create and authorize an airflow user.

```text
CREATE USER 'airflow'@'%' IDENTIFIED BY 'sczq6vP!';
GRANT ALL PRIVILEGES ON airflow.* TO 'airflow'@'%' WITH GRANT OPTION;
```

## Step 9 - Install RabbitMQ 

You can find a step by step guide on how to install RabbitMQ here [https://github.com/ozgunakin/rabbitmq-installation-on-ubuntu20.04](https://github.com/ozgunakin/rabbitmq-installation-on-ubuntu20.04)

## Step 10 - Create an Airflow User in RabbitMQ  

You need to create and authorize an airflow user in RabbitMQ.

```text
sudo rabbitmqctl add_user airflow 5Cmj5bw8
sudo rabbitmqctl set_user_tags airflow administrator
```

## Step 11 - Create a Virtual Host in RabbitMQ  

* [x] Go to RabbitMQ URL on your browser. [http://YOUR\_IP:15672](http://10.115.209.137:15672/#/users)
* [x] Click the Admin button. Then click the virtual hosts button which is placed right on the page.

![](.gitbook/assets/image%20%281%29.png)

* [x] Click "Add a new virtual host" and enter your hostname as airflow. Then click "Add a virtual host".

![](.gitbook/assets/image%20%282%29.png)

## Step 12 - Configure Airflow

* [x] Open the airflow configuration file.

```text
sudo nano /data/airflow/airflow.cfg
```

* [x] Change the following lines on the configuration file. **On Master and Workers.**
* executor = CeleryExecutor
* sql\_alchemy\_conn = mysql://airflow:sczq6vP!@localhost:3306/airflow
* broker\_url = amqp://airflow:5Cmj5bw8@10.115.209.137:5672/airflow
* result\_backend = db+mysql://airflow:sczq6vP!@10.115.209.137:3306/airflow

## Step 13 - Initilaze Airflow db

```text
airflow db init
```

## Step 14 - Create Admin User for Webserver \(Only on master\)

```text
airflow users create \
--email admin@admin.com --firstname admin \
--lastname admin --password admin \
--role Admin --username admin
```

## Step 15 - Run Airflow for Testing

* [x] Run webserver. \(ON MASTER\)

```text
airflow webserver
```

* [x] Run scheduler \(Run the code using a different terminal and do not close the terminal before you finish your test\)

```text
airflow scheduler
```

* [x] Run worker. \(ON WORKERS\)

```text
airflow celery worker
```

Go to your webserver URL \([http://YOUR\_IP:8080](http://10.113.200.130:8080/home)\) and run some tests on airflow. After you finish your tests follow the steps below to stop airflow on all nodes.

* [x] List PID's for airflow services

```text
ps aux | grep airflow
```

* [x] Kill all PID's you have seen on the list.

```text
kill -9 "PID"
```

## Step 16 - Create Airflow Environment File

* [x] Before you create services you need to create an airflow.env file then write the airflow home and config file path in this file.

```text
sudo nano /data/airflow/airflow.env
```

* [x] Enter the following lines and save the file.
* AIRFLOW\_CONFIG=/data/airflow/airflow.cfg 
* AIRFLOW\_HOME=/data/airflow

## Step 17 - Create airflow-webserver Service. \(**ONLY ON MASTER**\)

```text
sudo nano /etc/systemd/system/airflow-webserver.service
```

* [x] Enter the following lines and save the file.

```text
[Unit]
Description=Airflow webserver daemon
After=mysql.service rabbitmq-server.service
Wants=mysql.service rabbitmq-server.service

[Service]
EnvironmentFile=/data/airflow/airflow.env
User=dtpammlai
Group=dtpammlai
Type=simple
ExecStart=/bin/bash -c 'source /data/anaconda3/etc/profile.d/conda.sh; \
    conda activate airflow; \
    airflow webserver'
Restart=on-failure
RestartSec=5s
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

* [x] Start and enable the service

```text
sudo service airflow-webserver start
sudo systemctl enable airflow-webserver
```

## Step 18 - Create airflow-scheduler Service. \(**ONLY ON MASTER**\)

```text
sudo nano /etc/systemd/system/airflow-scheduler.service
```

* [x] Enter the following lines and save the file.

```text
[Unit]
Description=Airflow webserver daemon
After=mysql.service rabbitmq-server.service
Wants=mysql.service rabbitmq-server.service

[Service]
EnvironmentFile=/data/airflow/airflow.env
User=dtpammlai
Group=dtpammlai
Type=simple
ExecStart=/bin/bash -c 'source /data/anaconda3/etc/profile.d/conda.sh; \
    conda activate airflow; \
    airflow scheduler'
Restart=on-failure
RestartSec=5s
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

* [x] Start and enable the service

```text
sudo service airflow-scheduler start
sudo systemctl enable airflow-scheduler
```

## Step 19 - Create airflow-worker Service. \(**ONLY ON WORKERS**\)

```text
sudo nano /etc/systemd/system/airflow-worker.service
```

* [x] Enter the following lines and save the file.

```text
[Unit]
Description=Airflow webserver daemon
After=mysql.service rabbitmq-server.service
Wants=mysql.service rabbitmq-server.service

[Service]
EnvironmentFile=/data/airflow/airflow.env
User=dtpammlai
Group=dtpammlai
Type=simple
ExecStart=/bin/bash -c 'source /data/anaconda3/etc/profile.d/conda.sh; \
    conda activate airflow; \
    airflow scheduler'
Restart=on-failure
RestartSec=5s
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

* [x] Start and enable the service

```text
sudo service airflow-worker start
sudo systemctl enable airflow-worker
```

## Step 20 - Identify Worker's Hostnames \(ONLY ON MASTER\)

To be able to see all logs on the airflow webserver, you need to identify the hostnames of all workers on the master node.

```text
sudo nano /etc/hosts
```

* [x] Add the following lines and save the file.

```text
# IP and hostnames of airflow workers.

XX.XXX.XXX.XXX  HOSTNAME1
XX.XXX.XXX.XXX  HOSTNAME2
```



