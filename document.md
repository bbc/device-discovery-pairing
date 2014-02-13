# Discovery and Pairing Literature Review for MediaScape
## Libby Miller, Chris Needham

*Thanks to Matt Hammond for valuable comments and corrections.*

This is an initial literature review, starting from the state of the art as described in the MediaScape description of work, and covering, briefly, discovery techniques used by UPnP, MDNS, and various applications that use them, such as Chromecast, Airplay and more experimental techniques. It also looks at techniques used in NFC and Bluetooth, and various approaches for out-of-band linking of devices. It touches on various W3C and other specifications which are taking some of this work to the Web.

The goal is to provide a short overview of some of the previous work in this area, which we can use to focus our efforts.

Discovery and pairing are closely connected, since the purpose of discovery is to find a service, device or application to connect or pair to. Pairing itself then has particular characteristics, discussed later. I also provide brief summaries and links for various technologies I've looked at in this area. [a]

## Discovery / pairing Overview

We've identified three different models of discovery / pairing:

### 1. Enabling services to find each other on a given local network

UPnP and mDNS are two common techniques[b][c] for discovering media services on a local network. This process is typically characterised as discovering services rather than devices: devices may contain one or more services. In both these cases services on a network make themselves findable, and other services can find them with broadcast queries, following up with more specific queries to a particular service, and the possibility of controlling or getting information from them. "Pairing" is simply a matter of communicating.

Key features are:

* finding services of the right kind and
* finding more information about the services, including how to control them, and
* services making sure interested parties know about their state

Typical issues are:

* identity (ensuring items have identifiers and there's no collision between their identifiers, when they are coming online and offline frequently)
* persistence / cache control (are the devices still there? how do we tell when they have gone, and how quickly do we know and need to know?)
* matching and interoperability (you'll be searching for a specific type of service so we end up with registries of prefixes; once you know about them you need to find out more and then control them so you need to know how to do that - often / usually externally defined APIs are used for that)
* security - typically the LAN will be a trusted environment and as soon as you step out of the LAN, security problems emerge (c.f. UPnP-enabled routers)
* Some LANs (e.g. in public settings, such as hotels) are configured to prevent direct peer to peer routing of IP packets, thereby preventing local discovery
* Some LANs require you to authenticate yourself via a web page before granting the device access to other than the gateway, this can’t be done for devices that don’t support a web browser

### 2. Linking devices, assuming they are connected to the internet[d][e]

A different characterisation of discovery / pairing is in the context of devices on different networks connected by the internet. At its simplest level this is about placing an entry in an internet connected remote database[f] linking the identifiers for two services or devices, and then allowing them to communicate using this remote mechanism.[g] Examples are HBBTV second screen framework from FI content, and some kinds of synchronised second screen applications such as those developed in NoTube and P2PNext. The "discovery" part is the out of band means of passing identifiers between the devices.

Key features are:

* connecting devices on separate networks
* wider pool of potential implementers

Typical issues are:

* requirement for an external internet connection (my TV isn't working because my internet is broken!)
* out of band linking mechanism (there needs to be some mechanism such that one or both devices have a reference to the identifier for the other; typically this will be user-initiated, like scanning a QR code or typing in a numeric code or clicking a link)
* security (there's not even the implicit security we assumed from a LAN, so the linking mechanism needs to be trustable)
* starting point interoperability (where on the internet do the devices look for each other?)
* multiplication of services (every brand has its own cloud-based service leading to a poor user experience)
* identity and persistence may also be important (where some networks are unreliable objects may frequently go on and offline; however there are internet-based mechanisms for identity which there may not be on a LAN)
* timing issues over multiple networks (including broadcast)

### 3. Enabling devices in close physical proximity to find each other and initiate communication

A final sense of discovery and pairing is where devices identify other devices of a similar type using electromagnetic waves or signalling on a specific frequency, and use this to create or bootstrap a piconet, or small ad-hoc network. NFC (Near Field Communication, an extension of RFID) and Bluetooth work like this. 

Key features are:

* connecting powerless or low-power devices
* physical proximity requirement for operation

Such a piconet can be used to bootstrap a wider, web-based communication mechanism. Because NFC can be such short range, it is very selective and requires user intention to bring the objects into proximity. It can therefore be used as the means to determine permission / intent from the user to establish pairing before communication then begins over a longer range medium (e.g. wifi, bluetooth, internet)

## Pairing

"Discovery", then, is a something of a catch-all term that means something very general like "finding the device or service you want to talk to". Pairing generally has a more specific meaning. In the MediaScape description of work, we define it as "The process of establishing a secure communications channel between two unassociated devices typically over a wired network connection or over wireless (e.g. infrared, Bluetooth, 802.11, NFC, Wi-Fi Direct)."

Pairing has three basic dimensions:

### 1. Participants

Pairing can be between devices or between devices and web services.

Pairing between devices and web services is something that we'll look at in detail in Task 3.3, Multi-device connection, but there are overlaps with the pairing discussion more generally, and it's covered briefly below under RadioDNS and CPA.

Pairing devices is really pairing of services and control points which can both be on devices, which UPnP below makes explicit.

### 2. Communications transport

Pairing uses a network for communication between the paired items. This could be an existing network that they are both already connected to (LAN or internet), or an induced network  (NFC, Bluetooth). It may include elements of out of band communication, via human-mediated data input, or audio or optical communication. The out of band communication may be necessary for security purposes.

### 3. Protocol

This involves a series of communications between the devices to ascertain identity, roles, and capabilities, which will be dependent on the flexibility of the system and a trade-off between the amount of user input required and the desired level of security. For LAN-based systems, security is less important than for internet based systems.

## Notes

So far we have looked at:

* UPnP
* DNS-SD/mDNS
* Web intents
* Javascript discovery API
* Webinos
* QR codes / MHEG (internal R&D paper)
* IRT second screen framework for HbbTV (from the FI-Content project)
* RadioDNS / RadioTAG
* EBU Cross-Platform Authentication
* Google's AnyMote Protocol
* DIAL
* Chromecast
* Webrtc
* Audio and flashing mechanisms for connecting devices
* NFC
* Bluetooth
* XMPP

We haven't gone into a great deal of depth for any of these. Hopefully the summary and links to further content provide a starting point for further research / implementation.

### UPnP

> "Universal Plug and Play (UPnP) enables discovery and control of devices, including networked devices and services, such as network-attached printers, Internet gateways, and consumer electronics equipment"

UPnP discovery uses "SSDP" (Simple Service Discovery Protocol), which is roughly outlined here.

Devices have services. When these join the network, they first discover or assign themselves an IP address, and then must send some multicast discovery messages, usually on a specific port, advertising themselves and containing their type, identifier, a pointer to more information and optionally current state information.

Some devices are control points, which can find and control services. Control points on joining may send multicast search messages, to which matching services must respond.  Control points can learn of services via their discovery message or via search queries.

When services leave they should send a series of messages saying their services are unavailable. If their IP address changes they should revoke and rebroadcast. While on the network they should send periodic "I'm alive" discovery messages over multicast UDP. When services change state they can send updates via TCP IP to subscribers or via UDP as multicast to everything on the network. The state descriptions are XML.

SSDP uses part of HTTP 1.1 over UDP; then to gather information about a device, the control point uses HTTP GET over TCP, and the devices responds in kind. Thereafter, control of services is using SOAP and XML. Devices can have multiple control points.

A device will provide a “service description” xml document that lists all the state variables and actions for a given service. This therefore enables an application with no understanding of a service to construct a rudimentary user interface direct onto those state variables and actions.

It is assumed that communications on the local network are secure. The main issue with security and hence the need for authentication, comes from access outside the local network, e.g. NAT traversal.

### References: 

[http://www.upnp.org/specs/arch/UPnP-arch-DeviceArchitecture-v1.1.pdf](http://www.upnp.org/specs/arch/UPnP-arch-DeviceArchitecture-v1.1.pdf)
[http://en.wikipedia.org/wiki/Universal_Plug_and_Play](http://en.wikipedia.org/wiki/Universal_Plug_and_Play)
[http://www.upnp.org/download/UPNP_understandingUPNP.doc](http://www.upnp.org/download/UPNP_understandingUPNP.doc)

## DNS-SD/mDNS 

This is sometimes known as Zeroconf service discovery, and is the basis of Apple’s Airplay discovery mechanism.

DNS-SD allows you to create DNS records that advertise services. The DNS records can be either within a network or visible more widely.

If it is within a network, any device that implements enough of the DNS protocol can advertise itself. Clients can then query the DNS from within that domain as usual, for specific named services (e.g. "daap" for iTunes).

Otherwise if you are the owner of a domain you can create PTR and TXT records for it for the whole domain accessible outside it. (This is the approach used by RadioTAG (and TVTag) for associating records for hybrid services.)

For DNS-SD/mDNS (i.e. used with multicast) there is no client config required; unicast will usually require some.

There are three design goals:
* query services in a domain and receive results;
* resolve for more information;
* reasonable persistence

For discovery ("Service Instance Enumeration"), clients on the network do a PTR (pointer) query of DNS and get a list of instances of services of that kind on that domain, akin to a directory listing:

    Service Instance Name = <Instance> . <Service> . <Domain>

There are also a number of recommended general queries the client should make when it arrives on the network for registering and browsing services.

When a client needs to contact a previously discovered service, it queries for the SRV and TXT records of its Service Instance Name. The SRV record for a service gives the port number and target host name where the service may be found. The TXT record gives additional information about the service.

Setting DNS records can be delegated to a special kind of device, or services can just implement some of the DNS protocol themselves.

The key difference to point out between UPnP and Zeroconf is that Zeroconf does not define how services are to be defined and accessed, but UPnP does.

### Reference

[http://tools.ietf.org/html/rfc6763](http://tools.ietf.org/html/rfc6763)


## W3C Web intents

Currently discontinued: “The Web Intents Addendum defines how Web Intent may be used to discover services on a local network. Work on this specification has been discontinued since the Web Intent specification has been published as a W3C Working Group Note indicating no further development on the Recommendation track at this time.”

### Reference: 

[http://www.w3.org/TR/webintents-local-services/](http://www.w3.org/TR/webintents-local-services/)
[http://lists.w3.org/Archives/Public/public-device-apis/2013Mar/0023.html](http://lists.w3.org/Archives/Public/public-device-apis/2013Mar/0023.html)

## JavaScript discovery API

This is a work in progress, defining an interface for javascript to access UPnP, Zeroconf or DIAL from within javascript in a webpage.

The idea is that the web page can get access to local services using a number of well-used protocols: these provide promise-based access to APIs for control and feedback to and from locally networked devices. It just covers the discovery part - once the services and devices are discovered, you use whatever APIs those specific services support to communicate with them (so for example, UPnP mandates SOAP, while other types of services may use REST).


Example of requesting either a DNS-SD or UPnP advertised service from the reference:


    function showServices( services ) {
      // Show a list of all the services provided to the web page (+ service type)
      for(var i = 0, l = services.length; i < l; i++)
        console.log( services[i].name + '(' + services[i].type + ')' );
    }

 
    navigator.getNetworkServices([
      'zeroconf:_boxee-jsonrpc._tcp',
      'upnp:urn:schemas-upnp-org:service:ContentDirectory:1'
    ]).then(showServices);


It notes some security considerations:

"User authorization, where the user connects the web page to discovered services, is expected before the web page is able to interact with any Local-networked Services."

Security and privacy:
*  express, revocable permission
*  one api element at a time
*  or a prearranged relationship

It contains some useful worked-through examples as Javascript.

### Reference

[http://www.w3.org/TR/discovery-api/](http://www.w3.org/TR/discovery-api/) (Working Draft)

## Webinos

> "Webinos is an EU funded project aiming to deliver a platform for web applications across mobile, PC, home media (TV) and in-car devices."

The most interesting aspect I found was the architecture document, which is worth [http://www.webinos.org/content/html/D033/Architecture_Overview.htm](reading in full). 

It has the concept of a 'personal zone': 

> "Devices in an NFC area, a personal area network (PAN) or a local area network (LAN) are regarded local, no matter what kind of bearers they are using.

> The devices physically in the same local area may belong to different users and therefore logically be in different Personal Zones. That means the connection may be established across different Personal Zones and users can share services with each other."

Devices authenticate to this personal zone, while users authenticate to a device:

> "Single sign-on, where the user is authenticated to a device and applications, and the device is authenticated to the Personal Zone. This avoids the need for establishing direct peering relationships between each pair of devices. It also allows for stronger authentication with the services the user uses. The architecture also allows for situations where the user is offline, e.g. when the user is away from home and currently unable to access the Internet."

All devices have a shared model of the context and can synchronise:

> "This covers users, device capabilities and properties, and the environment. It enables applications to dynamically adapt to changes, and to increase usability by exploiting the context."

> "Synchronisation across the devices in the zone. This includes support for distributed authentication, as well as personal preferences, and replication of service-specific data, e.g. social contacts, and appointments. Synchronisation is essential for supporting offline usage."

It encompasses discovery and access to services.

> "This includes local discovery, e.g. of services exposed by the user's devices, whether connected through Wi-Fi, Bluetooth, or USB, as well as remote discovery for services exposed in the Cloud. The high level discovery API allows Web developers to search for all local services, or to filter by service type and context, or even to locate a named service instance. Remote discovery is based upon existing user names and Email addresses, resolving to a URI for a Personal Zone.

Local proximity:

> "One of the features webinos provides is a unified local connection service based on physical proximity, by making the different interconnect technologies transparent."

### Reference

[http://www.webinos.org/content/html/D033/Architecture_Overview.htm](http://www.webinos.org/content/html/D033/Architecture_Overview.htm)

## IRT Second Screen Framework for HbbTV (FI-Content)

This is a mechanism to create a persistent connection between a second screen and an HbbTV device using a cloud-based service.

The TV-based teletext overlay has a button which can be pressed to show a QR code.
* the ids for the devices are paired on a remote web service
* the teletext app can then send launch an app on the mobile device
* that app can then be used to browse teletext pages on the TV and on the app simultaneously, or hide itself so it's only on the tablet; the tablet can then control the version on the TV (via websockets?)
* the same framework can also be used with an HbbTV VOD service

There doesn't appear to be any link with the TV programmes or channels.

### References

[http://wiki.mediafi.org/doku.php/ficontent.socialtv.enabler.secondscreenframework](http://wiki.mediafi.org/doku.php/ficontent.socialtv.enabler.secondscreenframework)
[http://www.youtube.com/watch?v=pAS5jgXlQnM](http://www.youtube.com/watch?v=pAS5jgXlQnM)

## RadioDNS / RadioTAG


RadioDNS is a way of using the DNS system to identify radio stations. This means a DNS lookup using technical parameters of DAB and FM radio stations can be used to find a URL containing further information about other services that might be provided for the station, such as RadioVIS or RadioTAG.

RadioTAG is a pairing mechanism for an online account and a device (a radio). It uses two part authentication

* device gets a token from website
* the user registers with the website and input token
* they get a pin back and input that into device

### References: 

[http://radiodns.org/documentation/rdns01-1-0-0/](http://radiodns.org/documentation/rdns01-1-0-0/)
[http://radiotag.prototyping.bbc.co.uk/docs/radiotag-api-proposal-v1.00.html](http://radiotag.prototyping.bbc.co.uk/docs/radiotag-api-proposal-v1.00.htm)
[http://tools.ietf.org/html/draft-recordon-oauth-v2-device-00](http://tools.ietf.org/html/draft-recordon-oauth-v2-device-00)

## Cross-Platform Authentication

This is the Cross-Platform Authentication specification currently being developed by the EBU Project Group TVP-CPA.

This describes only part of the specification, which is currently a work in progress. This part is about authorising a device to access a web service, using the passing of a code between devices by an end user.

The goal is for a device such as a radio to be paired with an online account. In some ways it's similar to RadioTAG but with fewer user interaction steps, simplifying things for the user.

I include it here because although it's a slightly different aspect of pairing which will be covered elsewhere in the project, it shows a secure way to pair devices.

The basic flow is:
* the device requests a token
* gets back a device code, a user code and a verification url
* the device displays the user code to the user
* the end user uses the url provided, signs in, and inputs the user code
* the client polls the provider with the device code
* if successful (i.e. the user has logged in and input the user code correctly) it will eventually get a token back which it can use to access the provider

### References

[https://tech.ebu.ch/groups/CPA](https://tech.ebu.ch/groups/CPA)
[https://github.com/ebu/cpa-spec/blob/draft-1.0/specification.md](https://github.com/ebu/cpa-spec/blob/draft-1.0/specification.md) (private github repo)

## Google TV Pairing Protocol

> "The Google TV Pairing Protocol can be used to conduct pairing sessions between clients and servers on a local network, for example between a mobile phone and Google TV. In the pairing process, the client contacts the server, and the server issues a challenge for the client to complete. This usually involves the server showing a code of some kind, and the client echoing that code back to the server. For example, in the case of Google TV, the server might show an alphanumeric code for the user to enter on the client device's keyboard.

> To establish a proper challenge, the client must send the types of challenges it can accept to the server. For example, if the client has a Qwerty keyboard but no camera, it can respond to alphanumeric challenges but not to bar code challenges. When the server receives the types of supported challenges, it creates the challenge and asks the client to respond."

Essentially: there's an exchange of capabilities, then a secret is sent out of band and confirmed.

### Reference

[https://developers.google.com/tv/remote/docs/pairing?csw=1](https://developers.google.com/tv/remote/docs/pairing?csw=1)

## Anymote Device discovery phase

Uses mDNS on the local network, with a known prefix

> "The remote applications establish a pairing session with Pairing service following the Pairing Protocol. The Pairing service runs on the next consecutive port of the Anymote service. For example, If Google TV Anymote service announces itself at port 9551 then the Pairing service listens on 9552. The client should connect to the next consecutive port of Anymote service to set up the pairing session."

### References

[https://developers.google.com/tv/remote/?csw=1](https://developers.google.com/tv/remote/?csw=1)
[https://developers.google.com/tv/remote/docs/anymote](https://developers.google.com/tv/remote/docs/anymote)

## DIAL

> "DIAL (for DIscovery And Launch) is a protocol that 2nd screen (e.g., tablet, phone) devices can use to discover and launch apps on 1st screen (e.g., TV, set-top box, Blu-ray player) devices. It was originally developed by Netflix.

> DIAL maintains a centralized registry for (1st screen) application names and name-prefixes. The BBC has registered the names "iPlayer", "News", "Sport", "ConnectedRedButton", as well as the prefixes "com.bbc" and "uk.co.bbc".

> The DIAL protocol has two components: DIAL Service Discovery and DIAL REST Service.

> The DIAL Service Discovery protocol is based on SSDP (Simple Service Discovery Protocol) version 1.1 as defined in UPnP and HTTP.

> The DIAL REST Service provides means of starting and stopping applications on the 1st screen, and getting information about available applications, such as name, state (running/stopped)."

### References

[http://www.dial-multiscreen.org/dial-protocol-specification](http://www.dial-multiscreen.org/dial-protocol-specification)
[http://www.dial-multiscreen.org/dial-protocol-specification/DIAL-2ndScreenProtocol-1.6.4.pdf?attredirects=0&d=1](http://www.dial-multiscreen.org/dial-protocol-specification/DIAL-2ndScreenProtocol-1.6.4.pdf?attredirects=0&d=1)
[http://upnp.org/specs/arch/UPnP-arch-DeviceArchitecture-v1.1.pdf](http://upnp.org/specs/arch/UPnP-arch-DeviceArchitecture-v1.1.pdf)

## Chromecast

Chromecast uses DIAL for setup. You can see a walk-through of the whole process: [http://planb.nicecupoftea.org/2013/11/10/chromecast-setup/](http://planb.nicecupoftea.org/2013/11/10/chromecast-setup/)

### Reference
[http://forum.xda-developers.com/showpost.php?p=48350742&postcount=2](http://forum.xda-developers.com/showpost.php?p=48350742&postcount=2)

## WebRTC

This is for video and audio chat within the browser. It doesn't have any explicit discovery mechanisms per se.

> "Signaling methods and protocols are not specified by WebRTC: signaling is not part of the RTCPeerConnection API.

> Instead, WebRTC app developers can choose whatever messaging protocol they prefer, such as SIP or XMPP, and any appropriate duplex (two-way) communication channel."

### References

[http://www.webrtc.org/](http://www.webrtc.org/)
[http://www.html5rocks.com/en/tutorials/webrtc/basics/](http://www.html5rocks.com/en/tutorials/webrtc/basics/)

## Audio / optical pairing and communication

Audio or flashing light can be used as a means of out of band token-passing instead of QR codes or pin numbers.

For audio, it's known as "Frequency Shift Keying" - a way to map characters to frequencies, which can be audible or not.

The basic idea has been around for a long time: it's the way that modems work. There's been some recent work in this area: digital voices, Chirp, quietnet and others. We did some initial exploration in the NoTube project, which I have picked up again recently for Radiodan in W3C's web audio API ("Beep"). NFC can use frequency shift keying as well as Amplitude shift keying, but is non-audible.

For flashing lights, electric Imp is the most well known. More generally it's called "Optical Wireless Communication".

### References:

[http://en.wikipedia.org/wiki/Frequency-shift_keying](http://en.wikipedia.org/wiki/Frequency-shift_keying)
[http://www.ics.uci.edu/~lopes/dv/dv.html](http://www.ics.uci.edu/~lopes/dv/dv.html)
[http://wiki.foaf-project.org/w/DanBri/ChirpChirp](http://wiki.foaf-project.org/w/DanBri/ChirpChirp)
[http://chirp.io/tech/](http://chirp.io/tech/)
[http://chirp.io/tech/#sthash.8x6ypBMA.dpuf](http://chirp.io/tech/#sthash.8x6ypBMA.dpuf)
[https://github.com/Katee/quietnet](https://github.com/Katee/quietnet)
[http://smus.com/ultrasonic-networking/](http://smus.com/ultrasonic-networking/)
[http://electricimp.com](http://electricimp.com)
[http://www.researchgate.net/publication/220144443_Indoor_optical_wireless_communication_potential_and_state-of-the-art/file/79e4150cb19442f72a.pdf](http://www.researchgate.net/publication/220144443_Indoor_optical_wireless_communication_potential_and_state-of-the-art/file/79e4150cb19442f72a.pdf)

## NFC

NFC can be used for service initiation (which could be in the role of a QR code as above), or for bypassing some of the complexity of bluetooth pairing (see below).

> "NFC is a set of short-range wireless technologies, typically requiring a distance of 10 cm or less. NFC operates at 13.56 MHz on ISO/IEC 18000-3 air interface and at rates ranging from 106 kbit/s to 424 kbit/s. NFC always involves an initiator and a target; the initiator actively generates an RF field that can power a passive target"

> "Passive communication mode: The initiator device provides a carrier field and the target device answers by modulating the existing field. In this mode, the target device may draw its operating power from the initiator-provided electromagnetic field, thus making the target device a transponder.
Active communication mode: Both initiator and target device communicate by alternately generating their own fields. A device deactivates its RF field while it is waiting for data. In this mode, both devices typically have power supplies."

### References

[http://en.wikipedia.org/wiki/Near_field_communication](http://en.wikipedia.org/wiki/Near_field_communication)
[http://en.wikipedia.org/wiki/ISO/IEC_15693](http://en.wikipedia.org/wiki/ISO/IEC_15693)

## Bluetooth Secure Simple Pairing (SSP) using NFC

> "NFC can simplify the process of authenticated pairing between two Bluetooth devices by exchanging authentication information over an NFC link.
Devices that comply with [BLUETOOTH_CORE] and subsequent versions use Secure Simple Pairing (SSP). SSP provides a stronger level of security, yet makes it easier for the user to perform pairing. SSP explicitly introduces the notion of Out-of-Band (OOB) pairing. The information (Hash C and Randomizer R, described in Section 3.3) can be exchanged over an NFC link to be used as part of the OOB pairing process."

### References

[http://members.nfc-forum.org/resources/AppDocs/NFCForum_AD_BTSSP_1_0.pdf](http://members.nfc-forum.org/resources/AppDocs/NFCForum_AD_BTSSP_1_0.pdf)
[https://www.bluetooth.org/DocMan/handlers/DownloadDoc.ashx?doc_id=282159](https://www.bluetooth.org/DocMan/handlers/DownloadDoc.ashx?doc_id=282159)

## XMPP

The Extensible Messaging and Presence Protocol (XMPP) is an application profile of XML that enables the near-real-time exchange of structured yet extensible data between any two or more network entities. XMPP is based on the earlier Jabber protocol.

XMPP defines several core protocol methods:

* setup and teardown of XML streams
* channel encryption
* authentication
* error handling
* messaging
* network availability ("presence")
* request-response interactions

The purpose of XMPP is to enable the exchange of relatively small pieces of structured data (called "XML stanzas") over a network between any two (or more) entities.
XMPP is typically implemented using a distributed client-server architecture, where a client needs to connect to a server in order to gain access to the network and thus be allowed to exchange XML stanzas with other entities (which can be associated with other servers).
  
The process whereby a client connects to a server, exchanges XML stanzas, and ends the connection is:

1. Determine the IP address and port at which to connect, typically based on resolution of a fully qualified domain name
2. Open a TCP connection
3. Open an XML stream over TCP
4. Preferably negotiate TLS for channel encryption
5. Authenticate using a Simple Authentication and Security Layer (SASL) mechanism
6. Bind a resource to the stream
7. Exchange any number of XML stanzas with other entities on the network
8. Close the XML stream
9. Close the TCP connection

Because this architectural style involves ubiquitous knowledge of network availability and a conceptually unlimited number of concurrent information transactions in the context of a given client-to-server or server-to-server session, we label it "Availability for Concurrent Transactions" (ACT) to distinguish it from the "Representational State Transfer" (REST) architectural style familiar from the World Wide Web.
  
Although the architecture of XMPP is similar to that of email, it introduces several modifications to facilitate communication in close to real time. End-to-end communication in XMPP is logically peer-to-peer but physically client-to-server-to-server-to-client.
The salient features of this ACTive architectural style are as follows:


* As with email, XMPP uses globally unique addresses (based on the Domain Name System) in order to route and deliver messages over the network.
* Persistent connections between client and server enables pushing of data at any time for immediate routing or delivery.
* Messages include 'from' and 'to' fields to allow message routing.
* Clients announce their presence on the network by sending 'presence' messages.
* XMPP discovery uses DNS SRV records, e.g: _xmpp-client._tcp.example.net (for client-server) or _xmpp-server._tcp.example.net (for server-server).

An XMPP extension (http://xmpp.org/extensions/xep-0030.html) allows for discovering information about other XMPP entities. Two kinds of information can be discovered: (1) the identity and capabilities of an entity, including the protocols and features it supports; and (2) the items associated with an entity, such as the list of rooms hosted at a multi-user chat service.

Another extension [http://xmpp.org/extensions/xep-0060.html](http://xmpp.org/extensions/xep-0060.html) allows for publish-subscribe messaging.

### References

[http://xmpp.org/](http://xmpp.org/)
[http://xmpp.org/rfcs/rfc6120.html](http://xmpp.org/rfcs/rfc6120.html)
[http://xmpp.org/xmpp-protocols/xmpp-extensions/](http://xmpp.org/xmpp-protocols/xmpp-extensions/)

