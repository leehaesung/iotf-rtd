Device Management Requests
==========================

The IoT Platform provides some actions which can be initiated against one or more devices. These actions can be initiated through the dashboard as well as by using the REST API. The available actions are split into two groups, device actions and firmware actions.

Initiating Device Management Requests through the dashboard
-----------------------------------------------------------

Requests can be initiated through the dashboard by navigating to the Actions tab of the Devices page. The **Initiate Action** button provides a dialog for selecting an action, picking the devices to take the action against, and specifying any additional parameters that the selected action supports.

Initiating Device Management Requests using the REST API
--------------------------------------------------------

Requests can be initiated using the following REST API:

``POST https://<org>.internetofthings.ibmcloud.com/api/v0002/mgmt/requests``

For more information on the request body, refer to the `API documentation <../../swagger/v0002.html>`__.

Device Actions
--------------

A device may specify that it supports device actions when it publishes a manage request. This indicates to the IoT Platform that the device is able to respond to device reboot and device reset actions.

.. _device-actions-reboot:

Device Actions - Reboot
----------------------

The reboot action can be initiated by using either the IoT Platform dashboard, or the REST API.

To initiate a device reboot using the REST API, issue a POST request to:

``https://<org>.internetofthings.ibmcloud.com/api/v0002/mgmt/requests``

The information provided is:

- The action ``device/reboot``
- A list of devices to reboot, with a maximum of 5000 devices

Example device reboot request:

.. code::

	{
		"action": "device/reboot",
		"devices": [
			{
				"typeId": "someType",
				"deviceId": "someId"
			}
		]
	}

Once a request has been initiated, an MQTT message will be published to all devices specified in the request body. For each device, a response must be sent back indicating whether or not the reboot action can be initiated. If this operation can be initiated immediately, set ``rc`` to ``202``. If the reboot attempt fails, set ``rc`` to ``500`` and set the ``message`` field accordingly. If reboot is not supported, set ``rc`` to ``501`` and optionally set the ``message`` field accordingly.

The reboot action is considered complete when the device sends a Manage device request following its reboot.

Topic
~~~~~~

The server publishes this request to a device on the following topic:

.. code:: 

	iotdm-1/mgmt/initiate/device/reboot

	
Message format
~~~~~~~~~~~~~~~

Request Format:

.. code:: 

	Incoming message from the server:
	
	Topic: iotdm-1/mgmt/initiate/device/reboot
	{
		"reqId": "string"
	}

Response Format:

.. code::

	Outgoing message from the device:
	
	Topic: iotdevice-1/response
	{
		"rc": "response_code",
		"message": "string",
		"reqId": "string"
	}


----


.. _device-actions-factory-reset:

Device Actions - Factory Reset
-----------------------------

The factory reset action can be initiated by using either the IoT Platform dashboard, or the REST API.

To initiate a device factory reset using the REST API, issue a POST request to:

``https://<org>.internetofthings.ibmcloud.com/api/v0002/mgmt/requests``

The information provided is:

- The action ``device/factoryReset``
- A list of devices to reset, with a maximum of 5000 devices

Example device reset request:

.. code::

	{
		"action": "device/factoryReset",
		"devices": [
			{
				"typeId": "someType",
				"deviceId": "someId"
			}
		]
	}
	
Once a request has been initiated, an MQTT message will be published to all devices specified in the request body. For each device, a response must be sent back indicating whether or not the factory reset action can be initiated. The response code should be set to ``202`` if this action can be initiated immediately. If the factory reset attempt fails, set ``rc`` to ``500`` and set the ``message`` field accordingly. If the factory reset action is not supported, set ``rc`` to ``501`` and optionally set the ``message`` field accordingly.

The factory reset action is considered complete when the device sends a Manage device request following its factory reset.

Topic
~~~~~~

The server publishes this request to a device on the following topic:

.. code::

	iotdm-1/mgmt/initiate/device/factory_reset


Message format
~~~~~~~~~~~~~~~

Request Format:

.. code::

	Incoming message from the server:
	
	Topic: iotdm-1/mgmt/initiate/device/factory_reset
	{
		"reqId": "string"
	}

Response Format:

