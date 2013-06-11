使用Nmap进行代理IP识别的可行性研究
> *原文写于2012-8-8*

IP地址在互联网上被广泛使用，用来判定用户所在国家、地区、城市等位置信息，在网络反欺诈中也经常使用IP地址来判断用户是否存在欺诈嫌疑，比如：福建莆田以骗子众多而出名，如果IP地址出现在这个地区要特别小心，我们可以根据这样的位置判断来减少交易的风险。《<a href="https://www.vtrenz.net/imaeds/ownerassets/818/KnYrEnemy2.pdf" target="_blank">Geolocation -Knowing Your Enemy</a>》这篇文章中提到了几种常用的、通过<a href="http://en.wikipedia.org/wiki/Geolocation" target="_blank">Geolocation</a>位置信息来识别欺诈的方法。但是，随着在线代理的兴起和骗子技术手段的日益升级，这些方法慢慢失灵了，因为要想通过<a href="http://en.wikipedia.org/wiki/Geolocation" target="_blank">Geolocation</a>来反欺诈，首先要拿到骗子的真实IP地址，确定他们的真实位置，而通过在线代理的方式很容易绕过这些规则。如果识别用户是否使用了代理IP来恶意绕过系统规则，是保证<a href="http://en.wikipedia.org/wiki/Geolocation" target="_blank">Geolocation</a>技术有效的重要手段。 

用户不管通过何种Proxy访问网站时，走的都是HTTP协议，而HTTP协议头中其实也有相应的字段来标明是否使用了代理，比如这篇《<a href="http://blog.sina.com.cn/s/blog_73224682010105kj.html" target="_blank">检测代理IP匿名程度的方法</a>》中详细介绍，但现在很多代理服务器很容易地把协议中的代理信息屏蔽掉，因为这些值并非必需的，从而使从HTTP协议中识别的技术失效，需要借助其它技术手段来识别。 

从常识中可以知道，普通用户一般常用的操作系统为Windows(XP、Vista、7)，Mac OS，而服务器一般是Linux、Windows Server 2003等服务器操作系统，并且个人电脑一般很少开对外端口提供服务，当前有现成的<a title="又叫TCP/IP Stack Fingerprinting" href="http://en.wikipedia.org/wiki/OS_fingerprinting" target="_blank">OS Fingerprinting</a>技术可以识别用户主机上面的操作系统是什么版本，打开的端口都有哪些，通过操作系统的判定和端口的判定，基本可以准确识别是否是代理IP。这种探测技术又分两种：主动式和被动式，主动式是主动发送协议包到对方主机上，通过返回的包信息来确定，这种方式比较直接但隐蔽性不太好，<a href="http://nmap.org" target="_blank">Nmap</a>是代表。被动式是当对方访问我们的主机时通过抓包来判断，这种方式比较隐蔽对方完全不知道，但感觉效果不太好，<a href="http://lcamtuf.coredump.cx/p0f3/" target="_blank">p0f</a>和<a href="http://en.wikipedia.org/wiki/Ettercap_(computing)" target="_blank">ettercap</a>。 

每一种操作系统对网络协议包的返回内容都有所不同，主要是在TTL、Windows Size等标识位上有所区别，比如不同的操作系统的TTL和WS如下所示： 

<table border="1">
  <tr>
    <th>
      Operating System (OS)
    </th>
    
    <th>
      IP Initial TTL
    </th>
    
    <th>
      TCP window size
    </th>
  </tr>
  
  <tr>
    <td>
      <strong>Linux (kernel 2.4 and 2.6)</strong>
    </td>
    
    <td>
      64
    </td>
    
    <td>
      5840
    </td>
  </tr>
  
  <tr>
    <td>
      <strong>Google's customized Linux</strong>
    </td>
    
    <td>
      64
    </td>
    
    <td>
      5720
    </td>
  </tr>
  
  <tr>
    <td>
      <strong>FreeBSD</strong>
    </td>
    
    <td>
      64
    </td>
    
    <td>
      65535
    </td>
  </tr>
  
  <tr>
    <td>
      <strong>Windows XP</strong>
    </td>
    
    <td>
      128
    </td>
    
    <td>
      65535
    </td>
  </tr>
  
  <tr>
    <td>
      <strong>Windows 7, Vista and Server 2008</strong>
    </td>
    
    <td>
      128
    </td>
    
    <td>
      8192
    </td>
  </tr>
  
  <tr>
    <td>
      <strong>Cisco Router (IOS 12.4)</strong>
    </td>
    
    <td>
      255
    </td>
    
    <td>
      4128
    </td>
  </tr>
</table> 

