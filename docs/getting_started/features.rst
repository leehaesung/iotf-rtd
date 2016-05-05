Feature Overview
================

Device Registry
---------------
Manage your inventory, configure security, and store metadata for millions of unique devices.  Define 
device types to represent individual device models and apply default metadata to all devices of that type.


Connectivity
------------
Securely connect your devices, gateways and applications directly to the Watson IoT platform via MQTT.  See the section 
on MQTT in the :ref:`reference material <ref-mqtt>` to learn more about the advantages of using 
this protocol.  Model the data from your device as events and control the flow of events into your 
applications.


Gateway Support
---------------
In many cases where a direct connection can not be made between the service and a device, the Watson IoT platform allows 
gateway devices to connect that can provide indirect connectivity for multiple devices.


Device Management
-----------------
Optionally, allow the Watson IoT platform to manage the lifecycle of your devices by implementing support for 
the Watson IoT platform's device management protocol in your devices.  The means by which the device
connects to the service does not affect the device management protocol, which functions the 
same for directly connected, indirectly connected, and gateway devices.  


External Service Integration
----------------------------
The Watson IoT platform supports integration with external services to bring data and operations supported by 
other online services into the platform, allowing your application and device developers to
seemlessly interact with those services without ever leaving the comfort of the Watson IoT platform APIs.

.. important:: This feature is currently available as part of a limited beta.  Future updates 
  may include changes incompatible with the current version of this feature.  Try it out and `let us know what you 
  think <https://developer.ibm.com/answers/smart-spaces/17/internet-of-things.html>`_


Historian
---------
Configure the Watson IoT platform to store a record of the events your devices generate.

.. warning:: **Message format restrictions apply to the Historian**
  
  The historian feature only supports JSON messages meeting specific critera:
  
  * The message must be a valid JSON object (not an array) with only two top level
    elements: ``d`` and ``ts``
  * The message must be UTF-8 encoded
  * The message must be less than 4kb

  **Data**
  
  The **d** element is where you include all data for the event (or
  command) being transmitted in the message. 
  
  * This element is required for your message to meet the Watson IoT platform message specification.
  * This must always be a JSON object (not an array)
  * In the case where you wish to send no data the **d** element should 
    still be present, but contain an empty object.

  **Timestamp**
  
  The **ts** element allows you to associate a timestamp with the event
  (or command). This is an optional element, if included its value should
  be a valid ISO8601 timestamp.

  **Example**
  
  .. code:: json
  
      {
        "d": {
          "host": "IBM700-R9E683D", 
          "mem": 54.9, 
          "network": {
            "up": 1.22, 
            "down": 0.55
          },
          "cpu": 1.3, 
        },
        "ts": "2014-12-30T14:47:36+00:00"
      }

Last Event Cache
-----------------
The API can now return the last recorded value of an event-id for a specific device, or the last recorded value for each event-id reported by a specific device. The last event cache only applies to values sent in the last 30 days. 

To request the most recent value for a specific event-id, use the following API request. This request will return the last recorded value for the event-id "power".

.. code::

	GET /api/v0002/device/types/<device-type>/devices/<device-id>/events/power
	
The response is returned as JSON in the following format: 

.. code::

	{
		"deviceId": "<device-id>", 
		"eventId": "power", 
		"format": "json", 
		"payload": "eyJzdGF0ZSI6Im9uIn0=", 
		"timestamp": "2016-03-14T14:12:06.527+0000", 
		"typeId": "<device-type>"
	}
	
.. note
	
	While the API response is JSON, event payloads can be written in any format. Payloads returned by this API will be encoded in base64.
	
Alternatively, to request the most recent value for each event-id reported by this device, use the following API request.

.. code::

	GET /api/v0002/device/types/<device-type>/devices/<device-id>/events
	
The response will include all event-id's sent by the device. In this instance, it returns values for the "power" and "temperature" events.

.. code:: 

	[
	    {
	        "deviceId": "<device-id>", 
	        "eventId": "power", 
	        "format": "json", 
	        "payload": "eyJzdGF0ZSI6Im9uIn0=", 
	        "timestamp": "2016-03-14T14:12:06.527+0000", 
	        "typeId": "<device-type>"
	    }, 
	    {
	        "deviceId": "<device-id>", 
	        "eventId": "temperature", 
	        "format": "json", 
	        "payload": "eyJpbnRlcm5hbCI6MjIsICJleHRlcm5hbCI6MTZ9", 
	        "timestamp": "2016-03-14T14:17:44.891+0000", 
	        "typeId": "<device-type>"
	    }
	]