.. code::

	Outgoing message from the device:
	
	Topic: iotdevice-1/response
	{
		"rc": "response_code",
		"message": "string",
		"reqId": "string"
	}


----


Firmware Actions
----------------

The firmware level currently known to be on a given device is stored in the ``deviceInfo.fwVersion`` attribute. 
The ``mgmt.firmware`` attributes are used to perform a firmware update and observe its status.

.. important:: The managed device must support observation of the ``mgmt.firmware`` attribute in order to support firmware actions.

The firmware update process is separated into two distinct actions, Downloading Firmware, and Updating Firmware. 
The status of each of these actions is stored in a separate attribute on the device. The ``mgmt.firmware.state`` 
attribute describes the status of the firmware download. The possible values for ``mgmt.firmware.state`` are:

+--------------+-------------+------------------------------------------------------------------------------------------+
| Value        | State       | Meaning                                                                                  |
+==============+=============+==========================================================================================+
| 0            | Idle        | The device is currently not in the process of downloading firmware                       |
+--------------+-------------+------------------------------------------------------------------------------------------+
| 1            | Downloading | The device is currently downloading firmware                                             |
+--------------+-------------+------------------------------------------------------------------------------------------+
| 2            | Downloaded  | The device has successfully downloaded a firmware update and it is ready to install      |
+--------------+-------------+------------------------------------------------------------------------------------------+



The ``mgmt.firmware.updateStatus`` attribute describes the status of firmware update. The possible values for ``mgmt.firmware.updateStatus`` are:

+--------------+---------------------+----------------------------------------------------------------------+
| Value        | State               | Meaning                                                              |
+==============+=====================+======================================================================+
| 0            | Success             | The firmware has been successfully updated                           |
+--------------+---------------------+----------------------------------------------------------------------+
| 1            | In Progress         | The firmware update has been initiated but is not yet complete       |
+--------------+---------------------+----------------------------------------------------------------------+
| 2            | Out of Memory       | An out of memory condition has been detected during the operation.   |
+--------------+---------------------+----------------------------------------------------------------------+
| 3            | Connection Lost     | The connection was lost during the firmware download                 |
+--------------+---------------------+----------------------------------------------------------------------+
| 4            | Verification Failed | The firmware did not pass verification                               |
+--------------+---------------------+----------------------------------------------------------------------+
| 5            | Unsupported Image   | The downloaded firmware image is not supported by the device         |
+--------------+---------------------+----------------------------------------------------------------------+
| 6            | Invalid URI         | The device could not download the firmware from the provided URI     |
+--------------+---------------------+----------------------------------------------------------------------+


----

.. _firmware-actions-download:

Firmware Actions - Download
---------------------------

The Download Firmware action can be initiated by using either the IoT Platform dashboard, or the REST API.

To initiate a firmware download using the REST API, issue a POST request to:

``https://<org>.internetofthings.ibmcloud.com/api/v0002/mgmt/requests``

The information provided is:

- The action ``firmware/download``
- The URI for the firmware image
- A list of devices to receive the image, with a maximum of 5000 devices
- Optional verifier string to validate the image
- Optional firmware name
- Optional firmware version

Example firmware download request on which all the following example messages are based:

.. code::

   {
      "action" : "firmware/download",
      "parameters" : [{
            "name" : "uri",
            "value" : "some uri for firmware location"
         }, {
            "name" : "name",
            "value" : "some firmware name"
         }, {
            "name" : "verifier",
            "value" : "some validation code"
         }, {
            "name" : "version",
            "value" : "some firmware version"
         }
      ],
      "devices" : [{
            "typeId" : "someType",
            "deviceId" : "someId"
         }
      ]
   }

The device management server in the IoT Platform uses the Device Management Protocol to send a request to the devices, initiating the firmware download. There are multiple steps: 

1. Firmware details update request sent on topic ``iotdm-1/device/update``:

   This request let the device validate if the requested firmware differs from the currently installed firmware. If there is a difference, set ``rc`` to ``204``, which translates to the status ``Changed``.
   The following example shows which message is to be expected for the previously sent example firmware download request and what response should be sent, when a difference is detected:

.. code::
   
   Incoming request from the IoT Platform:
   
   Topic: iotdm-1/device/update
   Message: 
   {
      "reqId" : "f38faafc-53de-47a8-a940-e697552c3194",
      "d" : {
         "fields" : [{
               "field" : "mgmt.firmware",
               "value" : {
                  "version" : "some firmware version",
                  "name" : "some firmware name",
                  "uri" : "some uri for firmware location",
                  "verifier" : "some validation code",
                  "state" : 0,
                  "updateStatus" : 0,
                  "updatedDateTime" : ""
               }
            }
         ]
      }
   }
   
   Outgoing response from device:
   
   Topic: iotdevice-1/response
   Message: 
   {
      "rc" : 204,
      "reqId" : "f38faafc-53de-47a8-a940-e697552c3194"
   }   
   
This response will trigger the next request.      
      
|
   
2. Observation request for firmware download status ``iotdm-1/observe``:

   Verifies if the device is ready to start the firmware download. When the download can be started immediately, set ``rc`` to ``200`` (``Ok``), ``mgmt.firmware.state`` to 
   ``0`` (``Idle``) and ``mgmt.firmware.updateStatus`` to ``0`` (``Idle``). Here an example exchange between the IoT Platform and device:
   
.. code::

   Incoming request from the IoT Platform:
   
   Topic: iotdm-1/observe
   Message:
   {
      "reqId" : "909b477c-cd37-4bee-83fa-1d568664fbe8",
      "d" : {
         "fields" : [ { 
         	"field" : "mgmt.firmware" 
         	}
         ]
      }
   }

   Outgoing response from device:
   
   Topic: iotdevice-1/response
   Message:
   {
      "rc" : 200,
      "reqId" : "909b477c-cd37-4bee-83fa-1d568664fbe8"
   }
   
This exchange will trigger the last step.

|
   
3. Initiate the download request sent on topic ``iotdm-1/mgmt/initiate/firmware/download``:
   
   This request tells a device to actually start the firmware download. If the action can be initiated immediately, set ``rc`` to ``202``. Here an example:
   
.. code::

   Incoming request from the IoT Platform:
   
   Topic: iotdm-1/mgmt/initiate/firmware/download
   Message:
   {
      "reqId" : "7b244053-c08e-4d89-9ed6-6eb2618a8734"
   }

   Outgoing response from device:
   
   Topic: iotdevice-1/response
   Message:
   {
      "rc" : 202,
      "reqId" : "7b244053-c08e-4d89-9ed6-6eb2618a8734"
   }

|
   
After a firmware download is initiated this way, the device needs to report to the IoT Platform the status of the download. This is possible by publishing a message to the ``iotdevice-1/notify``-topic, where the ``mgmt.firmware.state`` is set to either ``1`` (``Downloading``) or ``2`` (``Downloaded``).
Here some examples:

.. code:: 

   Outgoing message from device:
   
   Topic: iotdevice-1/notify
   Message:
   {
      "reqId" : "123456789",
      "d" : {
         "fields" : [ {
         	"field" : "mgmt.firmware",
         	"value" : {
            		"state" : 1
            	}
         } ]
      }
   }
   
   
   Wait some time...
   
   
   Outgoing message from device:
   
   Topic: iotdevice-1/notify
   Message:
   {
      "reqId" : "1234567890",
      "d" : {
         "fields" : [ {
         	"field" : "mgmt.firmware",
         	"value" : {
            		"state" : 2
            	}
         } ]
      }
   }
|

After the notification with ``mgmt.firmware.state`` set to ``2`` was published, a request will be triggered on the ``iotdm-1/cancel``-topic, which cancels the observation of the ``mgmt.firmware`` field. 
After a response with ``rc`` set to ``200`` was sent the firmware download is completed. Example:

.. code:: 

   Incoming request from the IoT Platform:
   
   Topic: iotdm-1/cancel
   Message:
   {
      "reqId" : "d9ca3635-64d5-46e2-93ee-7d1b573fb20f",
      "d" : {
         "fields" : [{
               "field" : "mgmt.firmware"
            }
         ]
      }
   }


   Outgoing message from device:
   
   Topic: iotdevice-1/response
   Message:
   {
      "rc" : 200,
      "reqId" : "d9ca3635-64d5-46e2-93ee-7d1b573fb20f"
   }

