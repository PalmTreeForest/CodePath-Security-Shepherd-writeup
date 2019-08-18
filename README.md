## CodePath Security Shepherd Challenges Write-Up

# Introduction
Weeks 1-6 of CodePath and Security Shepherd took us through the OWASP Top 10 Application Vulnerabilities and provided us the means to test these vulnerabilities in their contained, safe and legal environment.

## Week 1
After getting burp set up, the certificate installed, and the proxy set, I began to dive into the assignments. The approximate amount of time I spent on this week was 15 hours, including the initial installation and configuration of Burp.

# Challenge 1 - Http Headers
The task was to find the cache-control headers in the server response from Facebook. After installing Burp I navigated to Proxy > Http History > clicked on the GET request for FB > Response > Headers > cache-control.

The correct value is: private, no-cache, no-store, must-revalidate

# Challenge 2 – Basic Routes
The task was to determine the URL that FB uses when the user “Likes” a post. I “Liked” a cat video in FB, switched back to my Burp terminal, and looked for the FB linked URL which made sense with the action. The URL that listed “reaction_type” seemed like the most apt description of user action that describes the act of expressing an opinion and/or their reaction to a particular post.

The correct value for the URL is:
/ufi/reaction/tooltip/?ft_ent_identifier=728293820710695&reaction_type=1&user_count=278&av=100018425533616&dpr=1 HTTP/1.1

# Challenge 3 – Query Parameters
The task was to post a status on FB and determine which query parameters contain the text of my message. Security Shepherd gave us a hint to check URLs that contain ajax/updatestatus, so I selected those events to inspect. I used the search tool at the bottom of Burp to search for the text that I had entered on my post in FB. The first 2 did not contain the text, but was successful on the third try. Under the Request tab > parameters, I found the value which I had entered. I then found the corresponding parameter name.

The correct value for the query parameter is: xhpc_message_text

# Challenge 4 – Insecure Direct Object References
The task was to conduct a man-in-the-middle attack by exploiting an Insecure Direct Object Reference vulnerability. We had to change/modify the request via burp to fetch a different user. I refreshed the login page on Security Shepherd, went to the Burp console, found the corresponding POST request, right-clicked on it and selected “Send to Repeater”. Once it was in Repeater, in the “Raw” tab at the bottom on the request there was a field named “username”. The corresponding value was “guest”.  I deleted this value and replaced it with “admin”. I then clicked the “Go” button, and the response that the server sent back contained the result key, which I copied and pasted into the applicable field in the browser.

# Challenge 5 – IDOR Challenge 1
The task was to notice patterns that administrators often use to assign a value to specific users. We were instructed to find the message of the user who was not in the list in the browser in Security Shepherd. Once you learn the database structure you can craft methods to exploit the database. For this particular challenge we were given a list of names and messages that corresponded to those user names. I clicked on each name individually and pressed the “Show this profile” button for each one. I then went back to Burp and looked at each event that was captured when “Show this profile” was pressed. I noticed that the Request Query Parameter had a field named “userID[]”. Each POST request had different values: 1, 3, 5, 7, 9.

Based upon this pattern of odd numbers, I hypothesized the next userID would be “11”. I sent one of the POST requests to Repeater, deleted the existing userID field value and replaced it with “11” and pressed “Go”. The server response included the result key.

# Challenge 6 – IDOR Challenge 2 (Bonus)
The task in this challenge was similar to Challenge 5. However, in this challenge, the userid was encoded. Our task was to figure out which type of encoding was being used, and then guess the upstream value that would generate the hashed value which we had available to us. First, I tried using the user’s names, but this did not work despite trying many combinations of the encoding and hashing. It turns out the userID plain text value was not a name or a series of letters, it was a number, just like the last challenge. Once I determined the userIDs were numbers, I went back and clicked the “Show this Profile” button for all users, and determined that the user ids were: 2, 3, 5, 7, 11. This is a prime number sequence.

The next value was likely 13. So, I MD5 hashed “13” and plugged the hashed value (c51ce410c124a10e0db5e4b97fc2af39) in to the userID, then pressed “Go”.  The server responded with the results key.

