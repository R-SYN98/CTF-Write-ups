# THM THECROWNJEWEL


<a href="https://tryhackme.com/room/thecrownjewel" Link to room> <a/>

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

![naughtyip](https://github.com/R-SYN98/CTF-Write-ups/blob/main/THM-TheCrownJewel/images/sussyip.png)
