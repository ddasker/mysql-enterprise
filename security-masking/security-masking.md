# SECURITY - DATA MASKING

## Introduction
Data Masking and de-identification
MySQL Enterprise Masking and De-identification provides an easy to use, built-in database solution to help organizations protect sensitive data from unauthorized uses by hiding and replacing real values with substitutes.
Objective: Install and use data masking functionalities

Estimated Lab Time: -- 12 minutes

### Objectives

In this lab, you will:
* Create sample data with random generation utilites which are part of Enterprise Masking
* Test Masking of Sensitive Data
* Create a View and user which only sees masked data

### Prerequisites

This lab assumes you have:
* An Oracle account
* All previous labs successfully completed

* Lab standard  
    - ![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell> the command must be executed in the Operating System shell
    - ![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql> the command must be executed in a client like MySQL, MySQL Workbench
    - ![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh> the command must be executed in MySQL shell
    

**Notes:**
- Data masking has more functions than what we test in the lab. The full list of functions is here
- https://dev.mysql.com/doc/refman/8.0/en/data-masking-usage.html 

## Task 1: Install masking plugin

1. To install the data masking plugin, execute with statements 

    a.  **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>** 
    ```
    <copy>mysql -uroot -p -h 127.0.0.1 -P 3306</copy>
    ```
    b. **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 
    ```
    <copy>INSTALL PLUGIN data_masking SONAME 'data_masking.so';</copy>
    ```
    c. **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 
    ```
    <copy>SHOW PLUGINS;</copy>
    ```
2. Look for data_masking and check the status? Is it active?

## Task 2: Use masking functions

1. Install masking functions

    a. **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>**
    ```
    <copy>CREATE FUNCTION gen_range RETURNS INTEGER SONAME 'data_masking.so';
    CREATE FUNCTION gen_rnd_email RETURNS STRING SONAME 'data_masking.so';
    CREATE FUNCTION gen_rnd_us_phone RETURNS STRING SONAME 'data_masking.so';
    CREATE FUNCTION gen_rnd_ssn RETURNS STRING SONAME 'data_masking.so';
    CREATE FUNCTION mask_inner RETURNS STRING SONAME 'data_masking.so';
    CREATE FUNCTION mask_outer RETURNS STRING SONAME 'data_masking.so';
    CREATE FUNCTION mask_ssn RETURNS STRING SONAME 'data_masking.so';
    </copy>
    ```

2. Use data masking functions

    a. **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 
    ```
    <copy>SELECT mask_inner(last_name, 2,1) FROM employees.employees limit 10;</copy>
    ```
    b. **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 
    ```
    <copy>SELECT mask_outer(last_name, 2,1) FROM employees.employees limit 10;</copy>
    ```

## Task 3: Discussion and use  Masking functions and random generators

1. Create Table to generate and add masking data

    **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 
    ```
    <copy>CREATE TABLE employees_mask LIKE employees;</copy>
    ```

2. Add data to newly created table

    **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>**  
    ```
    <copy>INSERT INTO employees_mask SELECT * FROM employees;</copy>
    ```

3. Create new column for SSN's

    **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>**  
    ```
    <copy>ALTER TABLE employees_mask ADD COLUMN ssn varchar(11);</copy>
    ```

4. Create new column for emails's

    **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>**  
    ```
    <copy>ALTER TABLE employees_mask ADD COLUMN email varchar(40);</copy>
    ```

5. Use Functions to generate sample SSN data

    **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>**  
    ```
    <copy>UPDATE employees_mask SET ssn = gen_rnd_ssn() WHERE 1;</copy>
    ```

6. Use Functions to generate sample Email data

    **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>**  
    ```
    <copy>UPDATE employees_mask SET email = gen_rnd_email() WHERE 1;</copy>
    ```

7. Let's look at the data that we just created

    **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>**  
    ```
    <copy>SELECT * FROM employees_mask LIMIT 5;</copy>
    ```

8. Let's mask the SSN

    **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>**  
    ```
    <copy>SELECT emp_no,first_name,last_name,mask_ssn(CONVERT(ssn USING latin1)) AS ssn FROM employees_mask LIMIT 5;</copy>
    ```

9. Let's create a view which only shows the masked data

    **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>**  
    ```
    <copy>CREATE VIEW masked_customer AS SELECT emp_no,first_name,last_name,mask_ssn(CONVERT(ssn USING latin1)) AS ssn FROM employees_mask;</copy>
    ```

10. Let's create a user who only has access to the view with the masked data

    **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>**  
    ```
    <copy>CREATE USER 'accounting'@'%' IDENTIFIED BY 'Pa33word!';</copy>
    ```

    **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>**  
    ```
    <copy>GRANT SELECT ON employees.masked_customer TO 'accounting'@'%';</copy>
    ```

11. Log in with new user account and run queries

    **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>**  
    ```
    <copy>quit;</copy>
    ```

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>** 
    ```
    <copy>mysql -uaccounting -pPa33word! -h 127.0.0.1 -P 3306</copy>
    ```

    **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>**  
    ```
    <copy>SELECT * FROM employees.masked_customer LIMIT 5;</copy>
    ```

12. Try accessing table that is not masked

    **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>**  
    ```
    <copy>SELECT * FROM employees.employees_mask LIMIT 5;</copy>
    ```


## Task 4: *** OPTIONAL *** Discussion and use  Masking functions and random generators

1. Discuss differences between  mask&#95;inner  and  mask&#95;outer 

    **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 
    ```
    <copy>SELECT mask_inner(last_name, 1,1, '&') FROM employees.employees limit 1;</copy>
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

* [Enterprise Data Masking Documentation](https://dev.mysql.com/doc/refman/8.0/en/data-masking.html)
* [Whitepaper: A MySQL Guide to PCI Compliance](https://www.mysql.com/why-mysql/white-papers/mysql-pci-data-security-compliance/)
* [Whitepaper: MySQL Enterprise and the GDPR](https://www.mysql.com/why-mysql/white-papers/mysql-enterprise-edition-gdpr/)
* [Whitepaper: MySQL Secure Deployment Guide](https://dev.mysql.com/doc/mysql-secure-deployment-guide/8.0/en/)

## Acknowledgements
* **Author** - Perside Foster, MySQL Engineering
* **Contributors** -  Dale Dasker, MySQL Engineering
* **Last Updated By/Date** - <Dale Dasker, January 2023
