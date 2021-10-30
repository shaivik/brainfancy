# OSI Model 

- OSI is the acronym for Open Systems Interconnected. It is a reference model that describes how information from a software application in one computer moves through a physical medium to the software application in another computer.

- It consists of 7 logical layers to deal with network packet (each layer has its own responsibility which is independent from others).

# Layers of OSI Model

## Layer 1: Physical layer 

Transferring of bits as electric signals via wires or radio waves via air or light wave via optical cables.

----

## Layer 2: Data Link Layer

Deals with MAC addresses, source and destination. Point to note here is that MAC addresses find relevance in the internal network only in the same subnet. ARP (Address Resolution Protocol) is used to find the MAC addresses of destination host. The IP Packet + Mac Addresses block of information is called a Frame.

----

## Layer 3: Network Layer

Deals with IPs of source and destination. Adds/Removes IPs as headers to the segment called IP Packet.

----

## Layer 4: Transport Layer

Deals with application ports of source and destination. Adds/Removes port headers to data and is called Segment. It is also concerned with packets ordering from sender to receiver, acknowledgements, congestion control in case TCP is used. Checksum functionality which ensures packets arriving to the receiver is intact is also responsibility of this layer.

----

## Layer 5: Session Layer

This is responsible for maintaining sessions but not used usually. We generally push session maintenance resposibility to the transport layer. 

----
## Layer 6: Presentation Layer

Used for encryption/decryption and serialisation/deserialisation of data. Practical usage not seen usually as we usually defer its job to the application layer.

---
## Layer 7: Application Layer

The data which we want to send/receive at application level. There are various protocols depending upon the application being used.

----

## About the data flow

The flow starts from sender from layer 7 upto layer 1. At the receiver reverse flow is there and it can go up to layer 7 depending upon the current receiver in the flow of data.