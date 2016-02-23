Device Management Protocol
==========================

Introduction
------------

The Device Management capabilities in the IoT Platform create a new class of connected devices, Managed Devices.

Managed Devices must, by definition, contain a management agent which can understand the IoT Platform Device Management Protocol, and send a Manage Device request to the IoT Platform Device Management server. Managed devices can access the device management operations as explained later in this document.

The Device Management Protocol defines a set of supported operations. A device management agent can support a subset of the operations, but the Manage device and Unmanage device operations must be supported. A device supporting firmware action operations must also support observation.

The Device Management Protocol is built on top of MQTT.  For details specific to how Device Management Protocol interacts with MQTT please see `MQTT Connectivity for Devices <../mqtt.html>`__


The Device Management Lifecycle
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. A device and its associated device type are created in the IoT Platform using the dashboard or API.
2. The device connects to the IoT Platform and uses the 'Manage Device' operation to become a managed device.
3. The device's metadata, as described in the Device Model can now be viewed and manipulated through device operations, for example, firmware update and device reboot.
4. The device can communicate updates through the device-management protocol, such as location or diagnostic information and error codes.
5. In order to provide a way to deal with defunct devices in large device populations, the 'Manage device' operation request has an optional lifetime parameter. This lifetime parameter is the number of seconds within which the device must make another 'Manage device' request in order to avoid being marked as dormant and becoming an unmanaged device.
6. When a device is decommissioned it can be removed from the IoT Platform using the dashboard or REST API.


Return Code Summary
~~~~~~~~~~~~~~~~~~~

There are several return codes which are sent in response to the actions listed above.

- 200: Operation succeeded
- 202: Accepted (for initiating commands)
- 204: Changed (for attribute updates)
- 400: Bad request, for example, if a device is not in the appropriate state for this command
- 404: Attribute was not found, this code is also used if the operation was published to an invalid topic.
- 409: Resource could not be updated due to a conflict, for example, the resource is being updated by two simultaneous requests, so update could be retried later
- 500: Unexpected device error
- 501: Operation not implemented


----


.. _manage-manage:

Manage Device
-------------

A device uses this request to become a managed device. It should be the first device management request sent by the device after connecting to the IoT Platform. It would be usual for a device management agent to send this whenever it starts or restarts.   

.. important:: Support for this operation is mandatory for any managed devices.


Topic
~~~~~~

.. code:: 

	iotdevice-1/mgmt/manage


Message Format
~~~~~~~~~~~~~~~~

For the request, the ``d`` field and all of its sub-fields are optional. The ``metadata`` and ``deviceInfo`` field values replace the corresponding attributes for the sending device if they are sent.

The optional ``lifetime`` field specifies the length of time in seconds within which the device must send another 'Manage device' request in order to avoid being reverted to an unmanaged device and marked as dormant. If omitted or if set to ``0``, the managed device will not become dormant.  When set, the minimum supported setting is ``3600`` (1 hour).

Optional ``supports.deviceActions`` and ``supports.firmwareActions`` indicate the capabilities of the device management agent. If ``supports.deviceActions`` is set, the agent supports Reboot and Factory Reset actions. For a device that does not distinguish between rebooting and factory reset, it is acceptable to use the same behaviour for both actions. If ``supports.firmwareActions`` is set, the agent supports Firmware Download and Firmware Update actions.

Request Format:

.. code:: 

	{
		"d": {
			"metadata":{},
			"lifetime": number,
			"supports": {
				"deviceActions": boolean,
				"firmwareActions": boolean
			},
			"deviceInfo": {
				"serialNumber": "string",
				"manufacturer": "string",
				"model": "string",
				"deviceClass": "string",
				"description" :"string",
				"fwVersion": "string",
				"hwVersion": "string",
				"descriptiveLocation": "string"
			}
		},
		"reqId": "string"
	}


Response Format:

.. code::

	{
		"rc": 200,
		"reqId": "string"
	}


Response Codes
~~~~~~~~~~~~~~

