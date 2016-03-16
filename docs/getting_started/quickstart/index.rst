Quickstart
==========

`Quickstart <https://quickstart.internetofthings.ibmcloud.com>`_ is an open sandbox allowing developers to quickly and easily get devices connected to the Watson IoT platform with registration required.  Any  device that can run an MQTT client can be connected to Quickstart within mimimum fuss, `DeveloperWorks Recipes <https://developer.ibm.com/recipes>`_ features dozens of community produced tutorials for connecting different devices to the service, including but not limited to:

- `OpenBlocks IoT BX1G <https://developer.ibm.com/recipes/tutorials/openblocks-iot-bx1g-for-iot-foundation-quickstart/>`_
- `Reactive Blocks <https://developer.ibm.com/recipes/tutorials/reactive-blocks-and-java-to-iot-foundation-part-1-quickstart/>`_


Simulated Device
----------------
In addition to these, we have developed a simple browser-based simulated device, designed for mobile devices, that can be used to connect any device with a web browser to the service.  Open the following URL on a mobile phone or tablet in your favourite browser: http://quickstart.internetofthings.ibmcloud.com/iotsensor. 

This will launch a browser-based simulated device connecting to the Watson IoT platform. There are three sensors which you can manipulate using the on-screen controls:

- Temperature
- Humidity
- Object temperature

.. image:: iotsensor.png


Data Visualization
------------------
Leaving the simulated device running on your phone/tablet launch the `Quickstart application <http://quickstart.internetofthings.ibmcloud.com>`_ and enter the 12 character generated ID displayed on your phone/tablet in the upper right-hand corner.

.. image:: quickstart.png

Now, as you adjust the sensor values in your simulated device you will be able to see the data from your device visualized in real time in the Quickstart application.

.. image:: iotsensor_data.png


Mosquitto Demonstration
-----------------------
`Mosquitto <http://mosquitto.org/>`_ is a cross platform open source MQTT client that is perfect for experimenting with the Watson IoT platform service.  Once you have installed mosquitto pick an ``<applicationId>``, a ``<deviceId>`` that have a good chance to be unique (otherwise your experiment will clash with someone 
else running through this same demonstration). 

.. note:: There are a couple of restrictions you must consider when deciding on these IDs:

    - Maximum length of 36 characters 
    - Must comprise only alpha-numeric characters (``a-z``, ``A-Z``, ``0-9``) and the following special characters:

      - dash (``-``)
      - underscore (``_``)
      - dot (``.``)


With your ``<applicationId>`` and ``<deviceId>`` in hand the first thing you want to do is create a connection representing your application using ``mosquitto_sub``. Using examples of ``<applicationId>`` = myApplicationId and ``<deviceId>`` = myDeviceId:

::
    
    [user@host ~]$ mosquitto_sub -h quickstart.messaging.internetofthings.ibmcloud.com -p 1883 -i "a:quickstart:myApplicationId" -t iot-2/type/mosquitto/id/myDeviceId/evt/helloworld/fmt/json

Leave that process running, it is now time to turn your attention towards creating your device.  We will connect a device of type ``mosquitto`` and send two events to the service using ``mosquitto_pub``:

::
    
    [user@host ~]$ mosquitto_pub -h quickstart.messaging.internetofthings.ibmcloud.com -p 1883 -i "d:quickstart:mosquitto:myDeviceId" -t iot-2/evt/helloworld/fmt/json -m "{\"helloworld\": 1}"
    [user@host ~]$ mosquitto_pub -h quickstart.messaging.internetofthings.ibmcloud.com -p 1883 -i "d:quickstart:mosquitto:myDeviceId" -t iot-2/evt/helloworld/fmt/json -m "{\"helloworld\": 2}"

Returning to your application terminal you should see the two events that you published almost instantly:

::
    
    [user@host ~]$ mosquitto_sub -h quickstart.messaging.internetofthings.ibmcloud.com -p 1883 -i "a:quickstart:myApplicationId" -t iot-2/type/mosquitto/id/myDeviceId/evt/helloworld/fmt/json
    {"helloworld": 1}
    {"helloworld": 2}

That's all there is to it.  You have successfully connected a device and an application to the Watson IoT platform over MQTT, sent an event from the device to the service and recieved that event in your application.
