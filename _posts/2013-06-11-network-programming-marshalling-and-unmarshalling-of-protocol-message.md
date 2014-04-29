---
layout: post
title: "Network Programming - Marshalling and Unmarshalling of Protocol Message"
categories: Programming
tags: Network Memory Alignment Endianess STUN
---

The application layer programming interface of TCP/IP network, aka socket, is designed to send and receive a sequence of byte stored in a block of contiguous memory. That means, the messages exchanged between communication parties must be mapped to/from a memory block (byte buffer). Actually an application layer protocol directly works on TCP/IP transport layer (i.e., TCP or UDP) must specify the memory formats of messages. Let's take STUN (IETF RFC 3489) as an example to see how a network message is defined.    

"STUN is a request-response protocol. Clients send a request, and the server sends a response. All STUN messages consist of a 20 byte header:"

<img src="/images/stun-header.png" />

"After the header are 0 or more attributes. Each attribute is TLV encoded, with a 16 bit type, 16 bit length, and variable value:"

<img src="/images/stun-attribute.png" />

Of course as a complete protocol, STUN specifiction included many other issues, such as the possible values (or formats) of each fields in message header and attributes, how messages are sent and received between client and server, and etc. But those are not our topics today. We will focus on some programming issues related with message itself. 

# Message Object

In object-oriented programming, it is obvious that a object class Message should be defined to encapsulate the data fields and the related operations. A possible class declaration for above STUN messages is given below:

{% highlight cpp %}
class Attribute
{
public:
  // Encode data fields into memory block
	int toBuffer(unsigned char* buffer, int size, int& offset);
  
  // Decode data fields from memory block
	bool fromBuffer(unsigned char* buffer, int size, int& offset);

	// Other Methods to set and get data fields
	
private:
	unsigned short _type;
	unsigned short _length;
}

class Message
{
public:
  // Encode data fields into memory block
	int toBuffer(unsigned char* buffer, int size);
  
  // Decode data fields from memory block
	bool fromBuffer(unsigned char* buffer, int size);
	
	// Other Methods to set and get data fields
	
private:
	unsigned short _type;	// Type of message
	unsigned short _length; // Length of message
	unsigned char _tid[16]; // Transaction ID
	vector<Attribute*> _attributes; // Zero or more attributes
}
{% endhighlight %}
 
With these classes, the applications can manipulate the data fields but not necessary to know the memory details of the message. In other words, the process of encoding and decoding are encapsulated into Message object. The following part of this article will focus on the implementations of message encoding and decoding. 

# Marshalling and Unmarshalling

These are jargon used in object-oriented programming and networking programming. In a few words, "marshalling" refers to the process of encoding the data or the object into a byte sequence, and "unmarshalling" is the reverse process of decoding the byte sequence beack to their original data or object. Accordingly, Message::toBuffer() will do marshalling and Message::fromBuffer() will do unmarshalling. 

Obviously, a straight-forward way to do marshalling and unmarshalling is to read and write the memory field by field, as is shown below:

{% highlight cpp %}
int Message::toBuffer(unsigned char* buffer, int size)
{
  int offset = 0;
  if(size >= 20) // Message header size
  {
    *(unsigned char*)(buffer + offset)  = _type; // Message type
    offset += 2;
    
    *(unsigned char*)(buffer + offset) = _length; // Message length
    offset += 2;
    
    memcpy(buffer + offset, _tid, 16); // Message transaction ID
    offset += 16;
    
    for(Attribute* attrib : _attributes)
    {
      attrib->toBuffer(buffer, size, offset);
    }
  }
  return offset;
}

bool Message::fromBuffer(unsigned char* buffer, int size)
{
  int offset = 0;
  if(size >= 20) // Message header size
  {
    _type = *(unsigned short*)(buffer + offset); // Message type
    offset += 2;
    
    _length = *(unsigned short*)(buffer + offset); // Message length
    offset += 2;
    
    memcpy(_tid, buffer + offset, 16); // Message transaction ID
    offset += 16;
    
    while(offset < size)
    {
      Attribute* attrib = AttributeFactory::fromBuffer(buffer, size, offset);
      if(attrib != NULL)
      {
        _attributes.push_back(attrib);
      }
    }
    assert(offset == size);
    return true;
  }
  return false;
}
{% endhighlight %}

# Mapping between Memory and "struct"

C/C++ provides us unbelievable power to manipulate memory. We can convert from a memory block directly to a "struct" object that has pre-defined data fields, and vice versa. This is another way to complete marshalling and unmarshalling. 

{% highlight cpp %}
struct MESSAGE_HEADER 
{
  unsigned short type;
  unsigned short length;
  unsigned char  tid[16];
};

int Message::toBuffer(unsigned char* buffer, int size)
{
  int offset = 0;
  if(size >= sizeof(MESSAGE_HEADER))
  {
    MESSAGE_HEADER header;
    header.type = _type;
    header.length = _length;
    memcpy(header.tid, _tid, 16);
  
    memcpy(buffer, (void*)&header, sizeof(MESSAGE_HEADER));
    offset += sizeof(MESSAGE_HEADER);
  
    ...
  }
  return offset;  
}

