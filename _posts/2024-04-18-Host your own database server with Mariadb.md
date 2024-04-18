---
title: Host your own database server with MariaDB  
date: 2024-04-18
categories: [database]
tags: [database]
image: assets/mariadb.jpg
---

**MariaDB is an open-source relational database management system derived from MySQL, offering enhanced features and compatibility. 
It provides robust performance, security, and scalability for various applications, making it a popular choice in the database community.**

You can host your very own central database server on your network, allowing all your applications to connect to one central database server. 

The major applications I use are WordPress and Nextcloud. When deploying these applications on your server, you are required to connect to a local database on the same host.
The database you connect to doesn't have to be on the same local host server and works great where applications are deployed in a shared environment.

For this tutorial, I will be using Ubuntu as my operating system on xneelo cloud. 
The steps will apply to any Debian-based Linux distro and can be performed on a cloud instance or physical hardware.

<br>
<br>

Step 1 - System Updates 

Access your server on the command line and perform updates

```
sudo apt update -y
```

Apply the available updates

```
sudo apt dist-upgrade -y
```
<br>
<br>
Step 2 - Install Maria DB server 

```
sudo apt install mariadb-server
```

Once the installation completes, secure the database by running the below command and following the instructions

```
sudo mysql_secure_installation
```
![secure](assets/secure1.jpg)
![secure](assets/secure2.jpg)

<br>

Step 3 - Set the Bind address 

Navigate to the following directory and edit the file `50-server.cnf` 

```
cd /etc/mysql/mariadb.conf.d
```

Look for the **bind-address** section and change it to **0.0.0.0** to allow remote connections to the database.

![bind](assets/bind.jpg)

<br>

Restart MariaDB to allow the changes to take effect 
```
sudo systemctl restart mariadb.service
```

Confirm that MariaDB is running after the service restart
```
sudo systemctl status mariadb.service
```
![confirm](assets/status.jpg)

<br>

***Create scripts to easily add users and delete users, and create a database backup***

Step 1 

Navigate to your home directory and create a folder named 'db_scripts'
```
sudo mkdir db_scripts
```
Change directory into the folder
```
cd db_scripts
```

Using your text editor of choice, add the following scripts:

**Add a new database and user**
```
sudo nano add_db_user.sh
```

Add the below snippet of code to the file:
```
#!/bin/bash

# Prompt the user for database details
read -p "Enter the desired database name: " db_name
read -p "Enter the desired username: " username
read -s -p "Enter the desired password: " password
echo

# Access the MariaDB server
mysql -u root -p <<EOF
# Create the database
CREATE DATABASE $db_name;

# Create the user and set the password, allowing connections from any host
CREATE USER '$username'@'%' IDENTIFIED BY '$password';

# Grant privileges to the user on the database for any host
GRANT ALL PRIVILEGES ON $db_name.* TO '$username'@'%';

# Flush privileges
FLUSH PRIVILEGES;

EOF

# Inform the user about the success
echo "Database, username, and password created successfully."
```
Save the file and exit your text editor

<br>

**Delete an existing user**
```
sudo nano del_db_user.sh
```

Add the below snippet of code to the file:
```
#!/bin/bash

# Prompt the user for the database name to delete
read -p "Enter the name of the database you want to delete: " db_name

# Access the MariaDB server
mysql -u root -p <<EOF
# Delete the database
DROP DATABASE IF EXISTS $db_name;
EOF

# Inform the user about the deletion
echo "The database '$db_name' has been deleted, if it existed."
```

<br>

**Create a script to backup databases**
```
sudo nano create_db_dump.sh
```

Add the below snippet of code to the file:
```
#!/bin/bash

# Directory to store backup files
backup_dir="/usr/databases/mysql_dumps/"

# Create the backup directory if it doesn't exist
mkdir -p "$backup_dir"

# Get list of databases (excluding system databases)
databases=$(mysql -e "SHOW DATABASES;" | grep -Ev "(Database|information_schema|performance_schema|mysql)")

# Loop through each database and create backup silently
for db in $databases; do
    # Perform database backup without displaying individual messages
    mysqldump --single-transaction --routines --triggers "$db" > "$backup_dir$db.sql"
done

# Display completion message after all backups are done
echo "All database backups completed successfully."
```

***Test the Scripts***

***CREATE A NEW DATABASE AND USER***
```
sudo bash add_db_user.sh
```

You should see the below available options:

![add](/assets/addb.jpg)


***DELETE AN EXISTING DATABASE AND USER***
```
sudo bash del_db_user.sh
```

You should see the below output:

![add](/assets/deldb.jpg)


***CREATE A DATABASE DUMP BACKUP***:

```
sudo bash create_db_dump.sh
```

You should see the below output:

`All database backups completed successfully.`

The script is set to first create a directory for the backup files and then dump the backup files in the directory `/usr/databases/mysql_dumps`. This can be changed in the bash script as desired.


Summary:
<br>
`You have successfully configured MariaDB and added bash scripts to interact with the database.`

Instead of manually running the scripts from the 'db_scripts' directory, we can create an 
alias in the server `.bashrc` file to allow the scripts to be run from anywhere on the server

To do this, first change the permissions of the files

```
sudo chmod a+x add_db_user.sh create_db_dump.sh del_db_user.sh
```

Now EDIT the .bashrc file

```
sudo nano .bashrc
```

Add the following aliases at the end of the file 

```
#database scripts

alias add_db_user='sudo bash /home/user/scripts/add_db_user.sh'
alias create_db_dump='sudo bash /home/user/scripts/create_db_dump.sh'
alias del_db_user='sudo bash /home/user/scripts/del_db_user.sh'
```
***Remember to update the paths in the script to point to the directory where you saved the scripts***

restart the .bashrc file 
```
source ~/.bashrc
```

You will now be able to run the scripts via the aliases set in the .bashrc file from anywhere on the server. 

The next post will cover how to connect your external applications to your database server. 
