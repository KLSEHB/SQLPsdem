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
![image](https://github.com/KLSEHB/SQLPsdem/assets/142284636/929e5624-09dc-44bb-9942-c1230e8a5c5d)

   #### 1.3 Deployment Access
Move the "dvwa1" folder (source code of DVWA) to the "htdocs" directory under the XAMPP root, and then modify the configuration file "config.php" within the "dvwa1" folder. Each web application typically has a configuration file, and with the exception of "petshop," which requires configuring the root directory, all other applications only need modifications for the database access address, database name, login user, and login password.  
![image](https://github.com/KLSEHB/SQLPsdem/assets/142284636/54562148-e7fe-4ab8-ab2d-f9e8f628f4ef)

Please note that the configuration you are referring to is the port for the database startup. Port 3306 is used for direct database connection mode, where requests are sent directly to the database. On the other hand, port 4040 is used for the database proxy forwarding mode, where requests are sent to the proxy, which then forwards them to the database.  
Under the condition of not using a proxy, the port should be set to 3306. The access address would be: localhost:port/dvwa1.  
For the "Petshop" application, unlike the other 11 applications, the configuration file requires specifying the root directory. The root directory can be set as the access path for Petshop. The "base_app" and "dev_data" settings do not need to be changed. Only the remaining database configuration items should be modified to correspond to the respective addresses.  
![image](https://github.com/KLSEHB/SQLPsdem/assets/142284636/905bfeab-8385-4f69-81bc-ee2d5b6c4507)

###  2.Intermediate Requests 
   #### 2.1 Fiddler Installation
   To download and install Fiddler, please follow these steps:  
    Open your web browser and visit the official website for Fiddler at https://www.telerik.com/download/fiddler.  
    On the website, you should see a "Download Free" button. Click on it to start the download.  
    Once the download is complete, locate the downloaded setup file (usually in your Downloads folder) and double-click on it to run the installer.  
    
   #### 2.2 Saving Intermediate Requests
   Run Fiddler and log in to DVWA. Navigate to any page, fill in the input fields, and submit the request. Then, in the Fiddler interface, locate the intercepted requests on the left side. Click on the desired request, then click on "Inspectors" on the right side. Finally, click on the "Raw" button at the bottom to view the detailed request body. Copy the request body and save it in a txt file (example save path: C:\Users\liujia\Desktop\request.txt).  
   ![image](https://github.com/KLSEHB/SQLPsdem/assets/142284636/e923942a-8ca4-443b-9b59-283441ecd978)

###  3. Database Proxy
   #### 3.1 Tagging Dvwa (Static Analysis) 
   To modify the basedir variable in the main function of soi_6.py to the directory path of DVWA，and run it directly.  
   ![image](https://github.com/KLSEHB/SQLPsdem/assets/142284636/563dbfa2-387f-4793-aa6a-e2b68e8a673a)

   After that, we get the new web application with the reconstructed sql queries and deployed it in XAMPP, please refer to 1.3. Please note that you need modify the configuration file of DVWA to change the port to 4040 and start the database proxy forwarding mode.
   
   #### 3.2 Starting Database Proxy (Dynamic Analysis)
   Open the detect-injection-6.lua file located in \share\doc\mysql-proxy within the mysql-proxy-0.8.5 folder. Modify the following configuration variables:  
    (1)saveResFileName: Path to save the results.  
    (2)toolName: Tool used to generate total test cases.  
    (3)app_name: Web application being tested.  
   No modifications are required for the remaining configuration items.  
   ![image](https://github.com/KLSEHB/SQLPsdem/assets/142284636/943397bd-777e-4004-92d8-a7c4cf14b531)

Click on the run.bat file located in the root directory of the mysql-proxy-0.8.5 folder to start the database proxy. Navigate to any page in DVWA. If you see similar results as below, it indicates that static analysis and proxy startup were successful.  
![image](https://github.com/KLSEHB/SQLPsdem/assets/142284636/cf6c8d9a-4bb3-4ee5-a8c0-a5862e9f5bd3)

###  4. Attack Case Generation Tools
   #### 4.1 SQLMAP
   Sqlmap needs to run in a Python 2 environment and requires the intermediate request from step 2. Open the sqlmap folder and execute the following command in the root directory:   
   ···sqlmap.py -r C:\Users\liujia\Desktop\request.txt --batch -o···  
You can use this command as a template, and for each subsequent request, you only need to modify the request path without any additional modifications.   
If the following interface appears, it indicates successful execution:  
![image](https://github.com/KLSEHB/SQLPsdem/assets/142284636/48372159-4445-4af9-93f6-d9d9649726ff)

The following result indicates that no vulnerabilities were detected:
![image](https://github.com/KLSEHB/SQLPsdem/assets/142284636/769d86ae-017c-4983-945d-3869434e3e34)

   #### 4.2 DSSS
   
   #### 4.3 SSQL
   
   #### 4.4 JSQLi
   
###  5. Results Analysis
   #### 5.1 Obtaining Detection Results
   
   #### 5.2 Time Analysis