- 200: The operation was successful.
- 400: The input message does not match the expected format, or one of the values is out of the valid range.
- 404: The topic name is incorrect, or the device is not in the database.
- 409: A conflict occurred during the device database update. To resolve this, simplify the operation is necessary.


.. _manage-unmanage:


----


Unmanage Device
---------------

A device uses this request when it no longer needs to be managed. The IoT Platform will no longer send new device management requests to this device and all device management requests from this device will be rejected other than a 'Manage device' request.

.. important:: Support for this operation is mandatory for any managed devices.

Topic
~~~~~~

.. code::

	iotdevice-1/mgmt/unmanage
	
Message Format
~~~~~~~~~~~~~~~

Request Format:

.. code::

	{
		"reqId": "string"
	}
	
Response Format:

.. code:: 

	{
		"rc": 200,
		"reqId": "string"
	}
	
Response Codes
~~~~~~~~~~~~~~

- 200: The operation was successful.
- 400: The input message does not match the expected format, or one of the values is out of the valid range.
- 404: The topic name is incorrect, or the device is not in the database.
- 409: A conflict occurred during the device database update. To resolve this, simplify the operation is necessary.


----


.. _update-location:

Update Location
----------------

Devices can change their location over time. The update of the location can happen in two ways:

- The device itself notifies the IoT Platform about the location update: The device retrieves its location from a GPS receiver and sends a device management message to the IoT Platform to update its location. The timestamp captures the time at which the location was retrieved from the GPS receiver. This means that the timestamp is valid, even if the transmission of the location message was delayed. In the event that the timestamp is omitted from the device management message sent, the current date and time on message receipt will be used when the device's location metadata is updated.

- A user / app updates the location of a device using the Rest API: The IoT Platform REST API is used to set the location metadata of a static device. This can be done at the time that the device is registered, or later if required. It is optional whether to include a timestamp. If omitted, the current date and time will be set as the device’s location metadata.

Location update triggered by device
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Devices that can determine their location can choose to notify the IoT Platform device management server about location changes.

Topic
~~~~~~

.. code::

	iotdevice-1/device/update/location


Location update triggered by user or app
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

When a user or application updates the location of an active managed device the device retrieves an update message: 

Topic
~~~~~~

.. code::

	iotdm-1/device/update

	
Message Format
~~~~~~~~~~~~~~

The "measuredDateTime" is the date of location measurement. The "updatedDateTime" is the date of the update to the device information. For efficiency reasons, the IoT Platform may batch updates to location information so the updates may be slightly delayed. The "latitude" and "longitude" should be specified in decimal degrees using WGS84. 

Whenever location is updated, the values provided for latitude, longitude, elevation and uncertainty are considered as a single multi-value update. The latitude and longitude are mandatory and must both be provided with each update.  Elevation and uncertainty are optional and can be omitted. 

If an optional value is provided on an update and then omitted on a later update, the earlier value is deleted by the later update. Each update is considered as a complete multi-value set.

Location update triggered by device
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Request Format:

.. code:: json

	{
		"d": {
			"longitude": number,
			"latitude": number,
	
			"elevation": number,
			"measuredDateTime": "string in ISO8601 format",
			"updatedDateTime": "string in ISO8601 format",
			"accuracy": number
		},
		"reqId": "string"
	}

Response Format:

.. code:: json 

	{
		"rc": 200,
		"reqId": "string"
	}
	
Response Codes
~~~~~~~~~~~~~~

- 200: The operation was successful.
- 400: The input message does not match the expected format, or one of the values is out of the valid range.
- 404: The topic name is incorrect, or the device is not in the database.
- 409: A conflict occurred during the device database update. To resolve this, simplify the operation is necessary.

Location update triggered by user or app
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Payload Format:

.. code:: json

    {
        "d": {
            "fields": [
                { 
                    "field": "location",
                    "value": {
                        "latitude": number,
                        "longitude": number,
                        "elevation": number,
                        "accuracy": number,
                        "measuredDateTime": "string in ISO8601 format"
                    }
                }
            ]
        }
    }
    


Please note: there is no reqId as no response by device is required.


----


.. _update-attributes:

