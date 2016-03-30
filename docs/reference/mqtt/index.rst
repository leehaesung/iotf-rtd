.. _ref-mqtt:

MQTT
====

The primary mechanism that devices and applications use to communicate
with the IBM Watson IoT platform is MQTT; this is a protocol
designed for the efficient exchange of real-time data with sensor and
mobile devices.

MQTT runs over TCP/IP and, while it is possible to code directly to
TCP/IP, you might prefer to use a library that handles the details of
the MQTT protocol for you. You will find there's a wide range of MQTT
client libraries available at mqtt.org_,
with the best place to start looking being the `Eclipse Paho project`_.  
IBM contributes to the development and support of many of these libraries.

.. _mqtt.org: https://github.com/mqtt/mqtt.github.io
.. _Eclipse Paho project: http://eclipse.org/paho/

MQTT 3.1 is the version of the protocol that is in widest use
today. Version 3.1.1 contains a number of minor enhancements, and has been ratified as an OASIS Standard. 

One reason for using version 3.1.1 is that the maximum length of the
MQTT Client Identifier (ClientId) is increased from the 23 character
limit imposed by 3.1. The IoT service will often require longer
ClientId's and will accept long ClientId's with either version of the
protocol however some 3.1 client libraries check the ClientId and
enforce the 23 character limit.


Device and application clients
------------------------------
We define two primary classes of thing: Devices & Applications

The class of thing that your MQTT client identifies itself to the service as 
will determine the capabilities of your client once connected as well as the 
mechanism through which you will need to authenticate.

Applications and devices also work with different MQTT topic spaces.  Devices
work within a device-scoped topic space, whereas applications have full access
to the topic space for an entire organization.

- :doc:`../../devices/mqtt`
- :doc:`../../applications/mqtt`
- :doc:`../../gateways/mqtt`


.. _qoslevels:

Quality of service
------------------
The MQTT protocol provides three qualities of service for delivering messages 
between clients and servers: "at most once", "at least once" and "exactly once".
Events and commands can be sent using any quality of service level, however you 
should carefully consider whether what the right level is for your needs.  It 
is not a simple case that QoS2 is "better" than QoS0.


At most once (QoS0)
~~~~~~~~~~~~~~~~~~~
The message is delivered at most once, or it might not be delivered at all. 
Delivery across the network is not acknowledged, and the message is not 
stored. The message could be lost if the client is disconnected, or if 
the server fails. QoS0 is the fastest mode of transfer. It is sometimes 
called "fire and forget".
    
The MQTT protocol does not require servers to forward publications at QoS0 
to a client. If the client is disconnected at the time the server receives the 
publication, the publication might be discarded, depending on the server 
implementation.

.. tip:: When sending real-time data on an interval we recommend using QoS0.  If a 
   single message goes missing it does not really matter as another message will
   be sent shortly after containing newer data.  In this scenario the extra cost
   of using higher quality of service does not result in any tangible benefit.


At least once (QoS1)
~~~~~~~~~~~~~~~~~~~~
The message is always delivered at least once. It might be delivered multiple 
times if there is a failure before an acknowledgment is received by the 
sender. The message must be stored locally at the sender, until the sender 
receives confirmation that the message has been published by the receiver. 
The message is stored in case the message must be sent again.


Exactly once (QoS2)
~~~~~~~~~~~~~~~~~~~
The message is always delivered exactly once. The message must be stored 
locally at the sender, until the sender receives confirmation that the message 
has been published by the receiver. The message is stored in case the message 
must be sent again. QoS2 is the safest, but slowest mode of transfer. A more 
sophisticated handshaking and acknowledgement sequence is used than for QoS1 
to ensure no duplication of messages occurs.

.. tip:: When sending commands we recommend using QoS2.  In many cases, when 
   processing commands you want to know that the command will only be actioned, 
   and that it will be actioned only once.  This is one of the clearest examples
   of when the additional overhead of QoS2 has a clear benefit.


Subscription Buffers and Clean Session
--------------------------------------
Each subscription from either a device or application is allocated a buffer of 
5000 messages.  This allows for any application or device to fall behind the 
live data it is processing and build up a backlog of up to 5000 pending 
messages for each subscription it has made.  Once the buffer fills up, any new 
message will result in the oldest message in the buffer being discarded.

The subscription buffer can be accessed using MQTT clean session option.  If clean
session is set to false, a subscriber will start receiving messages from the buffer.
If clean session is set to true, the buffer is reset.

.. note:: This limit applies regardless of the quality of service setting used.  
    Thus it is possible that a message sent at QoS1 or QoS2 may not be delivered 
    to an application that is unable to keep up with the messages rate for the 
    subscription(s) it has made.


Message Payload
---------------

The Watson IoT platform supports sending and recieving messages in any format.


Watson IoT Platform Maximum Message Payload Size
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The maximum payload size is 131072 bytes.  If messages with payloads larger
than this limit are sent, the client will be disconnected and the following log
message will appear in the Diagnostic Logs if the client is a device:

Closed connection from x.x.x.x. The message size is too large for this endpoint.