# Challenge 7 – IDOR Bank Challege
The task in this challenge was to make an account and then get the system to transfer money from a different account into the account I created. First, I created an account, then I logged in. The first thing I did was try to transfer money from my account to a random account number. In Burp, I sent this event to Repeater. There I learned my account number (the value next to the “senderAccountNumber” field). I deleted the existing value corresponding to the “receiverAccountNumber” field and inserted my account number (729). I then changed the value corresponding to the “transferAmount” field to “5000001” per the directions in Security Shepherd. I chose to input the account number before mine (728). When I first completed this challenge I took the balance from Account no. 1. I pressed “Go” and was given the message “Funds Transferred Successfully”. I went back to Security Shepherd and refreshed my balance, and despite transferring “5000001” (Security Shepherd is a little buggy), my balance showed as “5000000”. We were supposed to have over $5M Euros, so I went back and transferred another $5M Euros from account no 1. When I refreshed the balance in Security Shepherd again, I had enough funds to complete the challenge. Per the directions, I logged out and then back in, and a result key generated.

## Week 2
The first 2 challenges I tried random SQLI code that I found on websites. Then I discovered Burp’s Intruder which automated the process. I made a list with SQLI payloads I found on various websites and brute forced my way through. Except for the last one, that required a different approach. I spent 14+ hours on this week’s assignments.

# Challenge 0 – SQL Injection

Solution: ‘ or ‘1’=’1

# Challenge 1 – SQL Injection 1

Solution: “ or 0=0 #

# Challenge 2 – SQL Injection 2
For this challenge I had to make a new SQL injection list to use in Burp’s Intruder. I combined the different types of email name variations with some of the injections from the list I had previously created. I loaded the list into intruder and ran it.

Solution: user@[IPv6:2001:DB8::1]' or '1'='1

# Challenge 3 – SQL Injection Escaping

Solution: \’ or 0=0 #

# Challenge 4 – SQL Injection Challenge 3
For this challenge, I was not able to use my injection list, and had to craft an injection using the table and column names provided by Security Shepherd. I also found it was easier to use Burp Repeater than to inject directly on the website. I was also not able to get the site to reveal the whole table and could only get individual columns to show for each query.

First I used:    Mary Martin' AND false; UNION ALL SELECT creditcardnumber FROM customers; #; --;

This gave me 5 rows of credit card numbers:
1245 2514 2315 2147
5468 1763 1854 1451
9815 1547 3214 7569
175 1244 4758 8854
8454 1244 4712 2144

Then I ran:  Mary Martin' AND false; UNION ALL SELECT customername FROM customers; #; --;

This gave me 5 rows of names:

Mark Denihan
Jason McCoy
Mary Martin
Joseph McDonnell
John Doe

The objective for this challenge was to get Mary’s credit card number. Mary’s name was third in the customername column, and I matched that to the third credit card number from the creditcardnumber column.

The result key is Mary Martin’s credit card number: 9815 1547 3214 7569

## Week 3
This week’s challenges gave us basic practice in exploiting cross site scripting vulnerabilities. We were challenged to overcome some of the different filtering mechanisms employed by developers. This week’s challenges to me approximately 10-15 hours to complete.

# Challenge 0 – Cross-site scripting introduction

Solution: <SCRIPT>alert('XSS')</SCRIPT>

# Challenge 1 – Cross-site scripting 1
For this challenge the developer had filtered out <script> tags.

When <SCRIPT>alert('XSS')</SCRIPT> was input, the filter returned “ alert('xss') “.

Solution: <IMG SRC="#" ONERROR="alert('XSS')"/>

# Challenge 2 – Cross-site Scripting 2
CodePath directed us to try 4 different types of XSS, and then gave us a hint to visit W3schools DOM Events Objects webpage. I used the third template and then changed the DOM Event from “ONCLICK” to “oncontextmenu”.

Solution: <INPUT TYPE="BUTTON" oncontextmenu="alert('XSS')"/>

# Challenge 3 – Cross-site scripting 3
For this challenge we needed to insert a null value in html or hex to get around the DOM Event filter. The filter was set to drop a continuous text string with all standard DOM Events filtered out. The filter was not set to recognize and then ignore null encoded characters and so read “O” and “NCLICK”, which it did not filter. When the script got to the server, the server ignored the null encoded character and read “ONCLICK”.

Solution: %3CINPUT+TYPE%3D%22BUTTON%22+O%00NCLICK%3D%22alert('XSS')%22%2F%3E

# Challenge 4 – Cross Site Scripting 4
This challenge was similar to the last one in that we had to insert a html or hex encoded null value into the script string to get around the filter.

Solution: https://www.google.com"oNload=alert('hello')
 
The remaining challenges for this week were optional, and I chose to start work on the next week instead.

