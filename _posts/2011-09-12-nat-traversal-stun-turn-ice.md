---
layout: post
title: "NAT traversal with STUN, TURN, and ICE"
categories: Programming
tags: NAT STUN
close: 2011-10-01
---

<p>NAT (Network Address Translation) is a protocol to map multiple private hosts to one publicly exposed IP address. Typically packets passing from the private network to the public network will have their source address modified while packets passing from the public network back to the private network will have their destination address modified. The NAT address mapping and filtering behaviors prevent a local host from being connected directly by a host on public network, while that is necessary for peer-to-peer networking. A series of IETF specifications are proposed to resolve the connection issues raised from NAT. You may find the related knowledge in this blog <a href="#">post</a>.
	
<p>STUN (IETF RFC 3489/5389) is a protocol that allows two hosts to discover the presence and types of a NAT between them, and to find out what external IP address and port are to be replaced by the NAT. A work flow for NAT discovery is defined in these specifications. Based on address mapping and filtering behaviors of NAT, if exist, local address is classified into various types, which indicates the reachability of local host in peer-to-peer networking. Please find details in this blog <a href="#">post</a>.</p>

<img src="/images/stunclient.png" />

<h1>Demo Program</h1>
<p>This program is essentially a STUN client for NAT discovery. It is a command line tool running on Windows, Mac OS X, and Linux. The underlying well-designed C++ classes can be used in your applications to parse STUN messages. By default, the program connects to a stun server, does the test sequence of NAT discovery, and reports the (NAT) type of your host address. You may find a copy of source code at <a href="https://github.com/limlabs/stunclient" target="_blank">Github</a>. Below is a screenshot of the running program.</p>
	
<p><img src="/images/screenshot.png" /></p>

<h1>FAQ</h1>
<p>Q: What type of server does the program works with?</p>
<p>A: The program works with a STUN server. It is currently compliant with IETF RFC 3489. The following version will support RFC 5389.</p>
<p>Q: It displayed "Permission denied" when I ran the program. How to resolve it?</p>
<p>A: This may occur when you downloaded pre-compiled binary for Mac OS X or Linux system. The reason is the file you downloaded lack of "execution" right. Please run "chmod 777 [filename]" first.</p>
<p>Q: Why did it report that I was behind a restricted cone NAT when I first ran the test, but reported a full cone NAT in the following tests?</p>
<p>A: You should be behind a restricted cone NAT, so in general, as in the first test, you can not receive a response from a changed IP and port. But then ppconfig send a request to the changed IP and port to determine the type of NAT. A hole is opened to the changed IP and port at this time and it will keep open for a period (typically 30 seconds). So in the following test, you can receive a response from a changed IP and port, and it seems that you are behind a full cone NAT. This issue occurred because STUN server reuses a same changed address for different tests. A workaround is doing second test after a long enough period (e.g. 30 seconds or longer).</p>
