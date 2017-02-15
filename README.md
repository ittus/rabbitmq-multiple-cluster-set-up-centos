# rabbitmq-multiple-cluster-set-up-centos
Guide for setting up high available rabbitmq cluster

# Set up rabbitmq


# Config hosts file


# Config firewalld
Do it on master and all slave clusters:

```
firewall-cmd --zone=public --permanent --add-port=15672/tcp
firewall-cmd --zone=public --permanent --add-port=25672/tcp
firewall-cmd --zone=public --permanent --add-port=5672/tcp
firewall-cmd --zone=public --permanent --add-port=4369/tcp
firewall-cmd --reload
```
 

# Set up cluster

# Extra
## Remove all message in queue
```
rabbitmqadmin list queues name | awk '{print $2}' | xargs -I qn rabbitmqadmin delete queue name=qn
```

## Monitor by systemctl

```
chkconfig rabbitmq-server on
```
Edit /etc/systemd/system/rabbitmq-server.service

```
[Unit]
Description=Rabbitmq service
After=network.target

[Service]
User=root
Group=root
WorkingDirectory=/home/root/
ExecStart=/usr/sbin/rabbitmq-server
Restart=always

[Install]
WantedBy=multi-user.target
```

Now we can monitor by
```
systemctl start rabbitmq-server
systemctl status rabbitmq-server
systemctl stop rabbitmq-server
```



