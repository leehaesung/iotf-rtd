MQTT Connectivity for Gateways
==============================

MQTT client connection
----------------------
Every registered organization has a unique endpoint which must be used when 
connecting MQTT clients for gateways in that organization.

**org\_id**.messaging.internetofthings.ibmcloud.com


Unencrypted client connection
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Connect on port **1883**

.. important:: All information your gateway submits is being sent in 
    plain text (including the authentication credentials for your gateway).  
    We recommend the use of an encrypted connection whenever possible.


Encrypted client connection
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Connect on port **8883** or **443** for websockets.
it
In many client libraries you will need to provide the server's public certificate 
in pem format.  The following file contains the entire certificate chain for 
\*.messaging.internetofthings.ibmcloud.com: messaging.pem_

.. _messaging.pem: https://github.com/ibm-messaging/iot-python/blob/master/src/ibmiotf/messaging.pem

.. tip:: Some SSL client libraries have been shown to not handle wildcarded
    domains, in which case, if you can not change libraries, you will need to turn 
    off certificate checking.

.. note:: The IoT Platform requires TLS v1.2. We suggest the following cipher suites: ECDHE-RSA-AES256-GCM-SHA384, AES256-GCM-SHA384, ECDHE-RSA-AES128-GCM-SHA256 or AES128-GCM-SHA256 *(as of Jun 1 2015)*.
   


MQTT client identifier
----------------------

A gateway must authenticate using a client ID in the following format:

    g:**orgId**:**typeId**:**deviceId**

-  **g** idenfities your client as a gateway
-  **orgId** is your unique organization ID, assigned when you sign up
   with the service.  It will be a 6 character alphanumeric string.
-  **typeId** is intended to be used as an identifier of the type
   of gateway connecting, it may be useful to think of this as analogous
   to a model number. 
-  **deviceId** must uniquely identify a gateway device across all gateways of
   a specific type, it may be useful to think of this as
   analogous to a serial number.

.. note:: You can use any scheme of your choice when assigning values for 
    ``typeId`` and ``deviceId``, however the following restrictions apply to both:

    - Maximum length of 36 characters 
    - Must comprise only alpha-numeric characters (``a-z``, ``A-Z``, ``0-9``) and the following special characters:

      - dash (``-``)
      - underscore (``_``)
      - dot (``.``)


MQTT authentication
-------------------

Username
~~~~~~~~

The service currently only supports token-based authentication for
devices, as such there is only one valid username for gateways today.

A value of ``use-token-auth`` indicates to the service that the
authentication token for the gateway will be passed as the password for
the MQTT connection.

Password
~~~~~~~~

When using token based authentication submit the device authentication
token as the password when making your MQTT connection.


Publishing events
-----------------

A gateway can publish events from itself and on behalf of any device
connected via the gateway by using the following topic and substituting 
in the appropriate ``typeId`` and ``deviceId`` based on the intended 
origin of the event:

  iot-2/type/\ **typeId**/id/\ **deviceId**/evt/\ **eventId**/fmt/\ **formatString**

Example
~~~~~~~
+-----------+------------+--------------+
|           | ``typeId`` | ``deviceId`` |
+===========+============+==============+
| Gateway 1 | mygateway  | gateway1     |
+-----------+------------+--------------+
| Device 1  | mydevice   | device1      |
+-----------+------------+--------------+

- Gateway 1 can publish its own status events: ``iot-2/type/mygateway/id/gateway1/evt/status/fmt/json``
- Gateway 1 can publish status events on behalf of Device 1: ``iot-2/type/mydevice/id/device1/evt/status/fmt/json``


.. important:: The message payload is limited to a maximum of 4096 bytes.  Messages larger than this will be rejected.


Subscribing to commands
-----------------------

A gateway can subscribe to commands directed at the gateway itself and to any device
connected via the gateway by using the following topic and substituting 
in the appropriate ``typeId`` and ``deviceId``:

  iot-2/type/\ **typeId**/id/\ **deviceId**/cmd/\ **commandId**/fmt/\ **formatString**

The MQTT ``+`` wildcard can be used for ``typeId``, ``deviceId``, ``commandId`` 
and ``formatString`` to subscribe to multiple command sources.

Example
~~~~~~~

+-----------+------------+--------------+
|           | ``typeId`` | ``deviceId`` |
+===========+============+==============+
| Gateway 1 | mygateway  | gateway1     |
+-----------+------------+--------------+
| Device 1  | mydevice   | device1      |
+-----------+------------+--------------+

- Gateway 1 can subscribe to commands directed at the gateway: ``iot-2/type/mygateway/id/gateway1/cmd/+/fmt/+``
- Gateway 1 can subscribe to commands sent to Device 1: ``iot-2/type/mydevice/id/device1/cmd/+/fmt/+``
- Gateway 1 can subscribe any command sent to devices of type "mydevice": ``iot-2/type/mydevice/id/+/cmd/+/fmt/+``


