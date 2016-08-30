Python for Application Developers
=================================

- See `iot-python <https://github.com/ibm-messaging/iot-python>`_ in GitHub
- See `ibmiotf <https://pypi.python.org/pypi/ibmiotf>`_ on PyPi

Constructor
-----------

The constructor builds the client instance, and accepts an options dict containing the following definitions:

- ``org`` - Your organization ID.
- ``id`` - The unique ID of your application within your organization.
- ``auth-method`` - Method of authentication (the only value currently supported is ``apikey``).
- ``auth-key`` - API key (required if auth-method is ``apikey``).
- ``auth-token`` - API key token (required if auth-method is ``apikey``).

If no options dict is provided, the client will connect to the IoT Platform Quickstart, and default to an unregistered device. The options dict creates definitions which are used to interact with the IoT Platform module.

.. code:: python

    import ibmiotf.application
    try:
      options = {
        "org": organization, 
        "id": appId, 
        "auth-method": authMethod, 
        "auth-key": authKey, 
        "auth-token": authToken
      }
      client = ibmiotf.application.Client(options)
    except ibmiotf.ConnectionException  as e:
      ...


Using a configuration file
~~~~~~~~~~~~~~~~~~~~~~~~~~

If you are not using an options dict as shown above, you include a configuration file containing an options dict. If you are using a configuration file containing an options dict, use the following code format.

.. code:: python

    import ibmiotf.application
    try:
      options = ibmiotf.application.ParseConfigFile(configFilePath)
      client = ibmiotf.application.Client(options)
    except ibmiotf.ConnectionException  as e:
      ...

The application configuration file must be in the following format:

::

    [application]
    org=$orgId
    id=$myApplication
    auth-method=apikey
    auth-key=$key
    auth-token=$token


----


API calls
---------

Each method in the APIClient responds with either a valid response (JSON or boolean) in the case of success or IoTFCReSTException in the case of failure. The IoTFCReSTException contains the following properties that application can parse to get more information about the failure.

* httpcode - HTTP Status Code
* message - Exception message containing the reason for the failure
* response - JsonElement containing the partial response if any otherwise null

So in the case of failure, application needs to parse the response to see if the action is partially successful or not.


----


Subscribing to device events
-------------------------------------------------------------------------------
Events are the mechanism by which devices publish data to the IoT Platform. The device controls the content of the event and assigns a name for each event it sends.

When an event is received by the IoT Platform the credentials of the connection on which the event was received are used to determine from which device the event was sent. With this architecture it is impossible for a device to impersonate another device.

By default, applications will subscribe to all events from all connected devices. Use the type, id, event and msgFormat parameters to control the scope of the subscription. A single client can support multiple subscriptions. The code samples below give examples of how to subscribe to devices dependent on device type, id, event and msgFormat parameters.


To subscribe to all events from all devices
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code:: python

    import ibmiotf.application
    
    options = ibmiotf.application.ParseConfigFile(configFilePath)
    client = ibmiotf.application.Client(options)
    
    client.connect()
    client.subscribeToDeviceEvents()


To subscribe to all events from all devices of a specific type
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code:: python

    import ibmiotf.application
    
    options = ibmiotf.application.ParseConfigFile(configFilePath)
    client = ibmiotf.application.Client(options)
    
    client.connect()
    client.subscribeToDeviceEvents(deviceType=myDeviceType)


To subscribe to a specific event from all devices
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code:: python

    import ibmiotf.application
    
    options = ibmiotf.application.ParseConfigFile(configFilePath)
    client = ibmiotf.application.Client(options)
    
    client.connect()
    client.subscribeToDeviceEvents(event=myEvent)


To subscribe to a specific event from two or more different devices
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code:: python

    import ibmiotf.application
    
    options = ibmiotf.application.ParseConfigFile(configFilePath)
    client = ibmiotf.application.Client(options)
    
    client.connect()
    client.subscribeToDeviceEvents(deviceType=myDeviceType, deviceId=myDeviceId, event=myEvent)
    client.subscribeToDeviceEvents(deviceType=myOtherDeviceType, deviceId=myOtherDeviceId, event=myEvent)


