# SQLPsdem
Dataset and code for "SQLPsdem: A Proxy-based Mechanism towards Detecting, Locating and Preventing Second-Order SQL Injections"  

![image](https://github.com/KLSEHB/SQLPsdem/assets/142284636/a00780f9-6e01-4524-8d98-4abfb0cdfdd9)

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
   Run Fiddler and log in to DVWA. Navigate to any page, fill in the input fields, and submit the request. Then, in the Fiddler interface, locate the intercepted requests on the left side. Click on the desired request, then click on "Inspectors" on the right side. Finally, click on the "Raw" button at the bottom to view the detailed request body. Copy the 
   request body and save it in a txt file (example save path: C:\Users\liujia\Desktop\request.txt).  
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
   Sqlmap needs to run in a Python 2 environment and requires the intermediate request from Section 2.2. Open the sqlmap folder and execute the following command in the root directory:   
   sqlmap.py -r C:\Users\liujia\Desktop\request.txt --batch -o  
   You can use this command as a template, and for each subsequent request, you only need to modify the request path without any additional modifications.   
   If the following interface appears, it indicates successful execution:  
   ![image](https://github.com/KLSEHB/SQLPsdem/assets/142284636/48372159-4445-4af9-93f6-d9d9649726ff)

   The following result indicates that no vulnerabilities were detected:
   ![image](https://github.com/KLSEHB/SQLPsdem/assets/142284636/769d86ae-017c-4983-945d-3869434e3e34)

   #### 4.2 DSSS
   DSSS needs to be run in a Python 3 environment. Open the CMD window in the root directory of the DSSS folder. Enter the command:  
   python3 dsss.py --url="url" --data="data"  
   The readme.md file in the root directory provides detailed usage instructions for other parameters, which are not necessary for this experiment.

   For example, using the request from Section 2.2, enter the following command:  
   python3 dsss.py --url="http://localhost:8080/dvwa1/vulnerabilities/sqli/?id=1&Submit=Submit"

   #### 4.3 SSQL
   SSQL has developed a graphical user interface that is easy to use. Simply copy all the requests from Section 2.2 into the input box of the GUI, enter the domain name and target port, and click "Identify Injection". Once the attack is complete, the result information will be displayed.  
   ![image](https://github.com/KLSEHB/SQLPsdem/assets/142284636/1327a394-de46-4277-99f5-d53937980b37)

   #### 4.4 JSQLi
   Double-click to open the graphical interface of JSQLi. The usage method is relatively simple: you need to fill in the GET and POST input boxes. The GET field requires the request path, while the POST field includes parameters carried in a POST request. If the request method is POST, then it is necessary to fill in the parameters. After filling in the required information, click on the arrow inside the red box on the right to initiate the injection attack. Once the attack is completed, feedback with the result information will be provided.  
   ![image](https://github.com/KLSEHB/SQLPsdem/assets/142284636/efd39321-e6b6-4862-9386-89212e010715)

###  5. Results Analysis
   #### 5.1 Obtaining Detection Results
   In Section 3, we have configured a path to save the results, and at that path, you can find a validation.txt file. By running extractEvaluationIndex.py in Pycharm to analyze this file, you can obtain the detection results. In detail, you can modify the file path in line 20 of extractEvaluationIndex.py, and then run it directly.  
![image](https://github.com/KLSEHB/SQLPsdem/assets/142284636/55b68cdc-3466-423f-9c8f-81af1b1b638f)

   #### 5.2 Time Analysis
Taking DVWA as an example, to analyze time, you need to deploy another DVWA application which is the original version without static analysis.  
Remove the comments from the highlighted part in the "read_query_result" function in "detect-injection-6.lua". These three lines of code are used to save the time result, and the path information is where the result is saved.  
Select any request and add "--technique "B" "E" "I" "S" "Q" "U" to the sqlmap command to execute the original version request and static analysis version request, and obtain two time results for comparison and analysis.  
Sqlmap: (Turn off Time-based blind, parameter command setting:  
  sqlmap.py -r C: \ Users \ liujia \ Desktop \ request.txt --batch -o --technique B E I S Q U)  
Webchess1 (original version), total time/total number of cases NO  
Webchess4, enable proxy, total time/total number of cases YES  
The core function is "read_query_result", a function used to count time.  
![image](https://github.com/KLSEHB/SQLPsdem/assets/142284636/7f44e5b8-7b8e-49f7-9ad0-5851fd0e66f7)