|
   
Useful information regarding error handling:

- If ``mgmt.firmware.state`` is not ``0`` ("Idle") an error should be reported with response code ``400``, and an optional message text.- 
- If ``mgmt.firmware.uri`` is not set or is not a valid URI, set ``rc`` to ``400``. 
- If firmware download attempt fails, set ``rc`` to ``500`` and optionally set ``message`` accordingly. 
- If firmware download is not supported, set ``rc`` to ``501`` and optionally set ``message`` accordingly.
- When an excecute request is received by the device, ``mgmt.firmware.state`` should change from ``0`` (Idle) to ``1`` (Downloading). 
- When the download has been completed successfully, ``mgmt.firmware.state`` should be set to ``2`` (Downloaded).
- If an error occurrs during download ``mgmt.firmware.state`` should be set to ``0`` (Idle) and ``mgmt.firmware.updateStatus`` should be set to one of the error status values: 

  - 2 (Out of Memory)
  - 3 (Connection Lost)
  - 6 (Invalid URI)

- If a firmware verifier has been set, the device should attempt to verify the firmware image. If the image verification fails, ``mgmt.firmware.state`` should be set to ``0`` (Idle) and ``mgmt.firmware.updateStatus`` should be set to the error status value ``4`` (Verification Failed).


----


.. _firmware-actions-update:

Firmware Actions - Update
-------------------------

The firmware update action supports 2 scenarios:

1. Installing firmware that is already on the device from a previously completed firmware download request, or
2. Downloading and installing firmware within a single device management request.

The firmware update action is initiated using the REST API by issuing a POST request to:

``https://<org>.internetofthings.ibmcloud.com/api/v0002/mgmt/requests``

The information which should be provided is:

- The action ``firmware/update``
- The list of devices to receive the image, all of the same device type.
- Optional The URI for the firmware image
- Optional verifier string to validate the image
- Optional firmware name
- Optional firmware version

Here is an example request:

.. code ::

   {
      "action" : "firmware/update",
      "parameters" : [{
            "name" : "uri",
            "value" : "some uri for firmware location"
         }, {
            "name" : "name",
            "value" : "some firmware name"
         }, {
            "name" : "verifier",
            "value" : "some validation code"
         }, {
            "name" : "version",
            "value" : "some firmware version"
         }
      ],
      "devices" : [{
            "typeId" : "someType",
            "deviceId" : "someId"
         }
      ]
   }
  
|

If any of the optional parameters are specified in the request, then the device management server will send a firmware details update to the device.
The firmware details update request is sent on topic ``iotdm-1/device/update``.

This request lets the device validate if the requested firmware differs from the currently installed firmware. If there is a difference, set ``rc`` to ``204``, which translates to the status ``Changed``.
The following example shows which message is to be expected for the previously sent example firmware update request and what response should be sent, when a difference is detected:

.. code::
   
   Incoming request from the IoT Platform:
   
   Topic: iotdm-1/device/update
   Message: 
   {
      "reqId" : "f38faafc-53de-47a8-a940-e697552c3194",
      "d" : {
         "fields" : [{
               "field" : "mgmt.firmware",
               "value" : {
                  "version" : "some firmware version",
                  "name" : "some firmware name",
                  "uri" : "some uri for firmware location",
                  "verifier" : "some validation code",
                  "state" : 0,
                  "updateStatus" : 0,
                  "updatedDateTime" : ""
               }
            }
         ]
      }
   }
   
   Outgoing response from device:
   
   Topic: iotdevice-1/response
   Message: 
   {
      "rc" : 204,
      "reqId" : "f38faafc-53de-47a8-a940-e697552c3194"
   }   
   
This response will trigger the next request.  

|

In order to monitor the status of the firmware update the IoT Platform sends an observe request on the topic ``iotdm-1/observe``. 
When the device is ready to start the update process it sends a response with ``rc`` set to ``200``, ``mgmt.firmware.state`` set to ``0`` and ``mgmt.firmware.updateStatus`` set to ``0``.

