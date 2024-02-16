# SQLPsdem
Dataset and code for "SQLPsdem: A Proxy-based Mechanism towards Detecting, Locating and Preventing Second-Order SQL Injections"

###  1. Web Application Deployment
   #### 1.1 Overview
DVWA, short for Damn Vulnerable Web Application, is a web application intentionally designed with numerous vulnerabilities. It is developed using PHP and MySQL and serves as a legitimate environment for security professionals to test their tools and skills. We takes DVWA as an example, while the operational steps for the other 11 applications are similar to DVWA with minor differences.
   
   #### 1.2 XAMPP Installation 
XAMPP is a bundled package for PHP debugging environment that includes the necessary "PHP+MySQL+Apache" services.  
Download link: https://sourceforge.net/projects/xampp/files/XAMPP%20Windows/  
For the 12 web applications, you will need to download two versions of XAMPP (PHP7 and PHP5). It is recommended to download PHP7.4.18 and PHP5.4.13.  
DVWA needs to be deployed in the PHP7 environment (the other 11 applications are already categorized). Download PHP7.4.18, click "next" to proceed with the default configuration, modify the installation path, and then run xampp-control.exe.  
   ![image](https://github.com/KLSEHB/SQLPsdem/assets/142284636/8402edd1-b144-4ce4-9c7d-93adc44db1f3)

Running Apache and MySQL services, the "Ports" refer to the ports on which the services are started. You can modify the corresponding ports through the configuration settings. In the image above, Apache is associated with port 8080, while MySQL is associated with port 3306. To access the services, you can visit localhost:port in your web browser. If you see the following page, it indicates that the installation was successful.  
![image](https://github.com/KLSEHB/SQLPsdem/assets/142284636/40717b44-1d80-4490-b2d3-1498f8457ec9)

You can click on the "phpMyAdmin" link in the top right corner to access the database management interface. All the web application tested are listed in the Table as follows.  
![image](https://github.com/KLSEHB/SQLPsdem/assets/142284636/f81beade-e589-4402-932f-b91f433368c8)

   #### 1.3 Deployment Access

   
###  2.Intermediate Requests 
   #### 2.1 Fiddler Installation
   
   #### 2.2 Saving Intermediate Requests
   
###  3. Database Proxy
   #### 3.1 Tagging Dvwa (Static Analysis) 
   
   #### 3.2 Starting Database Proxy (Dynamic Analysis)
   
###  4. Attack Case Generation Tools
   #### 4.1 SQLMAP
   
   #### 4.2 DSSS
   
   #### 4.3 SSQL
   
   #### 4.4 JSQLi
   
###  5. Results Analysis
   #### 5.1 Obtaining Detection Results
   
   #### 5.2 Time Analysis
