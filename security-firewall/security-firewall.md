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

1. To install the firewall component, execute with statements 

    a.  **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>** 
    ```
    <copy>mysql -uroot -pWelcome1! -D mysql -e"source /usr/share/mysql-8.3/linux_install_firewall.sql"</copy>
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

    e. **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 
    ```
    <copy>SET PERSIST mysql_firewall_mode = ON;</copy>
    ``` 

## Task 2: Setup Firewall User and Rules

1. Create user (fw_user) to run Firewall rules and inspect Information Schema tables: 

    a. **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>**
    ```
    <copy>CREATE USER 'member1'@'localhost' IDENTIFIED BY 'Welcome1!';</copy>
    ```

    b. **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 
    ```
    <copy>GRANT ALL ON employees.* TO 'member1'@'localhost';</copy>
    ```

    c. **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 
    ```
    <copy>SELECT MODE FROM INFORMATION_SCHEMA.MYSQL_FIREWALL_USERS WHERE USERHOST = 'member1@localhost';</copy>
    ```

    d. **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 
    ```
    <copy>SELECT RULE FROM INFORMATION_SCHEMA.MYSQL_FIREWALL_WHITELIST WHERE USERHOST = 'member1@localhost';</copy>
    ```

2. Create Group Profile and turn on Recording of SQL commands and then test Firewall

    a. **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 
    ```
    <copy>CALL mysql.sp_set_firewall_groupmode('fwgrp', 'RECORDING');</copy>
    ```

    b. **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 
    ```
    <copy>CALL mysql.sp_set_firewall_enlist('fwgrp', 'member1@localhost');</copy>
    ```

    c. **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 
    ```
    <copy>SELECT MODE FROM INFORMATION_SCHEMA.MYSQL_FIREWALL_USERS WHERE USERHOST = 'member1@localhost';</copy>
    ```

    d. **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 
    ```
    <copy>SELECT RULE FROM INFORMATION_SCHEMA.MYSQL_FIREWALL_WHITELIST WHERE USERHOST = 'member1@localhost';</copy>
    ```

## Task 3: Run queries to test Firewall characteristics.

1. Login on a separate terminal as member1.

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>** 
    ```
    <copy>mysql -umember1 -pWelcome1!</copy>
    ```

2. Run some sample queries that are acceptable

    a. **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>**  
    ```
    <copy>USE employees;SELECT emp_no, title, from_date, to_date FROM titles WHERE emp_no = 10001; </copy>
    ```

    b. **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 
    ```
    <copy>UPDATE titles SET to_date = CURDATE() WHERE emp_no = 10001;</copy>
    ```

    c. **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 
    ```
    <copy>SELECT emp_no, first_name, last_name, birth_date FROM employees ORDER BY birth_date LIMIT 10;</copy>
    ```

## Task 4: Inspect MySQL Firewall 

1. Login on a separate terminal as root.

    a. **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>** 
    ```
    <copy>mysql -uroot -pWelcome1!</copy>
    ```

    b. **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 
    ```
    <copy>SELECT MODE FROM performance_schema.firewall_groups WHERE NAME = 'fwgrp';</copy>
    ```

    c. **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 
    ```
    <copy>SELECT * FROM performance_schema.firewall_membership WHERE GROUP_ID = 'fwgrp' ORDER BY MEMBER_ID;</copy>
    ```

    d. **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 
    ```
    <copy>SELECT MODE FROM performance_schema.firewall_groups WHERE NAME = 'fwgrp';</copy>
    ```

    e. **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 
    ```
    <copy>SELECT MODE FROM performance_schema.firewall_groups WHERE NAME = 'fwgrp';</copy>
    ```

    f. **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 
    ```
    <copy>SELECT MODE FROM performance_schema.firewall_groups WHERE NAME = 'fwgrp';</copy>
    ```

    g. **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 
    ```
    <copy>SELECT MODE FROM performance_schema.firewall_groups WHERE NAME = 'fwgrp';</copy>
    ```

## Optional: 


## Learn More

*(optional - include links to docs, white papers, blogs, etc)*

* [Firewall Docs](https://dev.mysql.com/doc/refman/8.3/en/firewall.html)
* [Enterprise Firewall with Drupal](https://dev.mysql.com/blog-archive/group-profiles-in-mysql-enterprise-firewall/)

## Acknowledgements
* **Author** - Dale Dasker, MySQL Engineering
* **Last Updated By/Date** - <Dale Dasker, February 2024