To subscribe to all events published by a device in json format
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code:: python

    import ibmiotf.application
    
    options = ibmiotf.application.ParseConfigFile(configFilePath)
    client = ibmiotf.application.Client(options)
    
    client.connect()
    client.subscribeToDeviceEvents(deviceType=myDeviceType, deviceId=myDeviceId, msgFormat="json")


----


Handling events from devices
-------------------------------------------------------------------------------
To process the events received by your subscriptions you need to register an event callback method. The messages are returned as an instance of the Event class:

* event.device - string (uniquely identifies the device across all types of devices in the organization)
* event.deviceType - string
* event.deviceId - string
* event.event - string
* event.format - string
* event.data - dict
* event.timestamp - datetime

.. code:: python

    import ibmiotf.application
    
    options = ibmiotf.application.ParseConfigFile(configFilePath)
    client = ibmiotf.application.Client(options)
    
    def myEventCallback(event):
      str = "%s event '%s' received from device [%s]: %s"
      print(str % (event.format, event.event, event.device, json.dumps(event.data)))

    ...
    client.connect()
    client.deviceEventCallback = myEventCallback
    client.subscribeToDeviceEvents()


----


Subscribing to device status
-------------------------------------------------------------------------------
By default, this will subscribe to status updates for all connected devices. Use the type and id parameters to control the scope of the subscription. A single client can support multiple subscriptions.

Subscribe to status updates for all devices
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code:: python

    import ibmiotf.application
    
    options = ibmiotf.application.ParseConfigFile(configFilePath)
    client = ibmiotf.application.Client(options)
    
    client.connect()
    client.subscribeToDeviceStatus()


Subscribe to status updates for all devices of a specific type
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code:: python

    import ibmiotf.application
    
    options = ibmiotf.application.ParseConfigFile(configFilePath)
    client = ibmiotf.application.Client(options)
    
    client.connect()
    client.subscribeToDeviceStatus(deviceType=myDeviceType)


Subscribe to status updates for two different devices
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code:: python

    import ibmiotf.application
    
    options = ibmiotf.application.ParseConfigFile(configFilePath)
    client = ibmiotf.application.Client(options)
    
    client.connect()
    client.subscribeToDeviceStatus(deviceType=myDeviceType, deviceId=myDeviceId)
    client.subscribeToDeviceStatus(deviceType=myOtherDeviceType, deviceId=myOtherDeviceId)


----


Handling status updates from devices
-------------------------------------------------------------------------------
To process the status updates received by your subscriptions you need to register an event callback method. The messages are returned as an instance of the Status class:

The following properties are set for both "Connect" and "Disconnect" status events:
  
* status.clientAddr - string
* status.protocol - string
* status.clientId - string
* status.user - string
* status.time - datetime
* status.action - string
* status.connectTime - datetime
* status.port - integer

The following properties are only set when the action is "Disconnect":

* status.writeMsg - integer
* status.readMsg - integer
* status.reason - string
* status.readBytes - integer
* status.writeBytes - integer

.. code:: python

    import ibmiotf.application
    
    options = ibmiotf.application.ParseConfigFile(configFilePath)
    client = ibmiotf.application.Client(options)
    
    def myStatusCallback(status):
      if status.action == "Disconnect":
        str = "%s - device %s - %s (%s)"
        print(str % (status.time.isoformat(), status.device, status.action, status.reason))
      else:
        print("%s - %s - %s" % (status.time.isoformat(), status.device, status.action))

    ...
    client.connect()
    client.deviceStatusCallback = myStatusCallback
    client.subscribeToDeviceStstus()


----


Publishing events from devices
-------------------------------------------------------------------------------
Applications can publish events as if they originated from a Device

.. code:: python

    import ibmiotf.application
    
    options = ibmiotf.application.ParseConfigFile(configFilePath)
    client = ibmiotf.application.Client(options)
    
    client.connect()
    myData={'name' : 'foo', 'cpu' : 60, 'mem' : 50}
    client.publishEvent(myDeviceType, myDeviceId, "status", "json", myData)


----


Publishing commands to devices
-------------------------------------------------------------------------------
Applications can publish commands to connected devices

