Get familiar with running search queries in the `Discover` screen with Packetbeat. This will be located on your fourth tab in Chrome. 

- On the Discover page, locate the search field.
- Start typing `source` and notice the suggestions that come up.
- Search for the `source.ip` of your attacking machine.
- Use `AND` and `NOT` to further filter you search and look for communications between your attacking machine and the victim machine.
- Other things to look for: 
	- `url`
	- `status_code`
	- `error_code`

After creating your dashboard and becoming familiar with the search syntax, use these tools to answer the questions below:


1. Identify the offensive traffic.
   - Identify the traffic between your machine and the web machine:
     - When did the interaction occur?
     - What responses did the victim send back?
     - What data is concerning from the Blue Team perspective?

the interaction occurs on the machine feb2, @ 1.05PM and occurs again at 1:45 and 150 PM. 
lost of 401 errors. also note a lot of 200 went through. this is because I dirbusted the secret_folder using credentials.  Lots of Errors and is definately worrying that there many successes. 

2. Find the request for the hidden directory.
   - In your attack, you found a secret folder. Let's look at that interaction between these two machines.
     - How many requests were made to this directory? At what time and from which IP address(es)?
     - Which files were requested? What information did they contain?
     - What kind of alarm would you set to detect this behavior in the future?
     - Identify at least one way to harden the vulnerable machine that would mitigate this attack.

51,000 requests into the /secret_folder/ and 5000 into /secret_folder. requests are screaming in from 192.168.1.90. secret folder files were gotten into and contains the secret information. 

alerts could consist for unique flows of traffice being way over baseline to trigger and including lockouts for response codes that are clearly unsuccessful attempts.


3. Identify the brute force attack.
   - After identifying the hidden directory, you used Hydra to brute-force the target server. Answer the following questions:
     - Can you identify packets specifically from Hydra?
     - How many requests were made in the brute-force attack?
     - How many requests had the attacker made before discovering the correct password in this one?
     - What kind of alarm would you set to detect this behavior in the future and at what threshold(s)?
     - Identify at least one way to harden the vulnerable machine that would mitigate this attack.

by filerting by the source ip and adding the url.original to /company_folders/secret_folder. 10,148 total bruteforce requests were made.  two out of these requests were response 301 which results in event.outcome: success, meaning the user would be redirected into the secret_folder.  The brute force into a folder like this needs a timeout or there will be no way of mitigating brute force threats. 

4. Find the WebDav connection.
   - Use your dashboard to answer the following questions:
     - How many requests were made to this directory? 
     - Which file(s) were requested?
     - What kind of alarm would you set to detect such access in the future?
     - Identify at least one way to harden the vulnerable machine that would mitigate this attack.

616 requests were made into the webdav folder because I tried to login through those credentials also. files requested all wiithin directory. alarm would need to be set 
two factor and timeouts. honeypot too for credentials that don't exist but exist and then blacklist those users by banning their traffic. removing functionality from the webdav connectins help. 

5. Identify the reverse shell and meterpreter traffic.
   - To finish off the attack, you uploaded a PHP reverse shell and started a meterpreter shell session. Answer the following questions:
     - Can you identify traffic from the meterpreter session?
     - What kinds of alarms would you set to detect this behavior in the future?
     - Identify at least one way to harden the vulnerable machine that would mitigate this attack.


reverse php shell detected meterpreter session. 
from packetbeat. destination to 192.168.1.90 on port 4444. can block this behavior going outbound and making calls. since the folder is not intended to contain reverse payloads shells . perhaps disallowing the operation of those files would prevent apache server making http requests in the dav to go outbound. 


