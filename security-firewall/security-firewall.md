# SECURITY - ENTERPRISE FIREWALL

## Introduction
MySQL Enterprise Firewall guards against cyber security threats by providing real-time protection against database specific attacks. Any application that has user-supplied input, such as login and personal information fields is at risk. Database attacks don't just come from applications. Data breaches can come from many sources including SQL virus attacks or from employee misuse. Successful attacks can quickly steal millions of customer records containing personal information, credit card, financial, healthcare or other valuable data.
Objective: Install and use data masking functionalities

Estimated Lab Time: -- 15 minutes

### Objectives

In this lab, you will:
* Install MySQL Enterprise Firewall
* Create a couple of MySQL Enterprise Firewall rules
* Test MySQL Enterprise Firewall

### Prerequisites

This lab assumes you have:
* An Oracle account
* All previous labs successfully completed

* Lab standard  
    - ![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell> the command must be executed in the Operating System shell
    - ![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql> the command must be executed in a client like MySQL, MySQL Workbench
    - ![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh> the command must be executed in MySQL shell
    

**Notes:**
- The detailed documentation for MySQL Enterprise Firewall is located here:
- https://dev.mysql.com/doc/refman/8.0/en/firewall.html

## Task 1: Install firewall plugin

1. To install the data masking plugin, execute with statements 

    a.  **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>** 
    ```
    <copy>mysql -uroot -pWelcome1! -e"source /usr/share/mysql-8.0/linux_install_firewall.sql"</copy>
    ```

    b.  **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>** 
    ```
    <copy>mysql -uroot -pWelcome1!</copy>
    ```

    c. **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 
    ```
    <copy>SHOW GLOBAL VARIABLES LIKE '%firewall%';</copy>
    ``` 

    d. **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 
    ```
    <copy>SHOW GLOBAL STATUS LIKE '%firewall%';</copy>
    ``` 

## Task 2: Setup Firewall User and Rules

1. Create user (fw_user) to run Firewall rules and inspect Information Schema tables: 

    a. **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>**
    ```
    <copy>CREATE USER 'fw_user'@'localhost' IDENTIFIED BY 'FWuser1!';</copy>
    ```

    b. **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 
    ```
    <copy>GRANT ALL ON *.* TO 'fw_user'@'localhost';</copy>
    ```

    c. **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 
    ```
    <copy>SELECT MODE FROM INFORMATION_SCHEMA.MYSQL_FIREWALL_USERS WHERE USERHOST = 'fw_user@localhost';</copy>
    ```

    d. **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 
    ```
    <copy>SELECT RULE FROM INFORMATION_SCHEMA.MYSQL_FIREWALL_WHITELIST WHERE USERHOST = 'fw_user@localhost';</copy>
    ```

2. Turn on Recording of SQL commands and then test Firewall

    a. **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 
    ```
    <copy>CALL mysql.sp_set_firewall_mode('fw_user@localhost', 'RECORDING');;</copy>
    ```

    b. **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 
    ```
    <copy>SELECT MODE FROM INFORMATION_SCHEMA.MYSQL_FIREWALL_USERS WHERE USERHOST = 'fw_user'@'localhost';</copy>
    ```

    c. **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 
    ```
    <copy>SELECT RULE FROM INFORMATION_SCHEMA.MYSQL_FIREWALL_WHITELIST WHERE USERHOST = 'fw_user'@'localhost';</copy>
    ```

## Task 3: Discussion and use  Masking functions and random generators

1. Discuss differences between  mask&#95;inner  and  mask&#95;outer 

    **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 
    ```
    <copy>SELECT mask_inner(NAME, 1,1, '&') FROM world.city limit 1;</copy>
    ```
2. Use data masking random generators to these statements several times

    a. **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>**  
    ```
    <copy>SELECT gen_range(1, 200);</copy>
    ```
    b. **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 
    ```
    <copy>SELECT gen_rnd_us_phone();</copy>
    ```
    c. **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 
    ```
    <copy>SELECT gen_rnd_email();</copy>
    ```

## Learn More

*(optional - include links to docs, white papers, blogs, etc)*

* [URL text 1](http://docs.oracle.com)
* [URL text 2](http://docs.oracle.com)

## Acknowledgements
* **Author** - Dale Dasker, MySQL Engineering
* **Last Updated By/Date** - <Dale Dasker, January 2023