Update Device Attributes
------------------------

The IoT Platform can send this request to a device to update values of one or more device attributes. Attributes that can be updated by the Rest API are location, metadata, device information and firmware.

The "value" is the new value of the device attribute. It is a complex field matching the device model. Only writeable fields should be updated as a result of this operation. Values can be updated in:

- location (see Update location section for details)
- metadata (Optional)
- deviceInfo (Optional)
- mgmt.firmware	(see Firmware update process for details)


Topic
~~~~~~~

.. code:: 

	iotdm-1/device/update

	
Message format
~~~~~~~~~~~~~~~~

Payload Format:

.. code:: 

	{
		"d": {
			"fields": [
				{ 
					"field": "location",
					"value": ""
				}
			]
		}
	}


----


.. _diag-add-error-code:

Add Error Code
--------------

Devices can choose to notify the IoT Platform device management server about changes in their error status.

Topic
~~~~~~~

.. code:: 

	iotdevice-1/add/diag/errorCodes

Message Format
~~~~~~~~~~~~~~~

The "errorCode" is a current device error code that needs to be added to the IoT Platform.

Request Format:

.. code:: 

	{
		"d": {
			"errorCode": number
		},
		"reqId": "string"
	}


Response Format:

.. code::

	{
		"rc": 200,
		"reqId": "string"
	}


Response Codes
~~~~~~~~~~~~~~

- 200: The operation was successful.
- 400: The input message does not match the expected format, or one of the values is out of the valid range.
- 404: The topic name is incorrect, or the device is not in the database.
- 409: A conflict occurred during the device database update. To resolve this, simplify the operation is necessary.


----

.. _diag-clear-error-codes:


Clear Error Codes
-----------------

Devices can request that the Internet of Things Foundation clear all of their error codes.

Topic
~~~~~~

.. code::

	iotdevice-1/clear/diag/errorCodes

Message Format
~~~~~~~~~~~~~~~

Request Format:

.. code:: 

	{
		"reqId": "string"
	}
	
Response Format:

.. code::

	{
		"rc": 200,
		"reqId": "string"
	}


Response Codes
~~~~~~~~~~~~~~

- 200: The operation was successful.
- 400: The input message does not match the expected format, or one of the values is out of the valid range.
- 404: The topic name is incorrect, or the device is not in the database.
- 409: A conflict occurred during the device database update. To resolve this, simplify the operation is necessary.


----


.. _diag-add-log:

Add Log
-------

Devices can choose to notify IoTF device management support about changes a new log entry. Log entry includes a log messages, its timestamp and severity, as well as an optional base64-encoded binary diagnostic data.

Topic
~~~~~

.. code:: 

	iotdevice-1/add/diag/log

Message Format
~~~~~~~~~~~~~~~

"message" is a diagnostic message that needs to be added to IoTF.
"timestamp" is a date and time of the log entry in ISO8601 format.
"data" is an optional base64-encoded diagnostic data.
"severity" is a severity of the message (0: informational, 1: warning, 2: error).

Request Format:

.. code:: 

	{
		"d": {
			"message": string,
			"timestamp": string,
			"data": string,
			"severity": number
		},
		"reqId": "string"
	}


Response Format:

.. code::

	{
		"rc": 200,
		"reqId": "string"
	}


Response Codes
~~~~~~~~~~~~~~

- 200: The operation was successful.
- 400: The input message does not match the expected format, or one of the values is out of the valid range.
- 404: The topic name is incorrect, or the device is not in the database.
- 409: A conflict occurred during the device database update. To resolve this, simplify the operation is necessary.


----


.. _diag-clear-logs:

Clear Logs
----------

Devices can request that the Internet of Things Foundation clear all of their log entries.

Topic
~~~~~~

.. code::

	iotdevice-1/clear/diag/log

Message format
~~~~~~~~~~~~~~~

Request Format:

.. code:: 

	{
		"reqId": "string"
	}
	
Response Format:

.. code::

	{
		"rc": 200,
		"reqId": "string"
	}

Response Codes
~~~~~~~~~~~~~~

