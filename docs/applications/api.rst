HTTP API for Applications
=========================

The IoT Platform API can be used to interact with your organization in the IoT Platform. 

API Capabilities
----------------

The IoT Platform API supports the following functionality for applications:

- View organization details.
- Bulk device operations (list all, add, remove).
- Device type operations (list all, create, delete, view details, update).
- Device operations (list devices, add, remove, view details, update, view location, view management information).
- Device diagnostic operations (clear log, retrieve logs, add log information, delete logs, get specific log, clear error codes, get device error codes, add an error code).
- Connection problem determination (list device connection log events).
- Historical event retrieval (view events from all devices, view events from a device type, view events for a specific device).
- Device management request operations (list device management requests, initiate a request, clear request status, get details of a request, get list of request statuses for each affected device,  get request status for a specific device).
- Usage management (retrieve number of active devices over a period of time, retrieve amount of storage used by historical event data, retrieve total amount of data used).
- Publish events on behalf of devices (beta)
- Service status queries (retrieve service statuses for an organization).


----


IoT Platform API Version 2 
------------------------------

The only supported version of the IoT Platform API is `version 2 <../swagger/v0002.html>`_.  All users must ensure their solutions are using this version.


----


IoT Platform API Version 1
------------------------------

`Version 1 <../swagger/v0001.html>`_ of the API is now disabled. You should ensure that all your applications use the current `version 2 <../swagger/v0002.html>`_ API.


