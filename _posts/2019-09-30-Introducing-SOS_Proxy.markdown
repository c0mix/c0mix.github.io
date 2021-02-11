---
title:  "Introducing SOS_Proxy"
date:   2019-09-30 15:04:23
categories: [PenTest]
tags: [IoT,PenTest,Proxy,Burp,HTTP,Python]
---
_In this post I will introduce a simple but effective tool that I developed in order to automate the invisible proxy technique and manage to intercept the HTTP traffic of any non proxy-aware device._


## Invisible Proxy Technique in bites

Is it possible to proxy a device that does not support this functionality? Sometimes, under particular circumstances, **YES**. You can do that with [Burp Suite][burp] and the invisible proxying technique explained in [Portswigger's article][invisible]. Following a brief summary of the steps that must be implemented in order to correctly intercept the HTTP traffic.

1. Create a separate virtual network interface for each destination host.
2. Create a separate Proxy listener for each interface (or two listeners if HTTP and HTTPS are both in use).
3. Using your hosts file, redirect each destination hostname to a different network interface (i.e., to a different listener).
4. Configure Burp’s listener on each interface to redirect all traffic to the IP address of the host whose traffic was redirected to it.

## SOS_Proxy
In order to automate the setup of multiple invisible proxies with Burp I've developed a simple Python script called SOS_Proxy. The main features of this tool and a simple step-by-step demo can be found insite the following [presentation][slide]. Otherwise, if you just want to check the source code, you can find it on [github][github]!

<iframe src="//www.slideshare.net/slideshow/embed_code/key/D7vbsbK5ku15PN" width="595" height="485" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe><br/>


Morover, the following video will show a real use case against an Android device.
<div><iframe width="560" height="315" src="https://www.youtube.com/embed/R9VAWpXcXAw" frameborder="0" allow="accelerometer; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></div>
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


## ESC Talk Record (in Italian)
Last summer i attended the [End Summer Camp (ESC)][esc] security conference in Venice (Italy) as a speaker in order to present this simple but useful tool. Below you can find the video recording of that talk.
<div class="video-container"><iframe width="560" height="315" src="https://www.youtube.com/embed/OP1gJEn1gSY" frameborder="0" allow="accelerometer; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></div>
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


[burp]: https://portswigger.net/burp
[invisible]: https://portswigger.net/burp/documentation/desktop/tools/proxy/options/invisible
[esc]: https://www.endsummercamp.org/index.php/End_Summer_Camp
[slide]: https://www.slideshare.net/LorenzoComi1/sosproxy-invisible-proxy-automation
[github]: https://github.com/c0mix/SOS_Proxy