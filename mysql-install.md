# MySQL installation
Default values for DB users
- user: `root` password: `dam36njE2ZY4HvfR2Vf.YgjB*V9-DK`
- user: `meadmin` password: `rte.uY694a8bNrW-MQ_mBz`
- user: `replica_user` password: `rte.uY694a8bNrW-MQ_mBz`
## Installation on RHEL/CENTOS/ROCKY LINUX 8
Installation instructions in sudo mode `su -`
```sh
dnf -y install @mysql
systemctl enable mysqld.service
sudo systemctl start mysqld.service
systemctl status mysqld.service
```
## MySQL configuration
```sh
mysql_secure_installation
```
config steps:
```text
Press y|Y for Yes, any other key for No: y
Please enter 0 = LOW, 1 = MEDIUM and 2 = STRONG: 2
New password: [ENTER STRONG PASSWORD HERE]
Re-enter new password: RE ENTER PASSWORD HERE
Do you wish to continue with the password provided?(Press y|Y for Yes, any other key for No) : y
Remove anonymous users? (Press y|Y for Yes, any other key for No) : y
Disallow root login remotely? (Press y|Y for Yes, any other key for No) : y
Remove test database and access to it? (Press y|Y for Yes, any other key for No) : y
Reload privilege tables now? (Press y|Y for Yes, any other key for No) : y
```
## Firewall setup
```sh
firewall-cmd --zone=public --permanent --add-port 3306/tcp
firewall-cmd --reload
```