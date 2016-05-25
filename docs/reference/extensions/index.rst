External Service Integrations
=============================

Overview
--------
External service integration allows you to bind metadata or function supported by 
another service to your Watson IoT account.

Jasper
------

Jasper is an administration and management platform for SIM devices. Jasper has been integrated into the IBM Watson IoT platform dashboard, making it possible to administer Jasper devices through your Watson IoT platform organization dashboard.

.. important:: Jasper integration is currently available as part of a limited beta.  Future updates 
  may include changes incompatible with the current version of this feature.


Supported Operations
~~~~~~~~~~~~~~~~~~~~

The built-in Jasper integration provided by our platform provides support for the following Jasper operations:

- View overall Jasper data

  - Shows: Status, Rate Plan, month-to-date data usage, month-to-date SMS usage, month-to-date voice usage, overage limits, date added, and date modified.
- Change SIM activation state.

  - Select from: Inventory, Activation Ready, Activated, Deactivated, and Retired.
- View SIM Usage

  - Shows: Cycle start date, billable and total data, billable and total SMS, billable and total voice.
  - The cycle start date can be set using a YYYY-MM-DD format.
- Send SMS to SIM
- Change rate plan


These operations can be accessed in the device drilldown of a Jasper connected device after the configuration steps below are completed.


Configuration
~~~~~~~~~~~~~

In order to connect your Watson IoT Platform platform organization with your Jasper account, there are two stages of configuration which must first be performed. The first stage of configuration is Organization Configuration.


Extension Enablement
^^^^^^^^^^^^^^^^^^^^

To enable Jasper integrtion with your Watson IoT organization follow these steps:

1. When in your Watson IoT platform dashboard, click the wrench icon on the right to open the Configuration Settings.
2. Scroll down the to the Extensions section, and set Jasper to 'On'.
3. Enter your Jasper username, password, license key, and API endpoint. The API endpoint must begin with ``http://`` or ``https://``. If the API endpoint begins with ``http://`` Watson IoT Platform will communicate with Jasper over an unencrypted connection. The API endpoint must begin with ``https://`` in order to use encryption. An example API endpoint is ``https://api.jasperwireless.com``.

Device Configuration
^^^^^^^^^^^^^^^^^^^^

Devices which are connected to both your Watson IoT platform organization and your Jasper account can be configured to display data from Jasper in the Watson IoT platform dashboard. Jasper configuration cannot be applied as part of the Add Device process, only already connected devices can be configured with Jasper. To configure your Jasper-connected devices, follow these steps:

1. In the devices tab of your Watson IoT platform dashboard, find the Jasper-connected device to be configured.
2. Select the device to open the Device Drilldown view.
3. Scroll down to the 'Extension Configuration section. The extension configuration must be entered as JSON in the following format:

.. code:: json
    
    {
        "jasper": {
            "iccid": "string"
        }
    }

4. When this has been entered, click 'Confirm changes' to save the configuration.

If the Organization configuration has been completed correctly, the Extensions section should now appear below the Extensions Configuration section in the Device Drilldown.
