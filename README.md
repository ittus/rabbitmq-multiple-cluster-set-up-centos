# rabbitmq-multiple-cluster-set-up-centos
Guide for setting up high available rabbitmq cluster

# Set up rabbitmq

Set up rabbitmq on both master and slave1 server
```
yum -y update
wget http://www.rabbitmq.com/releases/rabbitmq-server/v3.2.2/rabbitmq-server-3.2.2-1.noarch.rpm
rpm --import http://www.rabbitmq.com/rabbitmq-signing-key-public.asc
yum install rabbitmq-server-3.2.2-1.noarch.rpm
rabbitmq-plugins enable rabbitmq_management
```

## Start rabbitmq
```
rabbitmq-server -detached
rabbitmqctl status
```

Run rabbitmqctl cluster_status and you will receive:
```
Cluster status of node rabbit@localhost ...
[{nodes,[{disc,[rabbit@master]}]},
 {running_nodes,[rabbit@master]},
 {cluster_name,<<"rabbit@master">>},
 {partitions,[]},
 {alarms,[{rabbit@master,[]}]}]
 ```
Note `rabbit@master` name, need to use it in below section.

# Config hosts file
Edit /etc/hosts on both master and slave1 server:

```
10.0.0.1 master_name master_name
10.0.0.2 slave1_name slave1_name
```

Note that master_name and slave1_name is hostname (Can run `hostname -s` to check)

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
## Sync cookie
On master:
```
[master]# cat /var/lib/rabbitmq/.erlang.cookie 
XNZYNUHKGUCVDTGZXJKX
```

Copy `DQRRLCTUGODCRFNPIABC` value to slave1:
```
[slave1]# echo -n "DQRRLCTUGODCRFNPIABC" > /var/lib/rabbitmq/.erlang.cookie
```

## Config cluster
On master:

```
[master]# rabbitmqctl stop_app
[master]# rabbitmqctl reset
[master]# rabbitmqctl start_app
```
On slave1:
```
[slave1]# rabbitmqctl stop_app
[slave1]# rabbitmqctl reset
[slave1]# rabbitmqctl join_cluster rabbit@master
[slave1]# rabbitmqctl start_app
[slave1]# rabbitmqctl cluster_status
Cluster status of node 'rabbit@slave1' ...
[{nodes,[{disc,['rabbit@master',
                'rabbit@slave1']}]},
 {running_nodes,['rabbit@master',
                 'rabbit@slave1']},
 {cluster_name,<<"rabbit@master">>},
 {partitions,[]}]
...done.

```


## Cluster policy
```
rabbitmqctl set_policy ha-all "" '{"ha-mode":"all","ha-sync-mode":"automatic"}'
```

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
## Limit number of message in queue 
```
rabbitmqctl set_policy limit_celeryev_queues "^celeryev\." '{"max-length":10}' --apply-to queues
```
(Limit queue with start with `celeryev` maximum 10 messages)


## Set up user name and password
```
rabbitmqctl add_user test test
rabbitmqctl set_user_tags test administrator
rabbitmqctl set_permissions -p / test ".*" ".*" ".*"
```