## Week 4
The Week 4 Lab challenges focused on exploiting Session Management and practicing Cross Site Request Forgery. I found week 4’s challenges to be easier than prior weeks, and spent approximately 10 hours completing the assignment. 

# Challenge 0 – Broken Session Management
This task involved using Burp to conduct a man-in-the-middle attack to change cookie values in order to obtain a desired result. I pushed the “Lesson Complete” button on the Security Shepherd module, and then sent the event to Burp’s Repeater to have a look at the information. The cookie value was lessonComplete=lessonNotComplete. I deleted the “Not” in lessonNotComplete and pressed “Go”.

Solution: lessonComplete=lessonComplete

# Challenge 1 – Session Management Challenge 1
For this task we captured the POST request on Burp, decoded the cookie checksum using Base64, and changed the userRole from “user” to “administrator”. I also changed the adminDetected, returnPassword, and upgradeUserToAdmin fields from “false” to “true”.

Solution: checksum=dXNlclJvbGU9YWRtaW5pc3RyYXRvcg==

# Challenges 2-5
I did not record the solutions for Challenges 2-5, and so have included a screenshot of the Security Shepherd site showing that I have completed these challenges. 

 <img src="https://github.com/PalmTreeForest/CodePath-Security-Shepherd-writeup/blob/master/Week4Ch2-5.gif" width="800">


## Week 5
For this week’s challenges I made use of the open source tools available at dcode.fr and igolder.com. These challenges were easier than weeks 1-3, and mostly involved searching through open source tools to determine which ciphers had been used. I estimate that I spent 5 to 10 hours on this week’s challenges.

# Challenge 0 – Insecure Cryptographic Storage
For this task, we were given a value in Base64 and we were to decode it. I used Burp Suite to complete this task.

Solution & Result Key: base64isNotEncryptionBase64isEncodingBase64HidesNothingFromYou

# Challenge 1 – Insecure Cryptographic Storage Challenge 1
For this task we were given a value that had been encoded using an undisclosed cipher. Our task was to determine what cypher had been used an decode the value. I used the Cesar Cipher Decoder at dcode.fr, and used brute force to determine that ROT 5 had been used to encode the key.

Solution: THERESULTKEYFORTHISLESSONISTHEFOLLOWINGSTRINGMYLOVELYHORSERUNNINGTHROUGHTHEFIELDWHEREAREYOUGOINGWITHYOURBIGA

Result Key: MYLOVELYHORSERUNNINGTHROUGHTHEFIELDWHEREAREYOUGOINGWITHYOURBIGA

# Challenge 2 – Insecure Cryptographic Storage Challenge 2
For this task we were directed to inspect source code to see if the developer had left any clues about the login in the source code. The developer had left the encrypted value and the encryption key in the iframe source code. The developer had used Vigenere encoding. I used dcode.fr’s Vigenere Decoder tool to get the plain text value.

Solution & Result Key: TheVigenereCipherIsAmethodOfEncryptingAlphabeticTextByUsingPoly

# Challenge 3 – Insecure Cryptographic Storage Challenge 3
For this task we were given a plain text message and a PGP Public Key. We were to encrypt the plain text message with the public key. I used an online tool at igolder.com.

Solution: -----BEGIN PGP MESSAGE-----
Version: BCPG C# v1.6.1.0

hQIMA/7ILmbOlqA1AQ//TsNEbmImzfVdz/bwcyBx+m2HjCPBZdwIJsMArpbXPqrK
wNTr7fHVcykc5wjsZYUO+JYkFDIRJjD+47Rwr3BuJG+BhI88dcfJmMsz2bMDJncA
q7bvJV97XCD0U0eR0o+Sr06VlLbWf/xbPsMiiHFNHkC3zaguxFreohVOtlm6u32e
X4O694XYhxLqbdpfsbyA/9jb1ccP+hHrbs0aKqYtLWUt6j2iCkwoIbOqWkLOg2k4
r6Ij695pziFuFoT9UbpmwP5fCyNapPQepCPdUtAKx+ibe3AvXQJOyr/k5YghP7x/
1ZMxOpU+lCJJQ2OJZ8+ahxwmZvao/J9GdO7vYYEusRYZ2u2ph7IsNX1HU2TxOnFZ
dNnVndOFU8jWPAESA2rT/hLtA3CYeSSFlqnsDXOrpKgH1+er8yrEmwX9Zn+Mojmt
mASAuMjlVdEutH1jcyCcHLP1SxZmnnaAT7GiapoRtO87FSEBk4vEvUr9x/qcgF7b
+lpS3Xb0xb85S5weeb+Bejpf5ygAe6YBVUstBpUgYvcZvQ+OgCY3Qf6zxEmRjMXh
OL4/eSiDVLVc7jeq/HMKHm05QnDvrm/aN2cm4Wcj51aeR37qv+sUB0HZZamojXzD
pM2JO2kZ1tV6Bfo4+9d9b587wsgoVJ776AusnnywWixhoCBJp7OSOMfXSMZqpU7J
bOcEmt3V0iTl1yRgM6+flECsI+Px369ViJmverKGdOm/ClXFMWveK27ahATORMlV
aVkaMvSC0TrB3Nx6m2VZr+Obk6JbG8JfS2PD+zTzSY2zPTxTR1KitDxK34dZO1L2
vqtwOj9TMFWjuHQ7ZA==
=eSy6
-----END PGP MESSAGE-----

