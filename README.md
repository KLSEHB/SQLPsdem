# SQLPsdem
Dataset and code for "SQLPsdem: A Proxy-based Mechanism towards Detecting, Locating and Preventing Second-Order SQL Injections"  

![image](https://github.com/KLSEHB/SQLPsdem/assets/142284636/a00780f9-6e01-4524-8d98-4abfb0cdfdd9)

Moreover, there are 4 videos in the directory "7.Guide videos for automation". They describe the process of automation in four stages using bwapp web application as an example: converting original web apps to tagged web apps, automated capturing requests, automated attacking by sqlmap, and automatic result aggregation.


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
   Manually：
   
   Run Fiddler and log in to DVWA. Navigate to any page, fill in the input fields, and submit the request. Then, in the Fiddler interface, locate the intercepted requests on the left side. Click on the desired request, then click on "Inspectors" on the right side. Finally, click on the "Raw" button at the bottom to view the detailed request body. Copy the 
   request body and save it in a txt file (example save path: C:\Users\liujia\Desktop\request.txt).  
   
   ![image](https://github.com/KLSEHB/SQLPsdem/assets/142284636/e923942a-8ca4-443b-9b59-283441ecd978)
   
 Automatically（Complex configuration）：
 
 Requirements：  Fiddler+ Selenium + Webdriver + demp.py
 
 (1) By default, Fiddler can only listen to HTTP requests. To capture HTTPS request packets, you need to make some settings. The setting path is: Tools->Options->HTTPS.
 
   ![image](https://github.com/KLSEHB/SQLPsdem/assets/142284636/2ce88026-9103-4857-b2b8-f3511fe9aed6)
   ![image](https://github.com/KLSEHB/SQLPsdem/assets/142284636/165f2bc2-b536-4699-bacf-b03906509d28)
   ![image](https://github.com/KLSEHB/SQLPsdem/assets/142284636/e8aef8d2-f7cf-4599-adec-6c83ce971c5d)
   ![image](https://github.com/KLSEHB/SQLPsdem/assets/142284636/cc6bb10f-b0ec-4709-85e3-42c08479e8dc)

   Then Open the Chrome browser and import the certificate in the browser. Restart browser, restart Fiddler (reset certificate)
   
（2） Add a script to Fiddler：

   1）Set the path: Rules->Customize
   
![image](https://github.com/KLSEHB/SQLPsdem/assets/142284636/4e344966-24dd-4684-afed-cc41e743769b)

Press Ctrl+F to find the function OnBeforeRequest, and add the following script to this function.

1.	if ((oSession.fullUrl.Contains("?") || oSession.HTTPMethodIs("POST"))&& oSession.fullUrl.Contains("localhost")) {  
2.	            var fso;  
3.	            var file;  
4.	            fso = new ActiveXObject("Scripting.FileSystemObject");  
5.	            var fileName ="C:\\pythonProject1\\request\\" +  oSession.url.replace(/[^\w]/g, '_') + ".txt";  
6.	            // File save path, which can be customized.  
7.	            file = fso.OpenTextFile(fileName, 8, true, true);  
8.	            file.writeLine("Request url: " + oSession.url);  
9.	            file.writeLine("Request header:" + "\n" + oSession.oRequest.headers);  
10.	            file.writeLine("Request body: " + oSession.GetRequestBodyAsString());  
11.	            file.writeLine("\n");  
12.	            file.close();  
13.	        }  

![image](https://github.com/KLSEHB/SQLPsdem/assets/142284636/7830c13c-bd4b-4feb-a343-84966d88b120)

Then find the function OnBeforeResponse, and add the following script to this function.

1.	if ((oSession.fullUrl.Contains("?") || oSession.HTTPMethodIs("POST"))&& oSession.fullUrl.Contains("localhost")) {  
2.	            oSession.utilDecodeResponse();  
3.	            //Eliminate the situation that the saved request may be garbled.  
4.	            var fso;  
5.	            var file;  
6.	            fso = new ActiveXObject("Scripting.FileSystemObject");  
7.	           // File save path, which can be customized.
8.	            var fileName ="C:\\pythonProject1\\request\\" +  oSession.url.replace(/[^\w]/g, '_') + ".txt";  
9.	            file = fso.OpenTextFile(fileName,8 ,true, true);  
10.	            file.writeLine("Response code: " + oSession.responseCode);  
11.	            file.writeLine("Response body: " + oSession.GetResponseBodyAsString());  
12.	            file.writeLine("\n");  
13.	            file.close();  
14.	        }  

![image](https://github.com/KLSEHB/SQLPsdem/assets/142284636/5891b655-1324-46c9-a248-1edc8864cfd9)

（3）Run demo.py to get the request automatically.

【Step 1】: Install selenium

First, install all the uninstalled libraries, and enter the following command line in the terminal of pycharm.

    1. pip install selenium
    
The installation of other uninstalled libraries is similar to the above code.

【Step 2】: Install the chromedriver driver.

First of all, we need to check our own version of Google browser, enter chrome://version/ in the search box, and see that my browser version is 100.0.4896.75 (official version).

![image](https://github.com/KLSEHB/SQLPsdem/assets/142284636/492b4a39-d629-47f3-8733-204455970849)

Subsequently, enter the website https://registry.npmmirror.com/binary.html?. Path=chromedriver/ Just choose the corresponding version of the driver to download.

![image](https://github.com/KLSEHB/SQLPsdem/assets/142284636/3f0b764b-2ca0-42af-8f1d-f1baf5c827a1)

Decompress after downloading, put the driver in the startup directory of Google browser, and then configure its address into the environment variable.

![image](https://github.com/KLSEHB/SQLPsdem/assets/142284636/8800c5c7-04ba-4bb4-97a3-3bb65674113e)

![image](https://github.com/KLSEHB/SQLPsdem/assets/142284636/3373115c-28c7-4525-a3b0-bfee0ab4a2cf)

【Step 3】: Execute demo.py

Take bWAPP as an example, modify the paths of input file and output file at input_file and output_file. Note: input_file must be consistent with the file saving path modified in the previous Fiddler script, and output_file is the path to save the request.

![image](https://github.com/KLSEHB/SQLPsdem/assets/142284636/cc17b9c7-ce43-460c-9380-b5a9065326de)

It should be noted that when executing demo.py, all the configuration files and agents mentioned before in this article must be opened, and so must Fiddler.



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
   
   Manually:
   Sqlmap needs to run in a Python 2 environment and requires the intermediate request from Section 2.2. Open the sqlmap folder and execute the following command in the root directory:   
   sqlmap.py -r C:\Users\liujia\Desktop\request.txt --batch -o  
   You can use this command as a template, and for each subsequent request, you only need to modify the request path without any additional modifications.   
   If the following interface appears, it indicates successful execution:  
   ![image](https://github.com/KLSEHB/SQLPsdem/assets/142284636/48372159-4445-4af9-93f6-d9d9649726ff)

   The following result indicates that no vulnerabilities were detected:
   ![image](https://github.com/KLSEHB/SQLPsdem/assets/142284636/769d86ae-017c-4983-945d-3869434e3e34)

   Automatically：
   Sqlmap needs to run in python2 environment, and it needs to use the intermediate request in step 2. We have written a batch file, which can process each request file in sequence, create a new text, and enter the code (kk.bat) in it. In this file, the path of the line 4 needs to be changed to the installation path of Sqlmap, the path of the  line 5 needs to be changed to the storage path of the intermediate request of Section 2, and the path of the line 6 needs to be changed to the storage path of the results (validation.txt).
   
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


