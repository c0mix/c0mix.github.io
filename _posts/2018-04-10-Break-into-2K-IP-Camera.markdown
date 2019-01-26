---
title:  "Break into 2K IP-Camera"
date:   2018-04-10 15:04:23
categories: [IoT]
tags: [IoT,PenTest,CVE-2017–17101]
---
_In this post I will explain technical details of a vulnerability that I found on several IP-Camera models during a IoT-PenTest session and marked as CVE-2017–17101. This vulnerability allows an unauthenticated remote user to bypass the login panel and access to all the device features._

## General Information
It all started with a Penetration Test against a single device, one of the many medium price IoT IP-Cameras that allows a consumer to watch whatever is happening inside his home from anywhere ([here][amazon] some examples).
Only when I found an effective way to exploit this vulnerability I realized the real impact of it: according to [Shodan][shod], 2K Internet-exposed devices were afflicted and could be exploited by anyone. For this reason some parts of the PoC's details are redacted.

## Setup an IoT-Testing Lab
First of all, the Lab setup! An efficient configuration should always look like as follows:

![lab](/images/2018-04-10-Break-into-2K-IP-Camera/lab.png)

Long story short: an Android **Testing Smartphone** with IP-Camera's app installed and the targeted **Ip-Camera** device both configured to join a **Fake Access Point**. Moreover my laptop used as connection provider equipped with **Burp Suite** and **Wireshark** to cover and intercept every possible communication channel.

## Start to break the "Thing"
After finishing the classical network and information discovery phases of a PenTest, I focused my attention on the main door: the device web application that handles all device settings and information, exactly that web interface which someone exposes without any security concern on the public Internet, even without changing the default credentials!

### Discovery
So let's fire up Burp and walk the "happy path" looking for some interesting features to abuse… Just some minutes later, after the submission of login form and the navigation of settings menu, I found what I was looking for: Download and Upload functionality or **configuration backup** and **configuration restore** functionalities, to be precise!

With the below request, a logged user (admin:suadmin) is allowed to download a text file containing all the configuration parameters of the IP-Camera (in the next section I will provide more details about this file).

![download](/images/2018-04-10-Break-into-2K-IP-Camera/down.png)
<p style="text-align: center;">CGI that manage the configuration's backup (download)</p>

As you can see from the following screenshot, the upload request allows anyone, without providing any type of authentication or credentials or session cookies, to upload a configuration file to the webcam. Yes, you read it right: **it is possible to upload whatever you want without being authenticated** and the file that you will upload, if in right format, will overwrite the actual configuration of the IP-Camera, modifying the settings parameters.

![upload](/images/2018-04-10-Break-into-2K-IP-Camera/restore1.png)
<p style="text-align: center;">CGI that manage the configuration's restore (upload)</p>

That's cool. However, what can I upload to the Camera without causing DoS or breaking some internal stuff or without knowing the original configuration file?

### Exploiting
The answer is in the configuration file itself!

![config](/images/2018-04-10-Break-into-2K-IP-Camera/config_file.png)

As you can see from this snippet extracted from the configuration file, at a certain point there is also a section which handles the credentials and the privileges of each user.
In this case the idea was to upload only this little part of the configuration file in order to **overwrite only the first user's credentials**, avoiding a DoS and gathering an easy access from the login panel to the rest of the Camera menu. Final PoC looks like this:

![exploit](/images/2018-04-10-Break-into-2K-IP-Camera/finalepl1.png)

Wrapping all in a python script that takes as input IP & Port of a target + arbitrary Username & Password that the attacker wants to set, an authentication bypass exploit is ready… As you can see from the following video.

<div class="video-container"><iframe width="854" height="480" src="https://www.youtube.com/embed/B75C13Zw35Y" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe></div>
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

## Evaluation of the impact
Thanks to this exploit an **unauthenticated user is able to bypass the login screen and access the webcam contents** including: live video stream, configuration files with all the passwords, system information, and much more.

![shodan](/images/2018-04-10-Break-into-2K-IP-Camera/Shodan1.png)

As already mentioned before, with a specific query on Shodan it is possible to list about 2.000 vulnerable devices facing to the Internet.

## Final Consideration
After a responsible disclosure of this issue to many vendors, [CVE-2017–17101][cve] was assigned by Mitre at this vulnerability.
Please **take your devices up to date** with the latest firmware upgrade and always change default credentials!
I hope you liked this article, see you soon…

[cve]:      https://www.cvedetails.com/cve/CVE-2017-17101/
[shod]:     https://www.shodan.io/
[amazon]:   https://www.amazon.com/s/ref=nb_sb_noss?url=search-alias%3Daps&field-keywords=ip+camera