.. code:: python

    import ibmiotf.application
    
    options = ibmiotf.application.ParseConfigFile(configFilePath)
    client = ibmiotf.application.Client(options)
    
    client.connect()
    commandData={'rebootDelay' : 50}
    client.publishCommand(myDeviceType, myDeviceId, "reboot", "json", myData)


----


Organization details
--------------------

Applications can use the ``getOrganizationDetails()`` method to retrieve the details about the configuration of the organization.

.. code:: python

    import ibmiotf.application
    
    options = ibmiotf.application.ParseConfigFile(configFilePath)
    client = ibmiotf.application.Client(options)
    
    try:
        orgDetail = client.api.getOrganizationDetails()
    except IoTFCReSTException as e:
        print("ERROR [" + e.httpcode + "] " + e.message)
    

Refer to the Organization Configuration section of the `IBM IoT Platform API <https://docs.internetofthings.ibmcloud.com/swagger/v0002.html>`__ for information about the request & response model and http status code.


----



Bulk device operations
----------------------

Applications can use bulk operations to get, add or remove devices in bulk.

Refer to the Bulk Operations section of the `IBM IoT Platform API <https://docs.internetofthings.ibmcloud.com/swagger/v0002.html#!/Bulk_Operations/>`__ for information about the list of query parameters, the request & response model and http status code.

Retrieve device information
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Bulk device information can be retrieved using the ``getAllDevices()`` method. This method retrieves information on all 
registered devices in the organization, each request can contain a maximum of 512KB.

The response contains parameters required by the application to retrieve the dictionary *results* from the response to 
get the array of devices returned. Other parameters in the response are required to make further calls, for example, 
the ``_bookmark`` element can be used to page through results. Issue the first request without specifying a bookmark, 
then take the bookmark returned in the response and provide it on the request for the next page. Repeat until the 
end of the result set indicated by the absence of a bookmark. Each request must use exactly the same values for the 
other parameters, or the results are undefined.


.. code:: python

    import ibmiotf.application
    
    options = ibmiotf.application.ParseConfigFile(configFilePath)
    client = ibmiotf.application.Client(options)
    
    try:
        deviceList = client.api.getAllDevices()
    except IoTFCReSTException as e:
        print("ERROR [" + e.httpcode + "] " + e.message)


Add Devices in bulk
~~~~~~~~~~~~~~~~~~~

The ``addMultipleDevices()`` method can be used to add one or more devices to your IoT Platform 
organization, the maximum size of a request is set to 512KB.  The response will contain the generated authentication 
tokens for each added device. These authentication tokens must be recorded when processing the response, as lost 
authentication tokens cannot be retrieved.


.. code:: python

    import ibmiotf.application
    
    options = ibmiotf.application.ParseConfigFile(configFilePath)
    client = ibmiotf.application.Client(options)
    
    listOfDevicesToAdd = [
        {'typeId' : "pi-model-a", 'deviceId' : '200020002004'}, 
        {'typeId' : "pi-model-b", 'deviceId' : '200020002005'}
    ]
    
    try:
        deviceList = client.api.addMultipleDevices(listOfDevicesToAdd)
    except IoTFCReSTException as e:
        print("ERROR [" + e.httpcode + "] " + e.message)


Delete Devices in bulk
~~~~~~~~~~~~~~~~~~~~~~~~

The ``deleteMultipleDevices()`` method can be used to delete multiple devices from an IoT Platform 
organization, each request can contain a maximum of 512KB.

.. code:: python
    
    import ibmiotf.application
    
    options = ibmiotf.application.ParseConfigFile(configFilePath)
    client = ibmiotf.application.Client(options)
    
    listOfDevicesToDelete = [
        {'typeId' : "pi-model-a", 'deviceId' : '200020002004'}, 
        {'typeId' : "pi-model-b", 'deviceId' : '200020002005'}
    ]
    
    try:
        deviceList = client.api.deleteMultipleDevices(listOfDevicesToDelete)
    except IoTFCReSTException as e:
        print("ERROR [" + e.httpcode + "] " + e.message)


----


Device Type Operations
----------------------

Device types can be used as templates for adding device information to devices as they are added to 
your organization. Applications can use the IoT Platform API to list, create, 
delete, view, or update device types in your organization.

