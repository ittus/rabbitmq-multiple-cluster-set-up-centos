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


