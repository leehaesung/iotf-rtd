Extending Device Management
===========================

.. important:: This feature is currently available as part of a limited beta.  Future updates 
  may include changes incompatible with the current version of this feature.  Try it out and `let us know what you 
  think <https://developer.ibm.com/answers/smart-spaces/17/internet-of-things.html>`_

The IBM Watson IoT Platform provides the ability to extend the device management capabilities. This is done by adding
device management extensions using either the REST APIs or the IoT Platform dashboard.

Devices which connect to the IoT Platform may have a wide range of capabilities. The device management actions
supported by default in the IoT Platform (*device reboot*, *factory reset*, *firmware download* and *firmware update*) 
may not be sufficient to manage everything which a device can do.


Device Management Extension Packages
------------------------------------

An extension package is a JSON document which defines a set of device management actions. The actions can be initiated
against one or more devices which support those actions. The actions are initiated in the same way as the default
device management actions by using either the IoT Platform dashboard or the device management REST APIs.

Device management extension packages have the following format:

.. code:: 

	{
		"bundleId": "<unique identifier>",
		"displayName": {
			"<locale 0>": "<localized display name 0>"
		},
		"description": {
			"<locale 0>": "<localized description 0>"
		},
		"version": "<bundle version>",
		"provider": "<bundle provider>",
		"actions": {
			"<actionId 0>": {
				"actionDisplayName": {
					"<locale 0>": "<localized action display name 0>"
				},
				"description": {
					"<locale 0>": "<localized description>"
				},
				"parameters": [
					{
						"name": "<parameterId>",
						"value": "<regex pattern for value checking>",
						"required": false,
						"defaultValue": "<default>"
					}
				]
			}
		}
	}


Extension Package Properties:

- ``bundleId``: Unique identifier for a device management extension. Required.
- ``version``: Version string for a device management extension. Optional.
- ``provider``: Provider string for a device management extension. Maximum length of 1024 characters. Optional.
- ``displayName``: Map of ``locale``: ``String`` key-value pairs used for display in the IoT Platform dashboard. Required. Must contain at least one entry.
- ``description``: Map of ``locale``: ``String`` key-value pairs used for display in the IoT Platform dashboard. Optional. If specified, must contain at least one entry.
- ``actions``: Map of ``actionId``: ``<action>`` key-value pairs which defines the actions contained within a device management extension. Required. Must contain at least one entry.

Properties per action:

- ``actionDisplayName``: Map of ``locale``: ``String`` key-value pairs used for display in the IoT Platform dashboard. Required. Must contain at least one entry.
- ``description``:  Map of ``locale``: ``String`` key-value pairs used for display in the IoT Platform dashboard. Optional. If specified, must contain at least one entry.
- ``parameters``: Array of parameters allowed for a particular action. Optional. If specified, must contain at least one entry.

Properties per action parameter:

- ``name``: Unique identifier for a parameter within an action. Required.
- ``value``: Regular expression used for validating parameter values when a request is initiated. Optional. If not specified, value will not be validated.
- ``required``: Boolean value specifying whether the parameter is required. Optional. Default: False
- ``defaultValue``: Value to be used if the parameter is not provided when a request is initiated. Optional.

.. note::

	The ``bundleId``, ``version``, ``actionId`` and ``parameterId`` values have the following restrictions:
	
      - Maximum length of 255 characters
      - Must comprise only alpha-numeric characters (a-z, A-Z, 0-9) and the following special characters:
  
        - dash (-)
        - underscore (_)
        - dot (.)
    
    
Extension packages are managed in the IoT Platform using the following REST APIs.

List all device management extension packages:

	``GET https://<orgId>.internetofthings.ibmcloud.com:443/api/v0002/mgmt/custom/bundle``
	
Create a new device management extension package:

	``POST https://<orgId>.internetofthings.ibmcloud.com:443/api/v0002/mgmt/custom/bundle``

Get a specific device management extension package:

	``GET https://<orgId>.internetofthings.ibmcloud.com:443/api/v0002/mgmt/custom/bundle/{bundleId}``
	
Update a device management extension package:

	``PUT https://<orgId>.internetofthings.ibmcloud.com:443/api/v0002/mgmt/custom/bundle/{bundleId}``
	
Delete a device management extension package:

	``DELETE https://<orgId>.internetofthings.ibmcloud.com:443/api/v0002/mgmt/custom/bundle/{bundleId}``
	