Refer to the Device Types section of the `IBM IoT Platform API <https://docs.internetofthings.ibmcloud.com/swagger/v0002.html>`__ documentation 
for information about the list of query parameters, the request & response model, and http status codes.


Get all Device Types
~~~~~~~~~~~~~~~~~~~~~~~~

The ``getAllDeviceTypes()`` method can be used to retrieve all device types in your IoT Platform 
organization. The response contains parameters and application needs to retrieve the dictionary *results* from the 
response to get the array of devices returned. Other parameters in the response are required to make further call, 
for example, the *_bookmark* element can be used to page through results. Issue the first request without 
specifying a bookmark, then take the bookmark returned in the response and provide it on the request for the next page. 
Repeat until the end of the result set indicated by the absence of a bookmark. Each request must use exactly the 
same values for the other parameters, or the results are undefined.


.. code:: python

    import ibmiotf.application
    
    options = ibmiotf.application.ParseConfigFile(configFilePath)
    client = ibmiotf.application.Client(options)
    
    listOfDevicesToAdd = [
        {'typeId' : "pi-model-a", 'deviceId' : '200020002004'}, 
        {'typeId' : "pi-model-b", 'deviceId' : '200020002005'}
    ]
    
    try:
        options = {'_limit' : 2}
        deviceTypeList = client.api.getAllDeviceTypes(options)
    except IoTFCReSTException as e:
        print("ERROR [" + e.httpcode + "] " + e.message)    


Add a Device Type
~~~~~~~~~~~~~~~~~

The ``addDeviceType()`` method can be used to register a device type to IoT Platform. In 
each request, you must first define the device information, and device metadata elements which you want to be applied 
to all devices of this type. The device information element is comprised of several variables, including, serial 
number, manufacturer, model, class, description, firmware and hardware versions, and descriptive location. The 
metadata element is comprised of custom variables and values which can be defined by the user.


.. code:: python

    import ibmiotf.application
    
    options = ibmiotf.application.ParseConfigFile(configFilePath)
    client = ibmiotf.application.Client(options)
    
    info = {
        "serialNumber": "100087", 
        "manufacturer": "ACME Co.", 
        "model": "7865", 
        "deviceClass": "A", 
        "description": "My shiny device", 
        "fwVersion": "1.0.0", 
        "hwVersion": "1.0", 
        "descriptiveLocation": "Office 5, D Block"
    }
    meta = {
        "customField1": "customValue1", 
        "customField2": "customValue2"
    }
    
    try:
        deviceType = client.api.addDeviceType(deviceType = "myDeviceType", description = "My first device type", deviceInfo = info, metadata = meta)
    except IoTFCReSTException as e:
        print("ERROR [" + e.httpcode + "] " + e.message)    


Delete a Device Type
~~~~~~~~~~~~~~~~~~~~

The ``deleteDeviceType()`` method can be used to delete a device type from your IoT Platform 
organization.

.. code:: python

    import ibmiotf.application
    
    options = ibmiotf.application.ParseConfigFile(configFilePath)
    client = ibmiotf.application.Client(options)
    
    try:
        success = client.api.deleteDeviceType("myDeviceType")
    except IoTFCReSTException as e:
        print("ERROR [" + e.httpcode + "] " + e.message)    


Get a Device Type
~~~~~~~~~~~~~~~~~

The ``getDeviceType()`` method retrieves information on a given device type. The ``typeId`` of the device type you wish to retrieve must be used 
as a parameter

.. code:: python

    import ibmiotf.application
    
    options = ibmiotf.application.ParseConfigFile(configFilePath)
    client = ibmiotf.application.Client(options)
    
    try:
        deviceTypeInfo = client.api.getDeviceType("myDeviceType")
    except IoTFCReSTException as e:
        print("ERROR [" + e.httpcode + "] " + e.message)    


Update a Device Type
~~~~~~~~~~~~~~~~~~~~~~~~

The ``updateDeviceType()`` method can be used to modify the properties of a device type. When using this method, several parameters 
must be defined. Firstly, the ``typeId`` of the device type to be updated must be specified, then the ``description``, ``deviceInfo``, and ``metadata`` elements.

