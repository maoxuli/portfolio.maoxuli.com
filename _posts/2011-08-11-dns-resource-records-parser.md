---
layout: post
title: Network Programming - DNS Resource Records Parser
categories: Programming
tags: Network DNS
---

This article is not about programming on DNS protocol. The specific topic is parsing DNS Resource Records. Resource Records is the form that DNS servers store domain associated information. DNS server responds to a DNS query with a sequence of Resource Records. In this article, we design a Resource Records parser in a objecct-oriented fashion. 

# DNS Message and Answer Section

The format of DNS  message is shown as below: 

<img src="/images/dns_message_format_1.jpg" width="400px"/>

Usually, a query message consists of Header section and Question section, while a response message may contain all sections if necessary. DNS server returns Resource Records in a Answer section. A Answer section consists of a sequence of Resource Records. Each Resource Record contains the fields of NAME, TYPE, CLASS, TTL, RDLENTH and RDATA. The length of RDATA is given by RDLENTH. The content and format of RDATA depends on the TYPE of the Resource Record. Below is the formats of Answer section and a piece of inner Resource Record.

<img src="/images/dns_message_format_4.jpg" width="400px" />

<ul>
<li>OCTET 1,2,..n NAME</li>
<li>OCTET n+1,n+2 TYPE</li>
<li>OCTET n+3,n+4 CLASS</li>
<li>OCTET n+5,n+6,n+7,n+8 TTL</li>
<li>OCTET n+9,n+10 RDLENGTH</li>
<li>OCTET n+11,n+12,….. RDATA</li>
</ul>

# Domain Name Compression

To this point, we know that there are some fields in DNS message contains domain name. For example, the QNAME field in each piece of question in Question section, and the NAME field in each piece of Resource Record in Answer section. In addition, the RDATA field in Resource Record may also contain domain names. In order to reduce the size of message, DNS protocols utilizes a compression scheme for domain names, which will eliminate the repetition of domain names in the fields of QNAME, NAME, and RDATA. It is worth to mention that name compression is not mandatary in DNS message. In general, when we construct a request message, we would not like to compress domain names for simpleness. But DNS server usually compress domain names when construct a response message, so that we still need to handle compressed domain names in response message. 

With DNS domain name compression scheme, a domain name in a message can be represented with one of below three forms:

(1) A sequence of "count + characters" and ending in a ‘\0’ character.

Two examples:

<img src="/images/dns_name_compress_1.jpg" width="500px" />

The number in the green box is the count of the following characters. Each character and the ending zero are one byte width. The count is also one byte width, but the high two bits are always set to ‘00’, hence the range of count is 0-63. Each section of "count + characters" corresponds to a dot-divided part of domain name. That means, the maximum length of dot-divided part in domain name is 63 in DNS message.

(2) A pointer to another place that contains the name.

The pointer consists of two bytes, and the high two bits of the first byte are always set to ‘11’. The value of the pointer (exclude the high two bits) specifies an offset from the start of the packet (i.e., a zero offset specifies the first byte of the ID field).

The target position may be the start of another name string, or the middle of another name string.

<img src="/images/dns_name_compress_2.jpg" width="500px" />

(3) A sequence of "count + characters" and ending with a pointer.

In this situation, the first part of the name is given with the form 1 (i.e., a sequence of "count + characters"), while the last part is given with the form 2 (i.e., a pointer).

<img src="/images/dns_name_compress_3.jpg" width="500px" />

In addition, the pointer can be embedded. The decoding need to be implemented recursively.

<img src="/images/dns_name_compress_4.jpg" width="500px" />

Another rule is only one pointer is permitted in a name field, and the pointer must be the end of the field (followed by a '\0').

# Base Class and Factory 

We define a wrapper class here for a piece of Resource Record in Answer Section, rather than the Answer section itself. Design patterns of Strategy and Factory is used here for a extendable architecture to handle different types of Resource Records. A base class ResourceRecord handle the common parts of Resource Records and derived classes will handle RDATA according to the actual type. A class RRFactory works as a object factory to instantiate ResourceRecord object with given type. If there is no a derived class of ResourceRecord is defined for a given type, base class is instantiated. With this design, it is rather easy to support new types of Resource Records. What we need to do is define a new derived class of ResourceRecord and add a entry in RRFactory. We need to parse RDATA field of the Resource Record in derived class.

Please find a copy code at <a href="https://github.com/limlabs/dnsresolver">Github</a>. 

