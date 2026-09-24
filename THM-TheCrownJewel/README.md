# THM THECROWNJEWEL


<a href="https://tryhackme.com/room/thecrownjewel" Link to room><a/>

The Objective of this room is to be able to investigate a PCAP for signs of data exfiltration following an reverse shell alert. For this room I will be structuring the write up to fit the order of which the questions were presented on tryhackme. We are provided with the following tools to complete the room.

<ul>
  <li> Wireshark </li>
  <li> Splunk </li>
</ul>

"as a side note, for some reason my Splunk was not working correctly during the beginning of this challenge. I've had to use Wireshark for questions 1 and 2 when they should have been done in Splunk to make it alot easier"

# 1) From which internal IP did the suspicious connection originate?

Since we were given the information that it was a suspected reverse shell, we can filter for plaintext of common commands that could be used in this in Wireshark by looking for any TCP traffic containing certain strings. I decided to use the following.

"tcp contains "Microsoft Windows" || tcp contains "bash" || tcp contains "id=" || tcp contains "whoami"

breaking this down.
<ul>
  <li> the || parts of the command stand for "OR", basically allowing us to filter for packets that could contain any of these requirements, if we used && then the packets would need to contain all four</li>
  <li> tcp contains ____ = instructs wireshark to look inside the payloads of any TCP packets and check if it contains the specified string.</li>
  <li> Microsoft Windows = could show a remote command prompt being exposed over TCP </li>
  <li> Bash = used commonly in Linux reverse shells</li>
  <li> "id =" = shows if the id command was used, the id command shows the current users privileges</li>
  <li> whoami = the whoami command shows which user account is executing the commands, used in both windows and linux</li>
</ul>

![naughtyip](https://github.com/R-SYN98/CTF-Write-ups/blob/main/THM-TheCrownJewel/Images/sussyip.png)

as we can see in the image we were left with one packet. It looks like its sending packets to a DNS IP address, but regardless we have our first answer.

# 2) What outbound connection was detected as a C2 channel?

Actually we can see in the previous image, we have already answered this question. The Destination address and port in the previous packet is the outbound connection that question 2 is looking for. It's also worth noting down some more information about the compromised target, like its MAC address which is "00:0c:29:11:22:33".

# 3) Which MAC address is impersonating the gateway 10.10.10.1?

For this question we need to understand how ARP works. The Address Resolution Protocol basically is used to map ip addresses to MAC addresses. Normally when Device A wants to talk to Device B on the same local network, it knows the IP but needs the MAC for Device B to send data packets. It normally does this via broadcasting an "ARP Request" to the entire network, then Device B would respond to this and provide it's MAC address in an "ARP reply". 

ARP spoofing works because devices will accept ARP replies even if they never sent out an ARP request. The attacker will send the ARP reply to the target and cause the target to cache the attacker's MAC address with the wrong IP address. This is used most commonly in Man In the Middle Attacks.

I assume that due to the question mentioning the MAC address is impersonating a gateway ip. It it almost certainly done Via ARP spoofing. To filter for this we apply this filter.

"arp.opcode == 2 && arp.src.proto_ipv4 == 10.10.10.1"

to break this down:

<ul>
  <li>arp.opcode ==2 = filters for ARP replies only</li>
  <li>&& = the AND operator, makes it so that the packets have to match all requirements rather than || OR</li>
  <li>arp.src.proto_ipv4 == 10.10.10.1 = filters for only ARP replies that contain the gateway address, this helps us filter out any genuine ARP reply packets</li>
</ul>

![arp](https://github.com/R-SYN98/CTF-Write-ups/blob/main/THM-TheCrownJewel/Images/arpspoof.png)

We then open a packet and find the source MAC Address, as we can see that the MAC address contained within the packets is the same as our compromised MAC address from earlier. 

# 4) How many ARP spoofing attacks were observed in the PCAP?

Easily enough we can just remove the latter part of the filter to just show ARP reply packets to get this answer

![arpr](https://github.com/R-SYN98/CTF-Write-ups/blob/main/THM-TheCrownJewel/Images/manyspoof.png)

we can see the bottom of wireshark that 90 packets are visible.

# 5) What is the non-standard User-Agent hitting the Jira instance?

For this question I needed to use Splunk. I spent ages on this trying to make it work in wireshark but there were no packets with the answer I needed. However I found a website that helped me troubleshoot my issue and managed to get it working.

Simply we can just highlight the Agent field in Splunk to see all User-Agents that are present in the pcap.

![splunk](https://github.com/R-SYN98/CTF-Write-ups/blob/main/THM-TheCrownJewel/Images/naughtyuseragent.png)

CVE being present in the name and the words EXPLOIT is just an immediate red flag. 

# 6) What's the payload containing the plaintext creds found in the POST request?

I accidently figured the answer to this while I was trying to use Wireshark for question 5. Basically when filtering for HTTP traffic, You can add a new column to wireshark containing user agents by right clicking the highlighted section in the image below and selecting "add as column"

![column](https://github.com/R-SYN98/CTF-Write-ups/blob/main/THM-TheCrownJewel/Images/column.png)

Now with this I ordered the user agents alphabetically and was presented with only one packet that was missing any information, I assume that this packet is the same one from question 5 mentioning the CVE.

![accidental](https://github.com/R-SYN98/CTF-Write-ups/blob/main/THM-TheCrownJewel/Images/accidental.png)

You can clearly see in plaintext a username and password, indicating this POST request was a login attempt. To get the payload I simply followed the HTTP stream and was presented with this.

![payload](https://github.com/R-SYN98/CTF-Write-ups/blob/main/THM-TheCrownJewel/Images/payload.png)

# 7) What domain, owned by the attacker, was used for data exfiltration?

With the question mentioning domains, I knew that I needed to filter for DNS traffic. Once the filter was applied I searched through the domains mentioned and found two packets containing the same domain with abnormally large subdomains.

![dns]![payload](https://github.com/R-SYN98/CTF-Write-ups/blob/main/THM-TheCrownJewel/Images/dns.png)

# 8) After examining the logs, which protocol was used for data exfiltration?

As we can see from question 7 the protocol used for the data exfiltration was DNS.

and with that we have completed the room.

# Answers

<ul>
  <li> From which internal IP did the suspicious connection originate? = 10.10.10.100</li>
  <li> What outbound connection was detected as a C2 channel? = 1.1.1.1:8080</li>
  <li> Which MAC address is impersonating the gateway 10.10.10.1? = 00:0c:29:11:22:33 </li>
  <li> What is the Non-standard User-Agent hitting the Jira Instance? = CVE-202X-EXPLOIT</li>
  <li> How many ARP spoofing requests were observed in the PCAP? = 90</li>
  <li> What's the payload containing the plaintext creds found in the POST request? = username=dev_user&password=SecretPassword!</li>
  <li> What domain, owned by the attacker, was used for data exfiltration? = exfil-domain.xyz</li>
  <li> After examining the logs, which protocol was used for data exfiltration? = DNS</li>
</ul>

Thank you for reading :)
