Node.js for Device Developers
=============================

- See `iot-nodejs <https://github.com/ibm-messaging/iot-nodejs>`_ in GitHub
- See the `samples for device <https://github.com/ibm-messaging/iot-nodejs/tree/master/samples>`_ in Github
- See `ibmiotf <https://www.npmjs.com/package/ibmiotf>`_ on NPM

Constructor
--------------

The constructor builds the device client instance. It accepts a configuration json containing the following definitions:

- ``org`` - Your organization ID
- ``type`` - The type of your device
- ``id`` - The ID of your device
- ``auth-method`` - Method of authentication (the only value currently supported is ``token``)
- ``auth-token`` - API key token (required if auth-method is ``token``)

If you want to use Quickstart, then send only the first three properties.

.. code:: javascript

    var iotf = require("ibmiotf");
    var config = {
		"org" : "organization",
		"id" : "deviceId",
		"type" : "deviceType",
		"auth-method" : "token",
		"auth-token" : "authToken"
    };


    var deviceClient = new iotf.IotfDevice(config);

Using a configuration file
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Instead of passing the configuration json directly, you can also use a configuration json file. Use the following code snippet:

.. code:: javascript

    var iotf = require("ibmiotf");
    var config = require("./device.json");
    var deviceClient = new iotf.IotfDevice(config);

The configuration file `device.json` must be in the format of

.. code:: javascript

	{
	  "org": "xxxxx",
	  "type": "raspi",
	  "id": "pi1",
	  "auth-method" : "token",
	  "auth-token" : "xxxxxxxxxxxxxxxx"
	}

Connecting to the IoT Platform
-----------------------------------------------------

Connect to the IoT Platform by calling the *connect* function.

.. code:: javascript

	var iotf = require("ibmiotf");
	var config = require("./device.json");

	var deviceClient = new iotf.IotfDevice(config);

	//setting the log level to debug. By default its 'warn'
	deviceClient.log.setLevel('debug');

	deviceClient.connect();

	deviceClient.on('connect', function(){ 
		var i=0;
		console.log("connected");
		setInterval(function function_name () {
			i++;
			deviceClient.publish('myevt', 'json', '{"value":'+i+'}', 2);
		},2000);
	});

After the successful connection to the IoT Platform service, the device client sends a *connect* event. So all the device logic can be implemented inside this callback function.

Logging
-------

By default, all the logs of ``warn`` are logged. If you want to enable
more logs, use the *log.setLevel* function. Supported log levels -
*trace, debug, info, warn, error*.

.. code:: javascript

	var iotf = require("ibmiotf");
	var config = require("./device.json");

	var deviceClient = new iotf.IotfDevice(config);

	//setting the log level to debug. By default its 'warn'
	deviceClient.log.setLevel('debug');


Publishing events
------------------

Events are the mechanism by which devices publish data to the IoT Platform. The device controls the content of the event and assigns a name for each event it sends.

When an event is received by the IoT Platform the credentials of the connection on which the event was received are used to determine from which device the event was sent. With this architecture it is impossible for a device to impersonate another device.

Events can be published at any of the three quality of service levels defined by the MQTT protocol. By default events will be published as QoS level 0. Please not that if you are using the Internet of Things Quickstart service, events can only be published at QoS level 0.

Events can be published by using:


- eventType - Type of event to be published e.g status, gps.
- eventFormat - Format of the event e.g json.
- data - Payload of the event.(Must be buffer/String)
- QoS - MQTT quality of service for the publish event. Supported values : 0,1,2.

.. code:: javascript

    var deviceClient = new Client.IotfDevice(config);

    deviceClient.connect();

    deviceClient.on("connect", function () {
       //publishing event using the default quality of service
       deviceClient.publish("status","json",'{"d" : { "cpu" : 60, "mem" : 50 }}');

       //publishing event using the user-defined quality of service
       var myQosLevel=2
       deviceClient.publish("status","json",'{"d" : { "cpu" : 60, "mem" : 50 }}', myQosLevel); 
  });

Handling commands
------------------

When the device client connects, it automatically subscribes to any command for this device. To process specific commands you need to register a command callback function. The device client sends *command* when a command is received. The callback function has the following properties.

-   commandName - name of the command invoked
-   format - e.g json, xml
-   payload - payload for the command
-   topic - actual topic where the command was received

.. code:: javascript

	var deviceClient = new Client.IotfDevice(config);

	deviceClient.connect();

	deviceClient.on("connect", function () {
		//publishing event using the default quality of service
		deviceClient.publish("status","json",'{"d" : { "cpu" : 60, "mem" : 50 }}');

	});

	deviceClient.on("command", function (commandName,format,payload,topic) {
		if(commandName === "blink") {
			console.log(blink);
			//function to be performed for this command
			blink(payload);
		} else {
			console.log("Command not supported.. " + commandName);
		}
	});

Handling errors
----------------

When the device clients encounters an error, it emits an *error* event.

.. code:: javascript

	var deviceClient = new Client.IotfDevice(config);

	deviceClient.connect();

	deviceClient.on("connect", function () {
		//publishing event using the default quality of service
		deviceClient.publish("status","json",'{"d" : { "cpu" : 60, "mem" : 50 }}');

	});

	deviceClient.on("error", function (err) {
		console.log("Error : "+err);
	});
	.... 


Disconnect Client
--------------------

Disconnects the client and releases the connections

.. code:: javascript

	var deviceClient = new Client.IotfDevice(config);

	deviceClient.connect();

	client.on("connect", function () {
		//publishing event using the default quality of service
		client.publish("status","json",'{"d" : { "cpu" : 60, "mem" : 50 }}');

		//publishing event using the user-defined quality of service
		var myQosLevel=2
		client.publish("status","json",'{"d" : { "cpu" : 60, "mem" : 50 }}', myQosLevel); 

		//disconnect the client
		client.disconnect();
	});

	....
