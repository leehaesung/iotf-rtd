HTTP API for Devices
====================

.. important:: This feature is currently available as part of a limited beta.  Future updates 
  may include changes incompatible with the current version of this feature.  Try it out and `let us know what you 
  think <https://developer.ibm.com/answers/smart-spaces/17/internet-of-things.html>`_


Publish an event
----------------

As an alternative to MQTT it is possible for devices to submit events to the IoT Platform over an HTTP API.  Devices may submit a ``POST`` request to: ``https://${orgid}.internetofthings.ibmcloud.com/api/v0002/device/types/${typeId}/devices/${deviceId}/events/${eventId}``

For full details, see the relevant `API documentation <https://docs.internetofthings.ibmcloud.com/swagger/v0002.html#/>`__.  The request body (event payload) can have any content, however, MQTT message payload guidelines should be followed.

.. note:: When connecting a device or application to Quickstart you should use the following URL instead:  ``http://quickstart.internetofthings.ibmcloud.com/api/v0002/device/types/${typeId}/devices/${deviceId}/events/${eventId}``

.. important:: HTTP messaging only covers the ability to submit device events currently.  Device management 
    and command control require MQTT connectivity in your device.


Authentication
~~~~~~~~~~~~~~

Requests must include an authorization header. Basic authentication the only method supported. Applications 
are authenticated by API keys, when an application makes any request to the IoT Platform API it must present an API 
as it's credentials:

- username = "use-token-auth"
- password = Authentication token

Content-Type
~~~~~~~~~~~~

A ``Content-Type`` request header must be provided with the request. The following table shows the supported types and how they are mapped to the IoT Platform internal formats.

+----------------------------+-----------------------+
| Content-Type Header        | IoT Platform Format   |
+============================+=======================+
| text/plain                 | text                  |
+----------------------------+-----------------------+
| application/json           | json                  |
+----------------------------+-----------------------+
| application/xml            | xml                   |
+----------------------------+-----------------------+
| application/octet-stream   | bin                   |
+----------------------------+-----------------------+


Quality of Service
~~~~~~~~~~~~~~~~~~

The HTTP(S) protocol provides "at most once" best effort delivery, analagous to the QoS0 quality of service provided by the MQTT protocol. When using QoS0, or the HTTP(S) equivalent, to deliver event messages, the device or application must implement retry logic to guarantee delivery.

For more information on the MQTT protocol and Quality of Service levels, please see `MQTT for the IoT Platform <../reference/mqtt/index.html>`__ documentation.
