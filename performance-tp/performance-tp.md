# PERFORMANCE - MYSQL ENTERPRISE SCALABILITY (THREAD POOLING)

## Introduction
Performance enhancements
MySQL Enterprise Scalability provides an easy to use, built-in database solution to increase performance during heavy periods of application connections.
Objective: Install and use Thread Pooling functionalities

Estimated Lab Time: -- 40 minutes

### Objectives

In this lab, you will:
* Install and test out sysBench
* Install BMK Toolkit
* Run benchmarks with increasing number of connections
* Install Thread Pool
* Run benchmarks with increasing number of connections

### Prerequisites

This lab assumes you have:
* An Oracle account
* All previous labs successfully completed

* Lab standard  
    - ![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell> the command must be executed in the Operating System shell
    - ![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql> the command must be executed in a client like MySQL, MySQL Workbench
    - ![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh> the command must be executed in MySQL shell
    

**Notes:**
- Thread Pooling has more functions than what we test in the lab. The full list of functions is here
- https://dev.mysql.com/doc/refman/8.0/en/thread-pool.html



## Task 1: Install and setup environment  

1.	Setup hardware and networking as described in Lab 1 & Lab 2

    a. Follow steps listed below.  For Lab 2 where you create the Compute, use a 12 OCPU Core with 1TB of storage.  
    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>** 
*    [Lab 1](https://ddasker.github.io/mysql-enterprise/workshops/freetier/index.html?lab=create-vcn)
*    [Lab 2](https://ddasker.github.io/mysql-enterprise/workshops/freetier/index.html?lab=create-compute)

2. SSH to Server 

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    <copy>ssh -i id_rsa opc@public_ip </copy>
    ```

3.  Make /workshop Directory

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    <copy>sudo mkdir /workshop </copy>
    ```

4.  Download workshop files 

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    <copy>cd /workshop </copy>
    ```

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    <copy> sudo wget https://objectstorage.us-ashburn-1.oraclecloud.com/p/10Aj4sllv2ek8kIQSthP31EQXY1rS_RjXjG31WAS6ttnFQiLONkIWQCX1h2ck-CC/n/idazzjlcjqzj/b/bucket-20230816-0811j-ThreadPool/o/ThreadPoolthreadpoolfiles.tar</copy>
    ```
    
5.  Extract workshop files 

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    <copy>sudo tar xvf ThreadPoolthreadpoolfiles.tar</copy>
    ```

2.	Install EPEL Repository and sysbench
    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>** 
    ```
    <copy>sudo yum -y install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm</copy>
    ```

Download files:
sudo wget https://objectstorage.us-ashburn-1.oraclecloud.com/p/10Aj4sllv2ek8kIQSthP31EQXY1rS_RjXjG31WAS6ttnFQiLONkIWQCX1h2ck-CC/n/idazzjlcjqzj/b/bucket-20230816-0811j-ThreadPool/o/ThreadPoolthreadpoolfiles.tar

3.	Install all RPMs:

    a. Run Yum 

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>** 
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
    <copy> *                soft    nofile          131072
    *                hard    nofile          131072
    *                soft    nproc           65536
    *                hard    nproc           65536</copy>    
    ```

    c. Setup OS for more open files 

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    <copy>sudo sysctl net.ipv4.tcp_max_syn_backlog=30000
    sudo sysctl net.core.netdev_max_backlog=30000
    sudo sysctl net.core.somaxconn=30000</copy>
    ```

    d. Restart MySQL

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    <copy>sudo service mysqld restart</copy>
    ```

3.	Setup the benchmarking 

    a. Create the sample database

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    
    ```
    <copy>mysql -uroot -pWelcome1! -e"CREATE DATABASE sysbench"</copy>
    ```

    b. Create the benchmark user

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    <copy>mysql -uroot -pWelcome1! -e"CREATE USER 'dim'@'%' IDENTIFIED WITH mysql_native_password BY 'Welcome1!';"</copy>
    <copy>mysql -uroot -pWelcome1! -e"GRANT ALL ON *.* TO 'dim'@'%'";</copy>
    ```

    c. Setup MySQL to handle many connections

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    <copy>sudo vi /etc/my.cnf</copy>
    ```
    ```
    <copy> max_connections=30000
    back_log=30000
    max_prepared_stmt_count=64000</copy>
    ```

    d. Determine what file to edit if using systemd for MySQL

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    <copy>sudo service mysqld status</copy>
    ```

    e. Increase number of threads possible for the mysqld daemon

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    <copy>sudo vi /usr/lib/systemd/system/mysqld.service</copy>
    ```

    e. Modify LimitNOFILE under [Service] section

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    <copy>LimitNOFILE=50000</copy>
    ```

    f. Reload Daemons

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    <copy>sudo systemctl daemon-reload</copy>
    ```

    g. Restart MySQL

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    <copy>sudo service mysqld restart</copy>
    ```

4.	Install the Thread Pool plugin

    a. **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    
    ```
    <copy>sudo nano /etc/my.cnf</copy>
    ```

    b. Add the following lines to load the plugin

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    <copy>plugin-load-add=thread_pool.so</copy>    
    ```

    c. Restart MySQL

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    <copy>sudo service mysqld restart</copy>
    ```

    d. Confirm that the plugin has been loaded
    
    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    <copy>mysql -uroot -pWelcome1! -e"SELECT PLUGIN_NAME, PLUGIN_STATUS FROM INFORMATION_SCHEMA.PLUGINS
       WHERE PLUGIN_NAME LIKE 'thread%';"</copy>
    ```

5.	Now we enable Encryption on the employees.employees table:

    a.  **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>** 
    ```
    <copy>mysql -u root -pWelcome1! -P3306 -h127.0.0.1 </copy>
    ```

6.	"Spy" on employees.employees table again:

    a. **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    <copy>sudo strings "/var/lib/mysql/employees/employees.ibd" | head -n50</copy>
    ```


7.	Administrative commands

    a. Get details on encrypted key file:
    **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 
    ```
    <copy>SHOW VARIABLES LIKE 'keyring_encrypted_file_data'\G</copy>
    ```


## Task 2: Install and setup Benchmarks  
1.	Install EPEL Repository and sysbench

## Learn More

* [Thread Pool Plugins](https://dev.mysql.com/doc/refman/8.0/en/thread-pool.html)
* [sysBench code](https://github.com/akopytov/sysbench/tree/master)
* [sysBench useage](https://www.mortensi.com/2021/06/how-to-install-and-use-sysbench/)

## Acknowledgements
* **Author** - Dale Dasker, MySQL Solution Engineering
* **Last Updated By/Date** - Dale Dasker, August 2023