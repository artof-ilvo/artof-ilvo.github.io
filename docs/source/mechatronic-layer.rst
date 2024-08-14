
Mechatronic layer
=================

Introduction
------------

The mechatronic layer requires a custom implementation.
It depends on the vehicle configuration, the selected components and the safety concept for the platform.
The robot platforms developed at ILVO used Siemens technology to control its industrial components.

Requirements
------------

The ARTOF framework can be integrated after-marked or during the development of a new robot platform.

The minimal hardware requirements to perform steering guidance are:

#. **Safety** the implementation of the mechatronic layer is responsible for all the safety features of the platform in accordance to the national legislation.
#. **Programmable Logic Controller** A PLC interacts with all the actuators and sensors on the platform. The functionality that needs to be implemented are:

   + Execution of the *inverse and direct kinematic model* and operates the steering and driving actuators accordingly.
   + Example components: Siemens S7-1200 and Siemens S7-1500 series

#. **Steering angle control**: The steering angle needs to be controlled by the PLC.

   + For *hydraulic steering* this was done using a hydraulic steering block.
   + For *electric steering* this was done by controlling the motor drive.
   + *Example components*: `Raven RS1 <https://nl.ravenind.com/ag-products/guidance/rs1>`_

#. **Steering angle feedback**: The steering angle needs to be monitored by the PLC to enable closed-loop steering control.

   + For *Ackerman steering* robots this was done with an inductive angle sensor on the steering rack.
   + For *4WD4WS steering* robots all wheel angles need to be measured. This was also done using inductive angle sensors.
   + *Example components*: `Multiprox RI360P1-QR20-LU4X2-H1141 <https://www.turck.nl/nl/product/100000186>`_

#. **Velocity control**: The driving velocity needs to be controlled by the PLC.

   + For *hydraulic traction* this was done by controlling the swash plate by an electric actuator.
   + For *electric traction* this was done by setting the motor drives in velocity control.
   + For robots also enabling paddle velocity control for the driver an electronic analogue paddle was used.

#. **Velocity feedback**: The driving velocity needs to be monitored by the PLC to enable closed-loop velocity control.

   + For robots with *hydraulic transmission* this was done by a rotary encoder and two interlocking cogs.
   + For robots with *electrical transmission* this was done by reading in the hall-sensor, sin/cos encoder, etc.

Additional requirements to operate a robot platform:

#. **Hitch control**: The hitches need to be controlled by the PLC.

   + For *hydraulic hitches* this was done using electric valves.

#. **Hitch angle feedback**: The hitch angles need to be monitored by the PLC to enable closed-loop hitch height control.

   + *Example components*: `Multiprox RI360P1-QR20-LU4X2-H1141 <https://www.turck.nl/nl/product/100000186>`_


.. note::

   All safety measures must be implemented within the mechatronic layer in accordance with national legislation.
   The operational layer does not include any safety features and should only be used on products that are CE marked for autonomous operation.


Kinematic model
---------------

