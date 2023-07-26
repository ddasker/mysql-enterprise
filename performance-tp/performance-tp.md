# SECURITY - MYSQL ENTERPRISE TRANSPARENT DATA ENCRYPTION

## NOTES

Install all RPMs:
    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>** 
    ```
    <copy>sudo yum -y install *.rpm</copy>
    ```

As root, change the ulimit to 100000
    sudo ulimit -n 100000

Setup Linux for more connections:
    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>** 
    ```
    <copy>sudo vi /etc/security/limits.conf</copy>
    ```

    ```
    <copy>
    *                soft    nofile          131072
    *                hard    nofile          131072
    *                soft    nproc           65536
    *                hard    nproc           65536</copy>
    ```

Setup Linux for more connections:
    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>** 
    ```
    <copy>
    sudo sysctl net.ipv4.tcp_max_syn_backlog=30000
    sudo sysctl net.core.netdev_max_backlog=30000
    sudo sysctl net.core.somaxconn=30000</copy>
    ```

Adding benchmarking user
    mysql> CREATE USER 'dim'@'%' IDENTIFIED WITH mysql_native_password BY "Welcome1!";
    mysql> GRANT ALL ON *.* TO 'dim'@'%';

Setup sysBench database:
    mysql> CREATE DATABASE sysbench;

Configure MySQL for more connections:
    [mysqld]
    max_connections=30000
    back_log=30000
    max_prepared_stmt_count=64000

Install Plugins:
    [mysqld]
     plugin-load-add=thread_pool.so


## Introduction
3c) MySQL Enterprise Thread Pooling
Objective: Thread Pooling in action…

This lab will walk you through benchmarking Thread Pooling

Estimated Lab Time: 40 minutes

### Objectives

In this lab, you will:
* Install and setup benchmarking tools

### Prerequisites (Optional)

This lab assumes you have:
* An Oracle account
* All previous labs successfully completed

* Lab standard  
    - ![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell> the command must be executed in the Operating System shell
    - ![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql> the command must be executed in a client like MySQL, MySQL Workbench
    - ![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh> the command must be executed in MySQL shell
    

**Notes:**
- [InnoDB Data At Rest](https://dev.mysql.com/doc/refman/8.0/en/innodb-data-encryption.html)


## Task 1: Install and setup Benchmarks  
1.	Install EPEL Repository and sysbench
    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>** 
    ```
    <copy>sudo yum -y install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm</copy>
    ```

2.	Install all RPMs:
    a. **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>** 
    
    ```
    <copy>sudo yum -y install *.rpm</copy>
    ```

    b. Edit the /etc/security/limits.conf

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    <copy>sudo nano /etc/security/limits.conf</copy>
    ```

    b. Add the following lines to load the plugin and set the encrypted key file

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    <copy *                soft    nofile          131072
    *                hard    nofile          131072
    *                soft    nproc           65536
    *                hard    nproc           65536</copy>    
    ```

    c. Restart MySQL

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    <copy>sudo service mysqld restart</copy>
    ```

3.	"Spy" on employees.employees table

    a. **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    <copy>sudo strings "/var/lib/mysql/employees/employees.ibd" | head -n50</copy>
    ```


4.	Now we enable Encryption on the employees.employees table:

    a.  **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>** 
    ```
    <copy>mysql -u root -pWelcome1! -P3306 -h127.0.0.1 </copy>
    ```

    b. **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>**
    ```
    <copy>USE employees;</copy>
    ```

    c. **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 
    ```
    <copy>ALTER TABLE employees ENCRYPTION = 'Y';</copy>
    ```


5.	"Spy" on employees.employees table again:

    a. **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    <copy>sudo strings "/var/lib/mysql/employees/employees.ibd" | head -n50</copy>
    ```


6.	Administrative commands

    a. Get details on encrypted key file:
    **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 
    ```
    <copy>SHOW VARIABLES LIKE 'keyring_encrypted_file_data'\G</copy>
    ```

    b. Set default for all tables to be encrypted when creating them:
    **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 
    ```
    <copy>SET GLOBAL default_table_encryption=ON;</copy>
    ```

    c. Peek on the mysql System Tables:
    **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 
    ```
    <copy>sudo strings "/var/lib/mysql/mysql.ibd" | head -n70</copy>
    ```

    d. Encrypt the mysql System Tables:
    **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 
    ```
    <copy>ALTER TABLESPACE mysql ENCRYPTION = 'Y';</copy>
    ```

    e. Validate encryption of the mysql System Tables:
    **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 
    ```
    <copy>sudo strings "/var/lib/mysql/mysql.ibd" | head -n70</copy>
    ```

    f. Show all the encrypted tables:
    **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 
    ```
    <copy>SELECT SPACE, NAME, SPACE_TYPE, ENCRYPTION FROM INFORMATION_SCHEMA.INNODB_TABLESPACES WHERE ENCRYPTION='Y'\G</copy>
    ```


## Learn More

* [Keyring Plugins](https://dev.mysql.com/doc/refman/8.0/en/keyring.html)
* [InnoDB Data At Rest](https://dev.mysql.com/doc/refman/8.0/en/innodb-data-encryption.html)

## Acknowledgements
* **Author** - Dale Dasker, MySQL Solution Engineering
* **Last Updated By/Date** - <Dale Dasker, January 2023
