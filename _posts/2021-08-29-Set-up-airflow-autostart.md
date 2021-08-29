
Airflow, at the time being, doesn't set up the webserver/shceduler starting automatically as services on Linux.

In this post, I will show how to set it up using systemctrl.

As a root user:

# create pid folder

```{sh}
mkdir /run/airflow
chown -R airflow:airflow /run/airflow
```

# vim /lib/systemd/system/airflow-webserver.service

```
[Unit]
Description=Airflow webserver daemon
After=syslog.target network.target
[Service]
PIDFile=/run/airflow/webserver.pid
User=airflow
Group=airflow
Type=forking
ExecStart=/bin/bash -c 'export AIRFLOW_HOME=/etc/airflow ; /usr/bin/airflow webserver -D --pid /run/airflow/webserver.pid'
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s TERM $MAINPID
RuntimeDirectory=airflow
WorkingDirectory=/home/airflow
Restart=on-failure
RestartSec=42s
PrivateTmp=true
[Install]
WantedBy=multi-user.target
```

#  vim /lib/systemd/system/airflow-scheduler.service

```{config}
[Unit]
Description=Airflow scheduler daemon
After=syslog.target network.target
[Service]
PIDFile=/run/airflow/scheduler.pid
User=airflow
Group=airflow
Type=forking
ExecStart=/bin/bash -c 'export AIRFLOW_HOME=/etc/airflow ; /usr/bin/airflow scheduler -D --pid /run/airflow/scheduler.pid'
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s TERM $MAINPID
RuntimeDirectory=airflow
WorkingDirectory=/home/airflow
Restart=on-failure
RestartSec=42s
PrivateTmp=true
[Install]
WantedBy=multi-user.target
```

# Enable services
 
 ```{config}
 systemctl enable airflow-webserver.service
 systemctl enable airflow-scheduler.service
```

# Start services
 
 ```{config}
 systemctl start airflow-webserver
 systemctl start airflow-scheduler
 ```
 
 
