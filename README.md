# replication-master-master
Install MariaDB:
sudo yum install mariadb-server
sudo systemctl start mariadb
sudo systemctl enable mariadb
Secure MariaDB installation
sudo mysql_secure_installation
Configure Master-Master Replication: On both instances:

Edit MariaDB configuration file
sudo nano /etc/my.cnf.d/server.cnf
Add the following lines:
server-id = 1  # Use 2 on the other instance
log_bin = /var/log/mariadb/mariadb-bin
bind-address = 0.0.0.0
Restart MariaDB
sudo systemctl restart mariadb
Create replication user
CREATE USER 'replication_user'@'%' IDENTIFIED BY 'your_password';
GRANT REPLICATION SLAVE ON *.* TO 'replication_user'@'%';
FLUSH PRIVILEGES;
Get Master Status: Note down the current position of the binary log by running:
SHOW MASTER STATUS;
Record the values for File and Position.
Configure Replication:

On the first intstance:
CHANGE MASTER TO
   MASTER_HOST='second_instance_ip',
   MASTER_USER='replication_user',
   MASTER_PASSWORD='your_password',
   MASTER_LOG_FILE='second_master_log_file',
   MASTER_LOG_POS=second_master_log_pos;
On the second intance:
CHANGE MASTER TO
   MASTER_HOST='first_instance_ip',
   MASTER_USER='replication_user',
   MASTER_PASSWORD='your_password',
   MASTER_LOG_FILE='first_master_log_file',
   MASTER_LOG_POS=first_master_log_pos;
On both instances:
START SLAVE;
Verify Replication:

On either instances
SHOW SLAVE STATUS\G
Ensure Slave_IO_Running and Slave_SQL_Running are both Yes.
Create the load balancer user:

CREATE USER 'load_balancer_user'@'%' IDENTIFIED BY 'your_password';
GRANT ALL PRIVILEGES ON *.* TO 'load_balancer_user'@'%' IDENTIFIED BY 'your_password' WITH GRANT OPTION;
FLUSH PRIVILEGES;