使用<a href="http://nmap.org" target="_blank">Nmap</a>探测我的一台装有Windows XP操作系统的笔记本 
```
xxx@desktop:~$ sudo nmap -O -v 10.19.214.97

Starting Nmap 6.01 ( http://nmap.org ) at 2012-08-03 16:25 CST
Initiating Ping Scan at 16:25
Scanning 10.19.214.97 [4 ports]
Completed Ping Scan at 16:25, 0.02s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 16:25
Completed Parallel DNS resolution of 1 host. at 16:25, 0.00s elapsed
Initiating SYN Stealth Scan at 16:25
Scanning 10.19.214.97 [1000 ports]
Discovered open port 135/tcp on 10.19.214.97
Discovered open port 445/tcp on 10.19.214.97
Discovered open port 139/tcp on 10.19.214.97
Discovered open port 8090/tcp on 10.19.214.97
Completed SYN Stealth Scan at 16:25, 4.86s elapsed (1000 total ports)
Initiating OS detection (try #1) against 10.19.214.97
Retrying OS detection (try #2) against 10.19.214.97
Nmap scan report for 10.19.214.97
Host is up (0.0045s latency).
Not shown: 995 filtered ports
PORT     STATE  SERVICE
135/tcp  open   msrpc
139/tcp  open   netbios-ssn
445/tcp  open   microsoft-ds
3389/tcp closed ms-wbt-server
8090/tcp open   unknown
Device type: general purpose|WAP|broadband router
Running (JUST GUESSING): Microsoft Windows 2003|XP|2000 (93%), IBM AIX 6.X (88%), SMC embedded (88%), Belkin embedded (85%), Philips embedded (85%), Express Logic ThreadX G3.X (85%), T-Home embedded (85%)
OS CPE: cpe:/o:microsoft:windows_server_2003::sp2 cpe:/o:microsoft:windows_xp::sp3 cpe:/o:ibm:aix:6 cpe:/o:microsoft:windows_2000::sp4 cpe:/o:expresslogic:threadx:g3
Aggressive OS guesses: Microsoft Windows Server 2003 SP2 (93%), Microsoft Windows XP SP3 (93%), Microsoft Windows XP (92%), Microsoft Windows XP SP2 - SP3 (90%), Microsoft Windows Fundamentals for Legacy PCs (XP Embedded derivative) (88%), IBM AIX 6.1 (88%), SMC 7904WBRA-N wireless ADSL router (88%), Microsoft Windows 2000 SP4 (87%), Microsoft Windows XP SP2 or SP3 (87%), Microsoft Windows Server 2003 (87%)
No exact OS matches for host (test conditions non-ideal).
TCP Sequence Prediction: Difficulty=259 (Good luck!)
IP ID Sequence Generation: Incremental

Read data files from: /usr/bin/../share/nmap
OS detection performed. Please report any incorrect results at http://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.12 seconds
           Raw packets sent: 2057 (93.904KB) | Rcvd: 191 (8.700KB)
```

探测一个<a href="http://www.ip-adress.com/proxy_list/" target="_blank">IP-Address网站公布</a>的代理IP 
```
xxx@desktop:~$ sudo nmap -O -v 174.34.243.140
[sudo] password for simbo: 

Starting Nmap 6.01 ( http://nmap.org ) at 2012-08-03 17:52 CST
Initiating Ping Scan at 17:52
Scanning 174.34.243.140 [4 ports]
Completed Ping Scan at 17:52, 0.01s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 17:52
Completed Parallel DNS resolution of 1 host. at 17:52, 0.45s elapsed
Initiating SYN Stealth Scan at 17:52
Scanning unknown.carohosting.net (174.34.243.140) [1000 ports]
Discovered open port 22/tcp on 174.34.243.140
Discovered open port 80/tcp on 174.34.243.140
Completed SYN Stealth Scan at 17:52, 4.47s elapsed (1000 total ports)
Initiating OS detection (try #1) against unknown.carohosting.net (174.34.243.140)
Retrying OS detection (try #2) against unknown.carohosting.net (174.34.243.140)
Nmap scan report for unknown.carohosting.net (174.34.243.140)
Host is up (0.17s latency).
Not shown: 996 closed ports
PORT    STATE    SERVICE
22/tcp  open     ssh
80/tcp  open     http
139/tcp filtered netbios-ssn
445/tcp filtered microsoft-ds
Device type: general purpose|VoIP adapter|load balancer
Running (JUST GUESSING): Linux 2.6.X (89%), Cisco embedded (86%), F5 Networks embedded (85%)
OS CPE: cpe:/o:linux:kernel:2.6 cpe:/a:cisco:unified_communications_manager:7.1.1
Aggressive OS guesses: Linux 2.6.15-28-amd64-server (Ubuntu, x86_64, SMP) (89%), Linux 2.6.18.pi (x86) (88%), Linux 2.6.21-gentoo-r4 (PowerPC) (88%), Linux 2.6.11 - 2.6.18 (87%), Cisco Unified Communications Manager VoIP gateway (86%), F5 BIG-IP load balancer (85%)
No exact OS matches for host (test conditions non-ideal).
Uptime guess: 49.303 days (since Fri Jun 15 10:36:34 2012)
TCP Sequence Prediction: Difficulty=200 (Good luck!)
IP ID Sequence Generation: All zeros

Read data files from: /usr/bin/../share/nmap
OS detection performed. Please report any incorrect results at http://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.62 seconds
           Raw packets sent: 1044 (49.044KB) | Rcvd: 1029 (42.116KB)
```

通过对几十个公开代理IP的测试，Nmap准确识别操作系统的概率还是非常大的，但Nmap每次探测花费的时间有时也比较长，可能要好几秒钟，怎么运用到实际的系统当中还需要进一步研究。 

## 参考
1.<a href="http://en.wikipedia.org/wiki/TCP/IP_stack_fingerprinting" target="_blank">TCP/IP Stack Fingerprinting</a> 
2.<a href="http://www.insecure.in/network_hacking_03.asp" target="_blank">OS Fingerprinting</a>