.. code:: python

    import ibmiotf.application
    
    options = ibmiotf.application.ParseConfigFile(configFilePath)
    client = ibmiotf.application.Client(options)
    
    info = {
        "serialNumber": "100087", 
        "manufacturer": "ACME Co.", 
        "model": "7865", 
        "deviceClass": "A", 
        "description": "My shiny device", 
        "fwVersion": "1.0.0", 
        "hwVersion": "1.0", 
        "descriptiveLocation": "Office 5, D Block"
    }
    meta = {
        "customField1": "customValue1", 
        "customField2": "customValue2",
        "customField3": "customValue3"
    }
    
    try:
        updatedDeviceTypeInfo = client.api.updateDeviceType("myDeviceType", "New description", deviceInfo, metadata)
    except IoTFCReSTException as e:
        print("ERROR [" + e.httpcode + "] " + e.message)    


----


Device operations
-----------------

Device operations made available through the API include listing, adding, removing, viewing, updating, viewing location and viewing  device management information of devices in an IoT Platform organization.

Refer to the Device section of the `IoT Platform API <https://docs.internetofthings.ibmcloud.com/swagger/v0002.html>`__ for information about the list of query parameters, the request & response model and http status code.

Get Devices of a particular Device Type
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The *retrieveDevices()* method can be used to retrieve all the devices of a particular device type in an organization from IoT Platform. For example,

.. code:: python

     print("\nRetrieving All existing devices")	
     print("Retrieved Devices = ", apiCli.retrieveDevices(deviceTypeId))
    
The response contains parameters and application needs to retrieve the dictionary *results* from the response to get the array of devices returned. Other parameters in the response are required to make further call, for example, the *_bookmark* element can be used to page through results. Issue the first request without specifying a bookmark, then take the bookmark returned in the response and provide it on the request for the next page. Repeat until the end of the result set indicated by the absence of a bookmark. Each request must use exactly the same values for the other parameters, or the results are undefined.

In order to pass the *_bookmark* or any other condition, the overloaded method must be used. The overloaded method takes the parameters in the form of dictionary as shown below,

.. code:: python

    response = apiClient.retrieveDevices("iotsample-ardunio", parameters);
		
The above snippet sorts the response based on device id and uses the bookmark to page through the results.

Add a Device
~~~~~~~~~~~~~~~~~~~~~~~

The *registerDevice()* method is used to add a device to an IoT Platform organization. The *registerDevice()* method adds a single device to your IoT Platform organization. The parameters which can be set when adding a device are:

- deviceTypeId: *Optional*. Assigns a device type to the device. Where there is a clash between variables defined by the device type and variables defined by under deviceInfo, the device specific variables will take precedence.
- deviceId: *Mandatory*. 
- authToken: *Optional*. If no authentication token is supplied, one will be generated and included in the response.
- deviceInfo: *Optional*. This parameter is optional, and can contain a number of variables, including: serialNumber, manufacturer, model, deviceClass, description, firmware and hardware versions, and descriptiveLocation.
- metadata: *Optional*. Metadata can optionally be added in the form of custom field:value string pairs. An example is given in the code sample below.
- location: *Optional*. This parameter contains the longitude, latitude, elevation, accuracy, and mesauredDateTime variables.

For more information on the parameters presented here, and the response format and codes, please see the relevant section of `API documentation <https://docs.internetofthings.ibmcloud.com/swagger/v0002.html#!/Devices/post_device_types_typeId_devices>`__. 

When using the *registerDevice()* method, you must define the mandatory deviceID parameter, and any of the optional parameters you require for your device, then call the method using the parameters you've selected.

Sample
~~~~~~~

The following code sample should be inserted after the constructor code in a .py file. This code demonstrates defining the deviceId, authToken, metadata, deviceInfo parameters, and location parameters and then using the method with those parameters and adding a device type.

.. code:: python

    deviceId = "200020002000"
    authToken = "password"
    metadata = {"customField1": "customValue1", "customField2": "customValue2"}
    deviceInfo = {"serialNumber": "001", "manufacturer": "Blueberry", "model": "abc1", "deviceClass": "A", "descriptiveLocation" : "Bangalore", "fwVersion" : "1.0.1", "hwVersion" : "12.01"}
    location = {"longitude" : "12.78", "latitude" : "45.90", "elevation" : "2000", "accuracy" : "0", "measuredDateTime" : "2015-10-28T08:45:11.662Z"}
	
    apiCli.registerDevice(deviceTypeId, deviceId, metadata, deviceInfo, location)