For more information on the REST APIs for device management extension packages refer to 
the `version 2 API documentation <../../swagger/v0002.html>`_.


Supporting Custom Device Management Actions
-------------------------------------------

Device management actions defined in an extension package may only be initiated against devices which support those actions. 
A device specifies what types of actions it supports when it publishes a manage request to the IoT Platform. 
In order to allow a device to receive custom actions defined in a particular extension package, the device must specify that 
extension's bundle identifier in the supports object when publishing a manage request.

.. code::

	Outgoing message from device:
	
	Topic: iotdevice-1/mgmt/manage
	{
		"d": {
			"supports": {
				"deviceActions": false,
				"firmwareActions": false,
				"<bundleId>": true
			}
		},
		"reqId": "<request id>"
	}
	
	Incoming response from server:
	
	Topic: iotdm-1/response
	{
		"rc": 200,
		"reqId": "<request id>"
	}


For additional information about device manage requests, refer to `Device Management Protocol <index.html>`__.


Initiating Custom Device Management Actions
-------------------------------------------

Custom device management actions are initiated using the same REST API as the default device management actions:

	``POST https://<orgId>.internetofthings.ibmcloud.com:443/api/v0002/mgmt/requests``
	
The following information must be provided when initiating a request:

- The action ``<bundleId>/<actionId>``
- A list of devices to initiate the action against, with a maximum of 5000 devices
- A list of parameters as defined in the custom action definition

The payload for initiating a request has the following format:

.. code:: json

	{
		"action": "<bundleId>/<actionId>",
		"devices": [
			{
				"typeId": "<deviceType 0>",
				"deviceId": "<deviceId 0>"
			}
		],
		"parameters": [
			{
				"name": "<parameter0>",
				"value": "<parameter0 value>"
			}
		]
	}


Handling Custom Device Management Actions
-----------------------------------------

When a custom action is initiated against a device, an MQTT message will be published to the device.
The message will contain any parameters that were specified as part of the request. When the device 
receives this message, it is expected to either execute the action or respond with an error code
indicating that it cannot complete the action at this time.

To indicate that the action was completed successfully, a device should publish a response with
``rc`` set to ``200``.

Below is an example exchange between the server and a device.

.. code:: 

	Incoming message from the server:
	
	Topic: iotdm-1/mgmt/custom/<bundleId>/<actionId>
	{
		"d": {
			"fields": [
				{
					"field": "<parameter0>",
					"value": "<parameter0 value>"
				}
			]
		},
		"reqId": "<request id>"
	}
	
	... device carries out the requested action ... 

	Outgoing message from the device:
	
	Topic: iotdevice-1/response
	{
		"rc": 200,
		"reqId": "<request id>"
	}


Example
-------

In this example, we will walk through defining a new device management extension and executing an action defined within
that extension.

Some company manufactures ``exampleDeviceType`` devices. Users of these devices have the ability to manage plug-ins
which run on the devices. To facilitate remote management of plug-ins on ``exampleDeviceType`` devices, the manufacturer
provides a device management extension which users can import into their Watson IoT Platform organization.

The extension JSON document used in this example:

.. code:: json

	{
		"bundleId": "exampleDeviceType-actions-v1",
		"displayName": {
			"en_US": "exampleDeviceType Actions v1"
		},
		"description": {
			"en_US": "Device management actions for exampleDeviceType devices"
		},
		"version": "1.0",
		"provider": "some company",
		"actions": {
			"installPlugin": {
				"actionDisplayName": {
					"en_US": "Install Plug-in"
				},
				"description": {
					"en_US": "Install a new plug-in on the device"
				},
				"parameters": [
					{
						"name": "pluginId",
						"value": "\\w+",
						"required": true
					},
					{
						"name": "pluginURI",
						"value": "((http:\\/\\/|https:\\/\\/)(.*)+)",
						"required": true
					}
				]
			},
			"enablePlugin": {
				"actionDisplayName": {
					"en_US": "Enable Plug-in"
				},
				"description": {
					"en_US": "Enables a plug-in on the device" 
				},
				"parameters": [
					{
						"name": "pluginId",
						"value": "\\w+",
						"required": true
					}
				]
			},
			"disablePlugin": {
				"actionDisplayName": {
					"en_US": "Disable Plug-in"
				},
				"description": {
					"en_US": "Disables a plug-in on the device"
				},
				"parameters": [
					{
						"name": "pluginId",
						"value": "\\w+",
						"required": true
					}
				]
			},
			"uninstallPlugin": {
				"actionDisplayName": {
					"en_US": "Uninstall Plug-in"
				},
				"description": {
					"en_US": "Uninstall a plug-in from the device"
				},
				"parameters": [
					{
						"name": "pluginId",
						"value": "\\w+",
						"required": true
					}
				]
			}
		}
	}
	
