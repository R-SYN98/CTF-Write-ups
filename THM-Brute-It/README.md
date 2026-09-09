# THM Brute It

<a href="https://tryhackme.com/room/bruteit" Link to room> <a/>

This room tests us on our ability to deploy brute force attacks against a vulnerable web server. For this room I will be using Kali Linux instead of the attack box provided by THM as I'm more familier with this distro of Linux.

# Reconnaissance Phase

Our first objective is to do some recon and figure out information on the vulnerable web server. We will do this with a simple Nmap scan.

" Nmap 10.130.145.4 -sV -p- " 

![nmap](https://github.com/R-SYN98/CTF-Write-ups/blob/main/THM-Brute-It/images/Screenshot%202026-09-09%20093802.png)

breaking the command down
<ul>
  <li> -sV probes any open ports to determine service and version info </li>
  <li> -p- tells nmap to scan all 65,535 possible network ports on the target machine</li>
</ul>


Now as you can see by the image we gleaned a fair bit of information about the target machine
<ol>
  <li>The version of SSH running on the machine is OpenSSH 7.6p1</li>
  <li>The version of Apache that the webserver is running is 2.4.29</li>
  <li>The machine is running Linux and the distro is most likely Ubuntu</li>
</ol>

Now that we know that there's a web server running on port 80 we can try having a look at the website on a browser

remember to put http at the beginning of the url because the search bar will default to port 443 "https"

![website](https://github.com/R-SYN98/CTF-Write-ups/blob/main/THM-Brute-It/images/Website.png)

Nothing really of note here but we know for sure that there is a website here, now we can move onto using Gobuster to enumerate any domains that may be present on this website.

"Gobuster dir -u http://10.130.145.4 -w /usr/share/wordlists/dirb/common.txt "

to explain this command again
<ul>
  <li> dir makes gobuster work in directory mode </li>
  <li> -u is the flag that denotes the target, in this case we use the url of the website</li>
  <li> -w is the wordlist, this is what gobuster will reference to when checking the domains, its like a checklist for gobuster, we used the common.txt that comes default on kali as we don't need any bigger ones than this one</li>
</ul>

![gobuster](https://github.com/R-SYN98/CTF-Write-ups/blob/main/THM-Brute-It/images/gobuster.png)

One thing we can ignore for the time being is both the 403 error results and the index.html domain. Reason being is that index.html is generally the default homepage of a website and will just lead back to the initial image of the website above, and 403 is a Forbidden error, meaning we don't have the permissions to view these. 

With this we have some section to look at, /admin.

![admin](https://github.com/R-SYN98/CTF-Write-ups/blob/main/THM-Brute-It/images/admin.png) 

This is a login page for an administrator. Now that we have are at this page we can move onto our second section of the CTF.

# Getting a Shell

Our next step to is get a shell onto the webserver, we can start by trying to brute force the password for the admin user. Before we use our brute forcing tool we need to gather some information.

<ul>
  <li>What is the error message when we try to log in?</li>
  <li>Does the system lock us out after too many failed attempts?</li>
  <li>What is the name of the input section for the password and username sections in the HTML?</li>
</ul>

We can easily see if the system locks us out by spamming failed attempts. After I tried this I didn't get timed out and I was presented with this error message.

![error](https://github.com/R-SYN98/CTF-Write-ups/blob/main/THM-Brute-It/images/Failed.png)

" Username or password is invalid "

now we can use the inspector section in the Devtools of the browser with F12.

![logininfo](https://github.com/R-SYN98/CTF-Write-ups/blob/main/THM-Brute-It/images/Logininfo.png)

We can see that the website reads the username input as "user" and the password section as "pass".

With this we can now brute force the admin's password with Hydra using this following command.

"hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.130.145.4 -s 80 http-post-form '/admin/:user=^USER^&pass=^PASS^:F=Username or password invalid'"

again breaking this command down 

<ul>
  <li>-l is the username to be used, if you wanted to brute force a username you could put a path to a wordlist to use too </li>
  <li> -P is the password that's going to be used, here we used the rockyou.txt wordlist</li>
  <li> 10.130.145.4 is the link to the website </li>
  <li> -s is us telling hydra which port to use, 80 is http</li>
  <li> http-post-form is the section telling us what method hydra should use to send over the requests to the service, with this we're telling hydra to use post requests</li>
  <li> '/admin/ is telling hydra which path to use on the website, this ensures the requests are sent to /admin</li>
  <li> :user=^USER^&pass=^PASS^: this section tells hydra where it should put the values of the username and password flags, we change the user and pass sections depending on the target, but because the website's html has username and password input names as user and pass then we use these names here.</li>
  <li>F=Username or password invalid' is telling hydra the error message to look out for so it knows that the attempt didn't work.</li>
</ul>

![hydra](https://github.com/R-SYN98/CTF-Write-ups/blob/main/THM-Brute-It/images/Hydra.png)

Hydra has gathered that the password for the admin account is xavier, with this we attempt to log into the /admin section with these credentials and are greeted with this screen.

![rsalogin](https://github.com/R-SYN98/CTF-Write-ups/blob/main/THM-Brute-It/images/rsalogin.png)

We get our first flag and then we can see there's a link embeded into the text that leads us to an RSA key for the user John.

![rsa](https://github.com/R-SYN98/CTF-Write-ups/blob/main/THM-Brute-It/images/RSA.png)

we can now use the curl command to download this RSA key to our machine and then we can start prepping it to be cracked.

![curl](https://github.com/R-SYN98/CTF-Write-ups/blob/main/THM-Brute-It/images/curl.png)

Firstly we need to get this file ready to be read by John before we can crack it, then once we've prepped it we can use John to crack the password as seen below.

![john](https://github.com/R-SYN98/CTF-Write-ups/blob/main/THM-Brute-It/images/john.png)

Now we can see that John's RSA private key password is rockinroll. Now we can now connect to the machine via SSH.

Before we SSH we use the CHMOD command with 600 on the file to make sure that file can be read during the connection, then we can use the SSH command with the -i flag to include the RSA key and use John's username with his RSA password to connect to the webserver machine.

![ssh](https://github.com/R-SYN98/CTF-Write-ups/blob/main/THM-Brute-It/images/ssh.png)

we can use ls to find the user.txt file requested by THM and then cat it to get the next flag.

# Privilege Escalation

Now our job is to escalate our privilege to gain access to the root folder. We first can see what commands that John has access to by using the "Sudo -l" command.

![sudo](https://github.com/R-SYN98/CTF-Write-ups/blob/main/THM-Brute-It/images/cat.png)

we can see that John has access to run the Cat command with Sudo without needing the root password. Knowing this information we can use Sudo Cat /etc/shadow to get a print out of password hashes of the users, we can also pipe the grep command with the word root to try and only bring up the root's password hash.

![cat](https://github.com/R-SYN98/CTF-Write-ups/blob/main/THM-Brute-It/images/actualcat)

We can then copy the hash and bring it back to our machine to get it cracked by john, I like to use the nano command to make a .txt file with the hash and then run it through john.

![adminjohn](https://github.com/R-SYN98/CTF-Write-ups/blob/main/THM-Brute-It/images/rootpass.png)

We can see that john has cracked the hash and the root password is football. Hopping back to the shell we can use the su command and then try the password provided to switch to the root user of the machine.

![su](https://github.com/R-SYN98/CTF-Write-ups/blob/main/THM-Brute-It/images/su.png)

Now we have escalated our privilege we can have a snoop around for root.txt and get the final flag. With that the CTF is complete.

# Answers

Reconnaissance
<ul>
  <li>How many ports are open? = 2</li>
  <li>What Version of SSH is running? = OpenSSH 7.6p1</li>
  <li>What version of Apache is running? = 2.4.29</li>
  <li>Which Linux Distribution is running? = Ubuntu</li>
  <li>What is the hidden directory? = /admin</li>
</ul>

Getting a shell
<ul>
  <li>What is the user:password of the admin panel? = admin:xavier</li>
  <li>What is John's RSA Private Key passphrase? = rockinroll</li>
  <li>user.txt = THM{a_password_is_not_a_barrier}</li>
  <li>Web flag = THM{brut3_f0rce_is_e4sy}</li>
</ul>

Privilege Escalation
<ul>
  <li> What is the root's password? = football</li>
  <li>root.txt = THM{pr1v1l3g3_3sc4l4t10n}</li>
</ul>


Thank you for reading my write up :)