Delete a Device
~~~~~~~~~~~~~~~~~~~~~~~~

The *deleteDevice()* method is used to remove a device from an IoT Platform organization. When deleting a device using this method, the parameters which must be specified in the method are the deviceTypeId, and the deviceId.

The following code snippet provides an example of the format required for this method.

.. code:: java

    apiCli.deleteDevice(deviceTypeId, deviceId)

    
Get a Device
~~~~~~~~~~~~~~~~~~~~~~~~

The *getDevice()* method can be used to retrieve a device from an IoT Platform organization. When retrieving device details using this method, the parameters which must be specified in the method are the deviceTypeId, and the deviceId.

The following code snippet provides an example of the format required for this method.

.. code:: python

	apiCli.getDevice(deviceTypeId, deviceId)
    

Get all Devices
~~~~~~~~~~~~~~~~~~~~~~~~

The *getAllDevices()* method can be used to retrieve all devices within an IoT Platform organization.

.. code:: python

	apiCli.getAllDevices({'typeId' : deviceTypeId})


Update a Device
~~~~~~~~~~~~~~~~~~~~~~~~

The *updateDevice()* method is used to modify one or more properties of a device. Any property in the deviceInfo or metadata parameters can be updated. In order to update a device property, it must be defined above the method. The status parameter should contain "alert": True. The Alert property controls whether a device will display error codes in the IoT Platform user interface, and should be set by default to 'True'.

.. code:: python
    
    
    status = { "alert": { "enabled": True }  }
    apiCli.updateDevice(deviceTypeId, deviceId, metadata2, deviceInfo, status)

Sample
~~~~~~~

In this sample, the following code identifies a specific device, and updates several properties under the deviceInfo parameter.

.. code:: python

	status = { "alert": { "enabled": True } }
	deviceInfo = {descriptiveLocation: "London", hwVersion: "2.0.1", fwVersion: "2.5.1"}
    apiCli.updateDevice("MyDeviceType", "200020002000", deviceInfo, status)	
	
Get Location Information
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The *getDeviceLocation()* method can be used to retrieve the location information of a device. The parameters required for retrieving the location data are deviceTypeId and deviceId. 

.. code:: python
    
	apiClient.getDeviceLocation("iotsample-ardunio", "ardunio01")

The response to this method contains the longitude, latitude, elevation, accuracy, measuredTimeStamp, and updatedTimeStamp properties.	
	
	
Update Location Information
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The *updateDeviceLocation()* method is used to modify the location information for a device. Simlarly to updating device properties, the deviceLocation parameter must be defined with the changes you wish to apply. The code sample below demonstrates changing the location data for a given device.

.. code:: python
    
    deviceLocation = { "longitude": 0, "latitude": 0, "elevation": 0, "accuracy": 0, "measuredDateTime": "2015-10-28T08:45:11.673Z"}
    apiCli.updateDeviceLocation(deviceTypeId, deviceId, deviceLocation)

If no date is supplied, the entry is added with the current date and time. 


Get Device Management Information
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The *getDeviceManagementInformation()* method is used to get the device management information for a device. The response contains the last activity date-time, the device's dormant status (true/false), support for device and firmware actions, and firmware data. For a comprehensive list of response content, please see the relevant API documentation.

The following code sample will return the device management information for a device with the deviceId "00aabbccde03", with deviceTypeId "iotsample-arduino".

Sample
~~~~~~~~~

.. code:: python
    
    apiCli.getDeviceManagementInformation("iotsample-arduino", "00aabbccde03")
    

----

Device diagnostic operations
----------------------------------------------------

Applications can use device diagnostic operations to clear logs, retrieve all or specific logs for a device, add log information, delete logs, clear error codes, get device error codes, and add an error codes.

For more detailed information on query and response models, response codes, and query paramters, please see the relevant API documentation.

Get Diagnostic logs
~~~~~~~~~~~~~~~~~~~~~~

