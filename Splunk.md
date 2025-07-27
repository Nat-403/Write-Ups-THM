###  A Big corporate organization Wayne Enterprises  has recently faced a cyber-attack where the attackers broke into their network, found their way to their web server, and have successfully defaced their website http://www.imreallynotbatman.com . Their website is now showing the trademark of the attackers with the message YOUR SITE HAS BEEN DEFACED.
```
Search
Data Summary

```
### Reconnaissance Phase

Reconnaissance is an attempt to discover and collect information about a target. It could be knowledge about the system in use, the web application, employees or location, etc.


We will start our analysis by examining any reconnaissance attempt against the webserver imreallynotbatman.com. From an analyst perspective, where do we first need to look? If we look at the available log sources, we will find some log sources covering the network traffic, which means all the inbound communication towards our web server will be logged into the log source that contains the web traffic. Let's start by searching for the domain in the search head and see which log source includes the traces of our domain.

Search Query: index=botsv1 imreallynotbatman.com
                                                            
Here we have searched for the term imreallynotbatman.com in the index botsv1. In the sourcetype field, we saw that the following log sources contain the traces of this search term.

Suricata
stream:http
fortigate_utm
iis 

Our first task is to identify the IP address attempting to perform reconnaissance activity on our web server.
                                
Let us begin looking at the log source stream:http, which contains the http traffic logs, and examine the src_ip field from the left panel. Src_ip field contains the source IP address it finds in the logs. 
We have found two IPs in the src_ip field 40.80.148.42 and 23.22.63.114
The first IP seems to contain a high percentage of the logs as compared to the other IP, which could be the answer.

```
Search Query: index=botsv1 imreallynotbatman.com sourcetype=stream http src=40.80.148.42 

sourcetype=suricata                             
alert.signature=ET WEB_SERVER Possible CVE-2014-6271 Attempt in Headers                               
url  /joomla/index.php/component/search/
fileinfo.filename   /joomla/index.php/component/search/ 
http.http_user_agent  ";print(md5(acunetix_wvs_security_test));$a=\"
http_method  ACUNETIX

```
Answer the questions:

One suricata alert highlighted the CVE value associated with the attack attempt. What is the CVE value?  CVE-2014-6271

What is the CMS our web server is using? joomla

What is the web scanner, the attacker used to perform the scanning attempts?  acunetix

What is the IP address of the server imreallynotbatman.com?  192.168.250.70

### Exploitation Phase

The attacker needs to exploit the vulnerability to gain access to the system/server.
Let's use the following search query to see the number of counts by each source IP against the webserver.

To begin our investigation, let's note the information we have so far:

We found two IP addresses from the reconnaissance phase with sending requests to our server.
One of the IPs 40.80.148.42 was seen attempting to scan the server with IP 192.168.250.70.
The attacker was using the web scanner Acunetix for the scanning attempt.


```
Search Query:index=botsv1 imreallynotbatman.com sourcetype=stream* | stats count(src_ip) as Requests by src_ip | sort - Requests

index=botsv1 sourcetype=stream:http dest_ip="192.168.250.70"
src_ip  40.80.148.42  91.438%  23.22.63.114  7.447%
http_method=POST  28.831%
http_user_agent="Python-urllib/2.7"  
form_data: username=admin&task=login&return=aW5kZXgucGhw&option=com_login&passwd=rock&4a40c518220c1993f0e02dc4712c5794=1 

index=botsv1 sourcetype=stream:http dest_ip="192.168.250.70" http_method=POST uri="/joomla/administrator/index.php" form_data=*username*passwd* | table _time uri src_ip dest_ip form_data

index=botsv1 sourcetype=stream:http dest_ip="192.168.250.70" http_method=POST form_data=*username*passwd* | rex field=form_data "passwd=(?<creds>\w+)"  | table src_ip creds

index=botsv1 sourcetype=stream:http dest_ip="192.168.250.70" http_method=POST form_data=*username*passwd* | rex field=form_data "passwd=(?<creds>\w+)" |table _time src_ip uri http_user_agent creds
                                                                    
```
Answer the questions
                                
What was the URI which got multiple brute force attempts?   /joomla/administrator/index.php

Against which username was the brute force attempt made?  admin

What was the correct password for admin access to the content management system running imreallynotbatman.com?  batman

How many unique passwords were attempted in the brute force attempt?  412

What IP address is likely attempting a brute force password attack against imreallynotbatman.com?  23.22.63.114

After finding the correct password, which IP did the attacker use to log in to the admin panel?  40.80.148.42

### Installation Phase

Once the attacker has successfully exploited the security of a system, he will try to install a backdoor or an application for persistence or to gain more control of the system.
To begin an investigation, we first would narrow down any http traffic coming into our server 192.168.250.70 containing the term ".exe." 
Observing the interesting fields and values, we can see the field part_filename{} contains the two file names. an executable file 3791.exe and a PHP file agent.php
Next, we need to find if any of these files came from the IP addresses that were found to be associated with the attack earlier(c_ip).
Following the Host-centric log, sources were found to have traces of the executable 3791. exe.
Sysmon
WinEventlog
fortigate_utm
For the evidence of execution, we can leverage sysmon and look at the EventCode=1 for program execution.

```
Search Query: index=botsv1 sourcetype=stream:http dest_ip="192.168.250.70" *.exe
 filename=\"3791.exe\"
"part_filename{}"="3791.exe"
c_ip="40.80.148.42"

index=botsv1 3791.exe sourcetype=xmlwineventlog EventCode=1 CommandLine="3791.exe"

```
Answer the questions 

Sysmon also collects the Hash value of the processes being created. What is the MD5 HASH of the program 3791.exe?
AAE3F5A29935E6ABCC2C2754D12A9AF0

Looking at the logs, which user executed the program 3791.exe on the server?
NT AUTHORITY\IUSR

Search hash on the virustotal. What other name is associated with this file 3791.exe?
ab.exe  (SHA256=EC78C938D8453739CA2A370B9C275971EC46CAF6E479DE2B2D04E97CC47FA45D)



```

                                
                                
                                                        
                            
                            
                            
                                                            
                                                    


