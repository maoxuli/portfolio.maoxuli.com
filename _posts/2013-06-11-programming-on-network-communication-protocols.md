---
layout: post
title: Programming on network communication protocols
categories: Programming
tags: Network
---

A network communication protocol is specified, what is the next step? Programming! Yes, we discuss the steps from a network communication protocol to the codes that let it working. Specifically, our discuss here is based on lower level network programming interface, i.e., socket, rather than a RPC-style network programming interface. In addition, we only discuss "binary format" protocols here, i.e., the messages are encoded using binary fields. We will discuss "text format" protocols in another post. 

## How a network communication protocol is specified?

Let's take STUN (IETF RFC 3489) as an example.    

"STUN is a request-response protocol. Clients send a request, and the server sends a response. All STUN messages consist of a 20 byte header:"

<img src="/images/stun-header.png" />

"After the header are 0 or more attributes. Each attribute is TLV encoded, with a 16 bit type, 16 bit length, and variable value:"

<img src="/images/stun-attribute.png" />

Further, STUN specified the possible values (or formats) of each fields in message header and attributes.

Ignoring how messages are sent and received, this is the specification of STUN protocol. Yes, that's so simple and easy to understand. 

## From data block to object and vice versa.

The firs step of object-oriented programming is object modeling. Same here. The messages of the protocol is specified with the fields of data block (memory block). We need to abstract and encapsulate the messages into some kinds of objects (data structures, data types, or classes).  

Straight forward, we map the fields of messages into member variables of a object. The mapping is flexible to some extent. 
The object is used to denote and manipulate the fields of messages, rather than transfer over network directly. To send over network, a message need to be packed into a data block (memory black) with the exact format specified in the protocol. On the other hand, once a data block (that contains a message) is received, it needs to be unpacked into a object. 

Accordingly, a scaffold of STUN Message class is defined (in C++) as below: 

{% highlight cpp %}
class Message
{
public:
	size_t pack(unsigned char* buffer, size_t size);
	bool unpack(unsigned char* buffer, size_t size);
	
	// Other methods to set and get fields
	
private:
	unsigned short _type;	// Type of message
	unsigned short _length; // Length of message
	unsigned char _tid[16]; // Transaction ID
	vector<Attribute*> _attributes; // Zero or more attributes
}
{% endhighlight %}

Further, We define a class for nested attributes. 

{% highlight cpp %}
class Attribute
{
public:
	size_t pack(unsigned char* buffer, size_t size);
	bool unpack(unsigned char* buffer, size_t size);

	// Other methods to set and get fields
	
private:
	unsigned short _type;
	unsigned short _length;
}
{% endhighlight %}

Here we did not assign value of the attribute in the class because different types of attributes have different value formats, which can be handled with sub-classes typically. 

## Packing and unpacking?

 

## Alignment


## Endianess

Endianess is very important question in network communication but often neglected by programmers. The reason is endianess is not a issue at all when communication between systems with same endianess. 

To be simple, endianess is the lay out of bytes in memory of a multiple bytes value. For example, a unsigned short needs two bytes to present. Accordingly, it takes two bytes in memory. But what is the order to put those two bytes in memory? Which byte is first? Some system put the least significant byte firstly. It is called "little endian". Similarly, "big endian" put the most significant byte firstly. 

Obviously, the two parties of communication must use same endianess or known the actual endianess of memory and do necessary conversions when access the data in memory. To avoid the necessity of exchanging endianess information, the network communication protocols usually specify a particular endianess for data being exchanged. Then each party of communication only need to make necessary conversion when packing and unpacking the message. 

Actually, STUN specified its endianess also. "All integer fields are carried in network byte order, that is, most significant byte (octet) first. This byte order is commonly known as big-endian." 