In this device management extension package, 4 actions are defined:

- installPlugin
- enablePlugin
- disablePlugin
- uninstallPlugin

The extension is added using the REST API:

	``POST https://<orgId>.internetofthings.ibmcloud.com:443/api/v0002/mgmt/custom/bundle``
	
Devices registered in organization ``<orgId>`` are now able to specify that they support ``exampleDeviceType-actions-v1``
actions when publishing a manage request.

In this example, the manage request sent by a device would look like this:

.. code::

	Outgoing message from device:
	
	Topic: iotdevice-1/mgmt/manage
	{
		"d": {
			"supports": {
				"exampleDeviceType-actions-v1": true
			}
		},
		"reqId": "f38faafc-53de-47a8-a940-e697552c3194"
	}
	
The device should then receive the following response from the IoT Platform:

.. code::

	Incoming message from server:
	
	Topic: iotdm-1/response
	{
		"rc": 200,
		"reqId": "f38faafc-53de-47a8-a940-e697552c3194"
	}
	
At this point, an action defined in the ``exampleIoT-exampleDeviceType-v1`` extension can be initiated against some devices.

The payload for initiating an ``installPlugin`` action will look like the following:

.. code:: json

	{
		"action": "exampleDeviceType-actions-v1/installPlugin",
		"devices": [
			{
				"typeId": "exampleDeviceType",
				"deviceId": "device0"
			},
			{
				"typeId": "exampleDeviceType",
				"deviceId": "device1"
			}
		],
		"parameters": [
			{
				"name": "pluginId",
				"value": "testPluginA"
			},
			{
				"name": "pluginURI",
				"value": "http://www.example.com/testPluginA.zip"
			}
		]
	}
	
The request is initiated using the REST API:

.. code::

	POST https://<orgId>.internetofthings.ibmcloud.com:443/api/v0002/mgmt/requests
	
Devices ``device0`` and ``device1`` of type ``exampleDeviceType`` will then receive the following MQTT message:

.. code:: 

	Incoming message from server:
	
	Topic: iotdm-1/mgmt/custom/exampleDeviceType-actions-v1/installPlugin
	{
		"d": {
			"fields": [
				{
					"field": "pluginId",
					"value": "testPluginA"
				},
				{
					"field": "pluginURI",
					"value": "http://www.exampleiot.com/testPluginA.zip"
				}
			]
		},
		"reqId": "d38faafc-53de-47a8-a940-e697552c3194"
	}
	
	
Each device will take action on the message, installing the new plug-in. Once installation is complete, the devices
respond with the following message to indicate that the action completed successfully:

.. code:: 

	Outgoing message from device:
	
	Topic: iotdevice-1/response
	{
		"rc": 200,
		"reqId": "d38faafc-53de-47a8-a940-e697552c3194"
	}
	
At this point, the ``installPlugin`` action is now complete.


API Examples
------------

The following are examples of API requests that may be useful when performing device management actions:

Create a new device management extension:
	
	``curl -XPOST -d '<insert payload here>' -H "Content-Type: application/json" -u "<apiKey>:<apiToken>" https://<orgId>.internetofthings.ibmcloud.com:443/api/v0002/mgmt/custom/bundle``

List device management extensions:

	``curl -XGET -H "Content-Type: application/json" -u "<apiKey>:<apiToken>" https://<orgId>.internetofthings.ibmcloud.com:443/api/v0002/mgmt/custom/bundle``

Initiate a device management request:
	
	``curl -XPOST -d '<insert payload here>' -H "Content-Type: application/json" -u "<apiKey>:<apiToken>" https://<orgId>.internetofthings.ibmcloud.com:443/api/v0002/mgmt/requests``
	
List in-progress or completed device management requests:

	``curl -XGET -H "Content-Type: application/json" -u "<apiKey>:<apiToken>" https://<orgId>.internetofthings.ibmcloud.com:443/api/v0002/mgmt/requests``
	
View status of a particular device management request:

	``curl -XGET -H "Content-Type: application/json" -u "<apiKey>:<apiToken>" https://<orgId>.internetofthings.ibmcloud.com:443/api/v0002/mgmt/requests/<requestId>``