- 200: The operation was successful.
- 400: The input message does not match the expected format, or one of the values is out of the valid range.
- 404: The topic name is incorrect, or the device is not in the database.
- 409: A conflict occurred during the device database update. To resolve this, simplify the operation is necessary.


----


.. _observations-observe:

Observe Attribute Changes
-------------------------

The IoT Platform can send this request to a device to observe changes of one or more device attributes. When the device receives this request, it must send a notification request ("notify" message) to the IoT Platform whenever the observed attributes value changes.

.. important:: Devices must implement observe, notify & cancel operations in order to support :ref:`firmware-actions-update`.

Topic
~~~~~~

.. code:: 

	iotdm-1/observe

Message format
~~~~~~~~~~~~~~~

The "fields" field is an array of the device attribute names from the device model. For example, values could be "location", "mgmt.firmware" or "mgmt.firmware.state". If a complex field, such as "mgmt.firmware" is specified, it is expected that its underlying fields are updated at the same time, such that only a single notify message is generated.

The "message" field used in the response can be specified if "rc" is not 200. If any field value which was to be observed could not be retrieved, "rc" should be set to 404 (if not found) or 500 (any other reason). When values for fields to be observed cannot be found, "fields" should contain an array of elements with "field" set to the name of each field that could not be read, "value" fields should be omitted. For the response code to be set to 200, both "field" and "value" must be specified, "value" is the current value of an attribute identified by "field" content.

Request Format:

.. code::

	{
		"d": {
			"fields": [
				"string"
			]
		},
		"reqId": "string"
	}

Response Format:

.. code::

	{
		"rc": number,
		"message": "string",
		"d": {
			"fields": [
				{ 
					"field": "field_name",
					"value": "field_value"
				}
			]
		},
		"reqId": "string"  
	}


----


.. _observations-cancel:

Cancel Attribute Observation
----------------------------

The IoT Platform can send this request to a device to cancel the current observation of one or more device attributes. The "fields" is an array of the device attribute names from the device model, for example, values could be "location", "mgmt.firmware" or "mgmt.firmware.state".

The "message" field must be specified if "rc" is not 200.

.. important:: Devices must implement observe, notify & cancel operations in order to support :ref:`firmware-actions-update`.

Topic
~~~~~~

.. code::

	iotdm-1/cancel


Message format
~~~~~~~~~~~~~~~~

Request Format:

.. code::

	{
		"d": {
			"fields": [
				"string"
			]
		},
		"reqId": "string"
	}

Response Format:

.. code:: 

	{
		"rc": number,
		"message": "string",
		"reqId": "string"  
	}


----


.. _observations-notify:

Notify Attribute Changes
------------------------

The IoT Platform can make an observation request referring to a specific attribute or set of values. When the value of the attribute or attributes changes, the device must send a notification containing the latest value.

The "field_name" value is the name of the attribute that has changed, the "field_value" is the current value of the attribute. The attribute can be a complex field, if multiple values in a complex field are updated as a result of a single operation, only a single notification message should be sent.

If notify request is processed successfully, "rc" should be set to 200. If the request is not correct, "rc" should be set to 400. If the field specified in the notify request is not being observed, "rc" should be set to 404.

.. important:: Devices must implement observe, notify & cancel operations in order to support :ref:`firmware-actions-update`.


Topic
~~~~~~

.. code::

	iotdevice-1/notify
	
Message format
~~~~~~~~~~~~~~~

Request Format:

.. code::

	{
		"d": {
			"field": "field_name",
			"value": "field_value"
		}
		"reqId": "string"
	}
	
Response Format:

.. code::

	{
		"rc": number,
		"reqId": "string"
	}

Response Codes
~~~~~~~~~~~~~~

- 200: The operation was successful.
- 400: The input message does not match the expected format, or one of the values is out of the valid range.
- 404: The topic name is incorrect, the device is not in the database, or there is no observation for the field reported.
- 409: A conflict occurred during the device database update. To resolve this, simplify the operation is necessary.
- 500: An internal error occurred, contact IBM Support.