Gateway auto-registration
-------------------------
Gateway devices have the ability to automatically register devices which are connected to them. When a gateway publishes
a message or subscribes to a topic on behalf of another device, that device will automatically be registered if it does
not already exist.

Registration requests from gateway devices are throttled to 10 pending requests at a time. If trying to connect many new devices
to a gateway which have not previously been registered, then there may be some delay in the registration of the devices through
the gateway.

Gateway devices are limited to 500 active devices at a time. If exceeded, publish and subscribe requests for new devices 
will be dropped. The count is reset on gateway disconnect.

.. warning::
  If the gateway fails to automatically register a device, then it will not attempt to register that device again for a short
  period of time. Any messages or subscriptions from the failed device will be dropped during that time.
  
  
Gateway notifications
---------------------
When errors occur during the validation of the publish or subscribe topic, or during automatic registration, a notification will be 
sent to the gateway device. A gateway can receive these notifications by subscribing to the following topic, substituting in 
its own ``typeId`` and ``deviceId``:

  iot-2/type/\ **typeId**/id/\ **deviceId**/notify

Messages received on the notify topic will have the following format:

  .. code::
  
    { 
      "Request": "<Request_Type>", 
      "Time": "<Timestamp>", 
      "Topic": "<Topic>", 
      "Type": "<Device_Type>", 
      "Id": "<Device_Id>", 
      "Client": "<Client_ID", 
      "RC": <Return_Code>, 
      "Message": "<Message>" 
    }

- Request_Type: Either publish or subscribe
- Timestamp: Time in ISO 8601 Format
- Topic: The request topic from the gateway
- Device_Type: The device type from the topic
- Device_Id: The device id from the topic
- ClientID: The client id of the request
- RC: The return code
- Message: The error message

Notifications a gateway may receive:

- Topic does not match with any allowed topic rules.
- Device type is not valid.
- Device id is not valid.
- Maximum number of devices per gateway has been reached.
- Maximum number of devices per organization has been reached.
- Failed to create device due to internal errors.


Managed Gateways
----------------

Support for device lifecycle management is optional, the device management protocol 
used by IoT Platform utilises the same MQTT connection that your gateway already uses for events 
and command control.

Quality of Service Levels and Clean Session
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Managed gateways can publish messages with Quality of Service (QoS) level of 0 or 1. If 
QoS 1 is used, messages from the gateway will be queued if necessary. Messages from 
the gateway must not be retained messages. 

The IoT Platform publishes requests with a QoS level of 1 to support 
queuing of messages.  In order to queue messages sent while a managed gateway is not 
connected, the device should use ``cleansession=false``.

.. warning::
  If your managed gateway uses a durable subscription (cleansession=false) you need to be 
  aware that device management commands sent to your gateway while it is offline will be 
  reported as failed operations, however, when the gateway later connects those requests will 
  be actioned by the gateway.
  
  When handling failures it is important to take this into account if you are using durable
  subscriptions for your managed gateways.


Topics
~~~~~~

A managed gateway is required to subscribe to two topics to handle requests and responses from IoT Platform:

- The managed gateway will subscribe to device management reponses on ``iotdm-1/type/<typeId>/id/<deviceId>/response/+``
- The managed gateway will subscribe to device management requests on ``iotdm-1/type/<typeId>/id/<deviceId>/+``


A managed gateway will publish to two topics:

- The managed gateway will publish device management responses on ``iotdevice-1/type/<typeId>/id/<deviceId>/response/``
- The managed gateway will publish device management requests on ``iotdevice-1/type/<typeId>/id/<deviceId>/``

The gateway is able to process device management protocol messages for both itself and on behalf other connected devices
by using the relevant <typeId> and <deviceId>.


Message Format
~~~~~~~~~~~~~~

All messages are sent in JSON format. There are two types of message.

1. Request
    Requests are formatted as follows:
    
    .. code:: json
        
        {  "d": {...}, "reqId": "b53eb43e-401c-453c-b8f5-94b73290c056" }

    - ``d`` carries any data relevant to the request
    - ``reqId`` is an identifier of the request, and must be copied into a response. 
      If a response is not required, the field should be omitted.
    
2. Response
    Responses are formatted as follows:
    
    .. code::

        {
            "rc": 0,
            "message": "success",
            "d": {...},
            "reqId": "b53eb43e-401c-453c-b8f5-94b73290c056"
        }
    
    - "rc" is a result code of the original request.
    - ``message`` is an optional element with a text description of the response code.
    - ``d`` is an optional data element accompanying the response.
    - ``reqId`` is the request ID of the original request. This is used to correlate responses with 
      requests, and the device needs to ensure that all request IDs are unique.  When responding to IoT 
      Platform requests, the correct ``reqId`` value must be sent in the response.