The *getAllDiagnosticLogs()* method is used to retrieve all diagnostic logs for a specific device. The *getAllDiagnosticLogs()* method requires the deviceTypeId and deviceId parameters.

.. code:: python

    apiCli.getAllDiagnosticLogs(deviceTypeId, deviceId)
    
The response model for this method contains the logId, message, severity, data, and timestamp.

Clear Diagnostic logs for a Device 
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The *clearAllDiagnosticLogs()* method is used to delete all diagnostic logs for a specific device. The required parameters are deviceTypeId and deviceId. Care should be taken when deleting logs, as logs cannot be recovered once deleted.

.. code:: python

    apiCli.clearAllDiagnosticLogs(deviceTypeId, deviceId)
    

Add a Diagnostic log
~~~~~~~~~~~~~~~~~~~~~~

The *addDiagnosticLog()* method is used to add an entry in the diagnostic log of the device. The log may be pruned as the new entry is added. If no date is supplied, the entry is added with the current date and time. To use this method, first define a 'logs' parameter with the following variables:

- message: This variable is mandatory, and contains the new diagnostic message.
- severity: This variable is optional. If used it corresponds to the severity of the diagnostic log, and should be 0, 1, or 2, corresponding to the informational, warning, and error categories.
- data: This variable is optional, and should contain diagnostic data.
- timestamp: This variable is optional, and contains the date and time of the log entry in ISO8601 format. If this variable is not included, it is automatically added with the current date and time. 

The other necessary paramteres required in the method are the deviceTypeId and deviceId for the specific device.

The code sample below contains an example of the method.

.. code:: python

    logs = { "message": "MessageContent", "severity": 0, "data": "LogData"}
    apiCli.addDiagnosticLog(deviceTypeId, deviceId, logs)

	
Retrieve a specific Diagnostic log
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The *getDiagnosticLog()* method is used to retrieve a specific diagnostic log for a specified device based on the log id. The required parameters for this method are the deviceTypeId, deviceId, and logId.

.. code:: python

    apiCli.getDiagnosticLog(deviceTypeId, deviceId, logId)
    
	
Delete a Diagnostic log
~~~~~~~~~~~~~~~~~~~~~~~~~~

The *deleteDiagnosticLog()* can be used to delete a specific diagnostic log. In order to specify a diagnostic log, the deviceTypeId, deviceId, and logId parameters should be supplied.

.. code:: python

	apiCli.deleteDiagnosticLog(deviceTypeId, deviceId, logId)
    

Retrieve Device Error Codes
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The *getAllDiagnosticErrorCodes()* method is used to retrieve all diagnostic error codes associated with a specific device.

.. code:: python

	apiCli.getAllDiagnosticErrorCodes(deviceTypeId, deviceId)
	
		
Clear Diagnostic Error Codes
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The *clearAllErrorCodes()* method is used to clear the list of error codes associated with the device. The list is replaced with a single error code of zero.

.. code:: python

    apiCli.clearAllErrorCodes(deviceTypeId, deviceId)
	

Add single Diagnostic ErrorCode
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The *addErrorCode()* method is used to add an error code to the list of error codes associated with the device. The list may be pruned as the new entry is added. The parameters required in the method are deviceTypeId, deviceId, and errorCode. The errorCode parameter contains the following variables:

- errorCode: This variable is mandatory and should be set as an integer. This sets the number of the error code to be created.
- timestamp: This variable is optional, and contains the date and time of the log entry in ISO8601 format. If this variable is not included, it is automatically added with the current date and time. 

.. code:: python

    errorCode = { "errorCode": 1234, "timestamp": "2015-10-29T05:43:57.112Z" }
    apiCli.addErrorCode(deviceTypeId, deviceId, errorCode)

----

Connection problem determination
----------------------------------

The *getDeviceConnectionLogs()* method is used to list connection log events for a device. This information can be used to help diagnose connectivity problems between the device and the IoT Platform service. The entries record successful connection, unsuccessful connection attempts, intentional disconnection and server-initiated disconnection events.

.. code:: python

	apiCli.getDeviceConnectionLogs(deviceTypeId, deviceId)

The response includes a list of log entries, containing log messages and timestamps. 