bool Message::fromBuffer(unsigned char* buffer, int size)
{
  int offset = 0;
  if(size >= sizeof(MESSAGE_HEADER))
  {
    MESSAGE_HEADER* header = (MESSAGE_HEADER*)buffer;
    _type = header.type;
    _length = header.length;
    memcpy(_tid, header.tid, 16);
    offset += sizeof(MESSAGE_HEADER);
    
    ...    
  }
  return false;
}
{% endhighlight %} 

In above codes, struct is only used to map memory block into data fields. Actually the struct object can be used as data member of Message class to hold data fields, hence marshalling and unmarshalling can be completed with only one memory copy, as is shown below: 

{% highlight cpp %}
class Message
{
public:
  // Encode data fields into memory block
	int toBuffer(unsigned char* buffer, int size);
  
  // Decode data fields from memory block
	bool fromBuffer(unsigned char* buffer, int size);
	
	// Other Methods to set and get data fields
	
private:
  struct MESSAGE_HEADER 
  {
    unsigned short type;
    unsigned short length;
    unsigned char  tid[16];
  } _header;
  
	vector<Attribute*> _attributes; // Zero or more attributes
}

int Message::toBuffer(unsigned char* buffer, int size)
{
  int offset = 0;
  if(size >= sizeof(MESSAGE_HEADER))
  {
    memcpy(buffer, (void*)&_header, sizeof(MESSAGE_HEADER));
    offset += sizeof(MESSAGE_HEADER);
    
    ...
  }
  return offset;
}

bool Message::fromBuffer(unsigned char* buffer, int size)
{
  int offset = 0;
  if(size >= sizeof(MESSAGE_HEADER))
  {
    memcpy((void*)&_header, buffer, sizeof(MESSAGE_HEADER));
    offset += sizeof(MESSAGE_HEADER);
    
    ...
  }
  return false;
}
{% endhighlight %} 

# Memory Alignment

There is a basic assumption in above solution, that is the the "struct" object has a exact memory structure as defined with protocol message. But this is not always true because of memory alignment. To be specific, the data fields of a "struct" object may be not contiguous in memory, but filled with one or more byte as padding, so that certain data type always located on a particular address border. This kind of "alignment" usually leads to a better performance in terms of CPU's instruction system. 

Fortunately, most C/C++ compilers allow us to reset the "alignment" rules. Below code forces the memory alignment is at single byte border, that means there is no padding always. 

{% highlight cpp %}  
  #pragma pack (1)
  struct MESSAGE_HEADER
  {
    unsigned short type;
    unsigned short length;
    unsigned char  tid[16];
  };
  #pragma pack()
{% endhighlight %}

# Endianess

"STUN messages are encoded using binary fields. All integer fields are carried in network byte order, that is, most significant byte (octet) first. This byte order is commonly known as big-endian." (RFC3489/11. Protocol Details)
   
What is endianess? To be simple, endianess is the lay out of bytes in memory for a multiple bytes value. For example, a unsigned short needs two bytes to present. Accordingly, it takes two bytes in memory. But what is the order to put those two bytes into memory? Which byte is first? Some systems put the least significant byte firstly, which is called "little endian". While some systems put the most significant byte firstly, which is called "big endian". 

Obviously, two parties of communication must use same endianess or know the actual endianess of memory and do necessary conversions when access data in memory. To avoid the necessity of exchanging endianess information, network communication protocols usually specify a particular endianess for data being exchanged. Then each party of communication only need to make necessary conversion during marshalling and unmarshalling of the data fields. 

In above example, we assign (or copy) the values with type "unsigned short" without any endianess conversion. The result is the momory has same endianess with local system, i.e., "little endianess" in my case (on Mac OS X system). This is against the protocol. 

{% highlight cpp %}
int Message::toBuffer(unsigned char* buffer, int size)
{
  int offset = 0;
  if(size >= 20) // Message header size
  {
    *(unsigned char*)(buffer + offset)  = htons(_type); // convert to network endianess
    offset += 2;
    
    *(unsigned char*)(buffer + offset) = htons(_length); // Message length, 
    offset += 2;
    
    memcpy(buffer + offset, _tid, 16); // Message transaction ID
    offset += 16;
    
    ...
  }
  return offset;
}

bool Message::fromBuffer(unsigned char* buffer, int size)
{
  int offset = 0;
  if(size >= 20) // Message header size
  {
    _type = ntohs(*(unsigned short*)(buffer + offset)); // Convert to local endianess
    offset += 2;
    
    _length = ntohs(*(unsigned short*)(buffer + offset)); // Message length
    offset += 2;
    
    memcpy(_tid, buffer + offset, 16); // Message transaction ID
    offset += 16;
    
    ...
  }
  return false;
}
{% endhighlight %}