There was one optional challenge remaining, but I elected to move on to the next week to keep up with the class.

## Week 6
The Week 6 challenges ranged from relatively easy to fairly difficult. The most difficult challenge for this week was Challenge 2 – User Authentication Challenge 2.

# Challenge 0 -User Authentication
The purpose of this challenge was to demonstrate that default login credentials are not always changed. Security Shepherd gave us a login and password prompt to try out default passwords on.

Solution: User Name = admin     Password = password

# Challenge 1 – User Authentication Challenge 1
For this challenge, the task was to notice how encoding was applied differently to a value and placed in different fields. We needed to gain access to the system as an administrator and had to cycle through login variations and session IDs to login to the admin account. I changed the checksum, JSESSIONID3, and the userID values to get access to the admin account.

Solution: checksum=dXNlclJvbGU9YWRtaW4=           		 //userRole=admin
	    JSESSIONID3="HGBxBMyIE4+aLs1QTXhM1w=="      /*0000000000000021 sent through Base64 encoding twice*/
	userID=0000000000000021

# Challenge 2 – User Authentication Challenge 2
Solution: I changed the checksum from userRole=user to userRole=admin. Then I removed the “subUserName” from the POST request and replaced it with the “data” fields from the source code. I changed the trailing string on the POST header from “SendToken” to “ChangePass”. When I pressed go, the server response indicated that the token needed to be a time/date in GMT. The correct time format is DAY MN DT HR:MN:SC GMT YYYY. The value must be encoded with Base64.
	Once these values were changed then the system accepted the password change request and I was able to login with the credentials I had set via Security Shepherd.

# Challenge 3 – User Authentication Challenge 3

Solution:  x" and false UNION SELECT userid FROM users#

Challenge 4 User Auth Challenge 4 was optional, chose to do other work instead

# Challenge 4 – Password Hashing 1
The task for this challenge was to crack an MD5 password hash using Hashcat and the Rockyou-75 word list. The plain text value of the MD5 hashed password (and the Result Key) was: iloveyou!

# Challenge 5 – Password Hashing 2
For this task, we were given a salted SHA1 hashed password, as well as the salt value. We were told that the salt value was at the beginning of the hash. Our instructions were to figure out what hash-mode to use. The hash-mode was 120 in hashcat.

The plain text value of the SHA1 salted and hashed password (and the Result Key) was: pandemonium

# Challenge 5 – Password Hashing 2
For this task, we were given a salted SHA512 hashed password as well as the salt value. We were told that the salt value was at the beginning of the hash. Our instructions were to figure out what hash-mode to use.

## Conclusion
Overall, I found this experience to be highly valuable and I will be using the upcoming months to complete the challenges that I was not able to get to this fall. This course represents a very large time commitment, and I spent more time completing the work for this course than I did either of the two regular courses at Merritt. I recommend this course to everyone, but with the stipulation that it will be difficult to complete if the student has a full-time job and/or is taking additional courses. Further, I believe that a student who works full time should choose either this course or compete in external capture the flag competitions, but probably not both at the same time.
	In terms of hardware requirements, a student will need to make sure that they have sufficient free disk space to run the virtual machines from their primary operating system. I had not initially set up my primary operating system with sufficient hard disk space to run virtual machines, and spent a significant amount of time trying to solve this problem, time that could have been spent completing the assignments for this class or for other classes I was enrolled in.
	One of the best aspects of this course was the coordination and teamwork between the students. We learned from each other and helped each other complete the challenges, and thus built and practiced interpersonal skills that are necessary in a professional setting. I am looking forward to participating with the Merritt students in the Independent Study activities and continuing to learn what I can from the CodePath curriculum.