Here is an example:

.. code::

   Incoming request from the IoT Platform:
   
   Topic: iotdm-1/observe
   Message:
   {
      "reqId" : "909b477c-cd37-4bee-83fa-1d568664fbe8",
      "d" : {
         "fields" : [
            {
               "field": "mgmt.firmware"
            }
         ]
      }
   }

   Outgoing response from device:
   
   Topic: iotdevice-1/response
   Message:
   {
      "rc" : 200,
      "reqId" : "909b477c-cd37-4bee-83fa-1d568664fbe8",
      "d" : {
         "fields" : [{
               "field" : "mgmt.firmware",
               "value" : {
                  "state" : 0,
                  "updateStatus" : 0
               }
            }
         ]
      }
   }

|


Afterwards the device management server in the IoT Platform uses the device management protocol to request that the devices specified initiate the firmware installation by publishing using the topic ``iotdm-1/mgmt/initiate/firmware/update``.
If this operation can be initiated immediately, ``rc`` should be set to ``202``.
If firmware was not previously downloaded successfully, ``rc`` should be set to ``400``.

Here is an example:

.. code::

   Incoming request from the IoT Platform:
   
   Topic: iotdm-1/mgmt/initiate/firmware/update
   Message:
   {
      "reqId" : "7b244053-c08e-4d89-9ed6-6eb2618a8734"
   }

   Outgoing response from device:
   
   Topic: iotdevice-1/response
   Message:
   {
      "rc" : 202,
      "reqId" : "7b244053-c08e-4d89-9ed6-6eb2618a8734"
   }

|
   
In order to finish the firmware update request the device has to report its update status to the IoT Platform via a status message published on its ``iotdevice-1/notify``-topic.
Once firmware update is completed, ``mgmt.firmware.updateStatus`` should be set to ``0`` (``Success``), ``mgmt.firmware.state`` should be set to ``0`` (``Idle``), downloaded firmware image can be deleted from the device and ``deviceInfo.fwVersion`` should be set to the value of ``mgmt.firmware.version``.

Here is an example notify message:

.. code:: 
   
   Outgoing message from device:
   
   Topic: iotdevice-1/notify
   Message:
   {
      "d" : {
         "fields": [ 
            {
               "field" : "mgmt.firmware",
               "value" : {
                  "state" : 0,
                  "updateStatus" : 0
               }
            }
         ]
      }
   }
 
|

After the IoT Platform received the notify of a completed firmware update it will trigger a last request on the ``iotdm-1/cancel``-topic for cancelation of the observation of the ``mgmt.firmware``-field.
After a response with ``rc`` set to ``200`` was sent the firmware update request is completed. Example:

.. code:: 

   Incoming request from the IoT Platform:
   
   Topic: iotdm-1/cancel
   Message:
   {
      "reqId" : "d9ca3635-64d5-46e2-93ee-7d1b573fb20f",
      "d" : {
         "fields" : [
            {
               "field" : "mgmt.firmware"
            }
         ]
      }
   }


   Outgoing message from device:
   
   Topic: iotdevice-1/response
   Message:
   {
      "rc" : 200,
      "reqId" : "d9ca3635-64d5-46e2-93ee-7d1b573fb20f"
   }

|
   
Useful information regarding error and process handling:

- If firmware update attempt fails, ``rc`` should be set to ``500`` and the ``message`` field can optionally be set to contain relevant information.
- If firmware update is not supported ``rc`` should be set to ``501`` and the ``message`` field can optionally be set to contain relevant information.
- If ``mgmt.firmware.state`` is not ``2`` (Downloaded), an error should be reported with ``rc`` set to ``400`` and an optional message text. 
- Otherwise, ``mgmt.firmware.updateStatus`` should be set to ``1`` (In Progress) and firmware installation should start. 
- If firmware installation fails, ``mgmt.firmware.updateStatus`` should be set to either:

  - ``2`` (Out of Memory)
  - ``5`` (Unsupported Image)
  


.. important:: All fields under ``mgmt.firmware`` must be set at the same time, so that if there is a current observation for ``mgmt.firmware``, only a single notify message is sent. 


