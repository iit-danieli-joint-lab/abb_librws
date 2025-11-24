# abb_librws

[![license - bsd 3 clause](https://img.shields.io/:license-BSD%203--Clause-blue.svg)](https://opensource.org/licenses/BSD-3-Clause)

[![support level: community](https://img.shields.io/badge/support%20level-community-lightgray.svg)](http://rosindustrial.org/news/2016/10/7/better-supporting-a-growing-ros-industrial-software-platform)

## Important Notes

RobotWare main versions `6.x` and `7.x` are currently supported. 

## Overview

A C++ library for interfacing with ABB robot controllers supporting *Robot Web Services* (RWS) `1.0` (for RobotWare `6.x`) and *Robot Web Services* (RWS) `2.0` (for RobotWare `7.x`).  
Refer to the online [RWS 1.0 documentation](http://developercenter.robotstudio.com/webservice/api_reference) and [RWS 2.0 documentation](https://developercenter.robotstudio.com/api/RWS?urls.primaryName=Introduction) for a detailed description of what RWS is and how to use it.

Usage differs depending on how the `RWSInterface` constructor is instantiated:

- **For RWS 1.0:** only the robot’s IP address is required.  
- **For RWS 2.0:** in addition to the IP address, a `Poco::Net::Context` object must be provided, defined as follows:

```cpp
Poco::Net::Context::Ptr pContext =
    new Poco::Net::Context(
        Poco::Net::Context::CLIENT_USE, 
        "", 
        "", 
        "", 
        Poco::Net::Context::VERIFY_NONE
    );
```

Explanation of Parameters:

- `Poco::Net::Context::CLIENT_USE` — Creates an SSL context for a client (C++) connecting to a server (the robot).
- `""` (privateKeyFile) — No client private key.
- `""` (certificateFile) — No client certificate.
- `""` (caLocation) — No certificate authority bundle.
- `Poco::Net::Context::VERIFY_NONE` — Disables all TLS certificate verification on the client side.

Instantiating the relevant constructor automatically connects to the robot using the default ports:

- port **80** for RWS 1.0
- port **443** for RWS 2.0

If, when using RWS 2.0, the connection fails, check which port the controller is actually listening on.
You can force port **443** by editing:

`C:\Users\<user>\AppData\Local\ABB\RobotWare\RobotControl_7.xx.x\system\appweb.conf`

and setting:

`ListenSecure 443`

For RWS 2.0, mastership is required for write operations (and must be released immediately after the write).
To check in detail which operations require mastership, refer to the **RWS 2.0 documentation** linked above, look for the **“Mastership is required”** note within the function descriptions.

Please note that this package has not been productized, it is provided "as-is" and only limited support can be expected.

### Sketch

The following is a conceptual sketch of how this RWS library can be viewed, in relation to an ABB robot controller as well as the EGM companion library mentioned above. The optional *StateMachine Add-In* is related to the robot controller's RAPID program and system configuration.

![RWS sketch](docs/images/rws_sketch.png)

### Requirements

* RobotWare version `6.x` (for RWS `1.0`) or `7.x` (for RWS `2.0`).

### Dependencies

* [POCO C++ Libraries](https://pocoproject.org) (`>= 1.4.3` due to WebSocket support)

### Limitations

RWS provides access to several services and resources in the robot controller, and this library currently support the following:

* Reading/writing of IO-signals.
* Reading/writing of RAPID data.
* Reading of RAPID data properties.
* Starting/stopping/resetting the RAPID program.
* Subscriptions (i.e. receiving notifications when resources are updated).
* Uploading/downloading/removing files.
* Checking controller state (e.g. motors on/off, auto/manual mode and RAPID execution running/stopped).
* Reading the joint/Cartesian values of a mechanical unit.
* Register as a local/remote user (e.g. for interaction during manual mode).
* Turning the motors on/off.
* Reading of current RobotWare version and available tasks in the robot system.
*	Enable/disable lead-through.
*	Access to SmartGripper functionality.

### Recommendations

* This library has been verified to work with RobotWare `6.15.01` and `7.18.2`. Other versions are expected to work, but this cannot be guaranteed at the moment.
* It is a good idea to perform RobotStudio simulations before working with a real robot.
* It is prudent to familiarize oneself with general safety regulations (e.g. described in ABB manuals).
* Consider cyber security aspects, before connecting robot controllers to networks.

## Usage Hints

This is a generic library, which can be used together with any RAPID program and system configuration. The library's primary classes are:

* [POCOClient](include/abb_librws/rws_poco_client.h): Sets up and manages HTTP and WebSocket communication and is unaware of the RWS protocol.
* [RWSClient](include/abb_librws/rws_client.h): Inherits from `POCOClient` and provides interaction methods for using the RWS services and resources.
* [RWSInterface](include/abb_librws/rws_interface.h): Encapsulates an `RWSClient` instance and provides more user-friendly methods for using the RWS services and resources.
* [RWSStateMachineInterface](include/abb_librws/rws_state_machine_interface.h): Inherits from `RWSInterface` and has been designed to interact with the aforementioned *StateMachine Add-In*. The interface knows about the custom RAPID variables and routines, as well as system configurations, loaded by the RobotWare Add-In.

The optional *StateMachine Add-In* for RobotWare can be used in combination with any of the classes above, but it works especially well with the `RWSStateMachineInterface` class.

### StateMachine Add-In [Optional]

The purpose of the RobotWare Add-In is to *ease the setup* of ABB robot controllers. It is made for both *real controllers* and *virtual controllers* (simulated in RobotStudio). If the Add-In is selected during a RobotWare system installation, then the Add-In will load several RAPID modules and system configurations based on the system specifications (e.g. number of robots and present options).

The RAPID modules and configurations constitute a customizable, but ready to run, RAPID program which contains a state machine implementation. Each motion task in the robot system receives its own state machine instance, and the intention is to use this in combination with external systems that require interaction with the robot(s). The following is a conceptual sketch of the RAPID program's execution flow.

<p align="center">
  <img src="docs/images/statemachine_addin_sketch.png" width="500">
</p>

To install the Add-In:

1. Go to the *Add-Ins* tab in RobotStudio.
2. Search for *StateMachine Add-In* in the *RobotApps* window.
3. Select the desired Add-In version and retrieve it by pressing the *Add* button.
4. Verify that the Add-In was added to the list *Installed Packages*.
5. The Add-In should appear as an option during the installation of a RobotWare system.

See the Add-In's user manual ([1.0](https://robotapps.blob.core.windows.net/appreferences/docs/27e5bd15-b5ec-401d-986a-30c9d2934e97UserManual.pdf) or [1.1](https://robotapps.blob.core.windows.net/appreferences/docs/cd504500-80e2-4cb6-9419-c60ea4ad6d56UserManual.pdf)) for more details, as well as for install instructions for RobotWare systems. The manual can also be accessed by right-clicking on the Add-In in the *Installed Packages* list and selecting *Documentation*.

## Acknowledgements

This work is based on the [abb_librws](https://github.com/ros-industrial/abb_librws) classes developed by Jon Tjerngren for ABB IRC5 controllers (for RWS `1.0`, running RobotWare `6.x`) and on the [abb_librws](https://github.com/JOiiNT-LAB/abb_wrapper/tree/master/abb_librws) classes developed by JOiiNT-LAB for ABB controllers (for RWS `2.0`, running RobotWare `7.x`).
