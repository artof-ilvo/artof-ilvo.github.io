
Mechatronic layer
=================

Introduction
------------

The mechatronic layer requires a custom implementation.
It depends on the vehicle configuration, the selected components, and the platform's safety concept.

Requirements
------------

The ARTOF framework can be integrated after or during the development of a new robot platform.

The minimal hardware requirements to perform steering guidance are:

#. **Safety** The implementation of the mechatronic layer is responsible for all the platform's safety features in accordance with national legislation.
#. **Programmable Logic Controller** A PLC interacts with all the actuators and sensors on the platform. The functionality that needs to be implemented are:

   + Executing the *inverse and direct kinematic model* and operating the steering and driving actuators accordingly.
   + Example components: Siemens S7-1200 and Siemens S7-1500 series

#. **Steering angle control**: The steering angle needs to be controlled by the PLC.

   + For *hydraulic steering* this was done using a hydraulic steering block.
   + For *electric steering*, this was done by controlling the motor drive.
   + *Example components*: `Raven RS1 <https://nl.ravenind.com/ag-products/guidance/rs1>`_

#. **Steering angle feedback**: The PLC needs to monitor the steering angle to enable closed-loop steering control.

   + For *Ackerman steering* robots, this was done with an inductive angle sensor on the steering rack.
   + For *4WD4WS steering* robots, all the wheel angles need to be measured. This was also done using inductive angle sensors.
   + *Example components*: `Multiprox RI360P1-QR20-LU4X2-H1141 <https://www.turck.nl/nl/product/100000186>`_

#. **Velocity control**: The driving velocity needs to be controlled by the PLC.

   + For *hydraulic traction* this was done by controlling the swash plate with an electric actuator.
   + For *electric traction*, this was done by setting the motor drives to velocity control.
   + An electronic analogue paddle was used for robots that enabled paddle velocity control for the driver.

#. **Velocity feedback**: The PLC needs to monitor the driving velocity to enable closed-loop velocity control.

   + For robots with *hydraulic transmission* this was done by a rotary encoder and two interlocking cogs.
   + For robots with *electrical transmission*, this was done by reading the hall sensor, sin/cos encoder, etc.

Additional requirements to operate a robot platform:

#. **Hitch control**: The hitches need to be controlled by the PLC.

   + For *hydraulic hitches* this was done using electric valves.

#. **Hitch angle feedback**: The PLC needs to monitor the hitch angles to enable closed-loop hitch height control.

   + *Example components*: `Multiprox RI360P1-QR20-LU4X2-H1141 <https://www.turck.nl/nl/product/100000186>`_


.. note::

   All safety measures must be implemented within the mechatronic layer.
   The operational layer does not include safety features and should only be used on products approved for autonomous operation in accordance with national legislation.


Safety state diagram
--------------------

For more information on the safety state diagram, we refer to the paper *<Coming soon>*.

Kinematic model
---------------

Introduction
^^^^^^^^^^^^

.. note::

   Abbreviation used:

      #. longitude (lng), latitude (lat)
      #. front (f), rear (r), so fr (front-right)
      #. angular velocity (:math:`\omega`), steering angle (:math:`\epsilon`)

The *direct kinematic model* describes the platform's motion by the velocity vector :math:`\vec{V}_r = (v_{lng}, v_{lat}, \omega_z)` based on the actuators feedback e.g. :math:`\omega_{wheel,rr}`, :math:`\omega_{wheel,rl}`, :math:`\epsilon_{f}`, :math:`\epsilon_{fl}`, depending on the vehicle configuration.

The *inverse kinematic model* does the opposite and starts from the required motion or velocity vector :math:`\vec{V}_r = (v_{lng}, v_{lat}, \omega_z)` to calculate the position or velocities of the actuators.

Kinematic means no external forces are incorporated in this model to come to this motion. A model where the external forces are incorporated is called a dynamic model.

The kinematic model is integrated into the mechatronic layer, making the operational layer independent of the vehicle configuration.
An essential part of the *mechatronic-operational layer interface* is the velocity vector :math:`\vec{V}_r = (v_{lng}, v_{lat}, \omega_z)`, which are described in the Redis variables ``plc.control.navigation.velocity.longitudinal``,  ``plc.control.navigation.velocity.lateral``, ``plc.control.navigation.velocity.angular``.

The book Vehicle Dynamics by Reza N. Nazar contains much information about *vehicle kinematics and dynamics*.

.. code-block::

   Jazar, R. N. (2008). Vehicle dynamics (Vol. 1). Berlin/Heidelberg,
   Germany: Springer.

Four-wheel-drive four-wheel-steer (4WD4WS)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Figure 1 was used to derive the kinematic model of a four-wheel-drive, four-wheel-steer (4WD4WS) robot.

.. figure:: images/fig_kinematics_4ws4wd.png
   :width: 70%
   :align: center

   **Figure 1.** Kinematics 4WD4WS vehicle configuration

For the *direct kinematic model*

.. math::
   f(v_{fl}, v_{fr}, v_{rl}, v_{rr}, \epsilon_{fl}, \epsilon_{fr}, \epsilon_{rl}, \epsilon_{rr}) = (v_x, v_y, \omega_z)

The following formulas can be extracted using *Figure 1*.

.. math::
    v_{fl} &= R_{w} \cdot \omega_{w,fl} \\
    v_{fr} &= R_{w} \cdot \omega_{w,fr} \\
    v_{rl} &= R_{w} \cdot \omega_{w,rl} \\
    v_{rr} &= R_{w} \cdot \omega_{w,rr} \\
    v_{y} &= \frac{1}{4} \left[ v_{fl} \cdot cos(\epsilon_{fl}) + v_{fr} \cdot cos(\epsilon_{fr}) + v_{rl} \cdot cos(\epsilon_{rl}) + v_{rr} \cdot cos(\epsilon_{rr}) \right] \\
    v_{x} &= \frac{1}{4} \left[ v_{fl} \cdot sin(\epsilon_{fl}) + v_{fr} \cdot sin(\epsilon_{fr}) + v_{rl} \cdot sin(\epsilon_{rl}) + v_{rr} \cdot sin(\epsilon_{rr}) \right] \\
    \omega_{z} & =  \frac{l}{2}  \left( \left[ v_{rl} \cdot sin(\epsilon_{rl}) + v_{rr} \cdot sin(\epsilon_{rr}) \right] - \left[  v_{fl} \cdot sin(\epsilon_{fl}) + v_{fr} \cdot sin(\epsilon_{fr})  \right] \right)


The *inverse kinematic model*

.. math::
   g(v_x, v_y, \omega_z) = (v_{fl}, v_{fr}, v_{rl}, v_{rr}, \epsilon_{fl}, \epsilon_{fr}, \epsilon_{rl}, \epsilon_{rr})

First, the points :math:`\mathrm{A}` up to :math:`\mathrm{F}` indicated in *Figure 1*, :math:`\mathrm{R}_{xx}` were determined.

.. math::
    &\left|\mathrm{AE}\right| = \left|\mathrm{BF}\right| = \frac{\mathrm{l}}{2} + \mathrm{R} \sin(\gamma) \\
    &\left|\mathrm{EC}\right| = \left|\mathrm{FD}\right| = \frac{\mathrm{l}}{2} - \mathrm{R} \sin(\gamma) \\
    &\left|\mathrm{OE}\right| = \mathrm{R} \cos(\gamma) - \frac{\mathrm{b}}{2} \\
    &\left|\mathrm{OF}\right| = \mathrm{R} \cos(\gamma) + \frac{\mathrm{b}}{2} \\
    &\mathrm{R}_{fl} = \left(\left|\mathrm{AE}\right|^2 + \left|\mathrm{OE}\right|^2 \right)^{\frac{1}{2}} \\
    &\mathrm{R}_{fr} = \left(\left|\mathrm{OF}\right|^2 + \left|\mathrm{BF}\right|^2 \right)^{\frac{1}{2}} \\
    &\mathrm{R}_{rl} = \left(\left|\mathrm{OE}\right|^2 + \left|\mathrm{EC}\right|^2 \right)^{\frac{1}{2}} \\
    &\mathrm{R}_{rr} = \left(\left|\mathrm{OF}\right|^2 + \left|\mathrm{FD}\right|^2 \right)^{\frac{1}{2}}


Also, we know:

.. math::
   &\mathrm{\Delta v}_f = \big| (\mathrm{R}_{fl}  -  \mathrm{R}_{fr}) \cdot \dot{\mathrm{\theta}}  \big| \\
   &\mathrm{\Delta v}_r = \big| (\mathrm{R}_{rl}  -  \mathrm{R}_{rr}) \cdot \dot{\mathrm{\theta}}  \big|

Using the radius :math:`\mathrm{R}_{xx}` given the equations above, :math:`\omega_{w,xx}` and :math:`\epsilon_{xx}` were calculated.

.. math::
   & \mathrm{\omega}_{w,fl} = \mathrm{sgn}(\dot{\mathrm{y}})\cdot \big(- \mathrm{sgn}(\dot{\mathrm{\theta}})\,\frac{\mathrm{\Delta v}_f}{2} + v\big) \cdot \frac{2}{\mathrm{D}_w}\\
   & \mathrm{\omega}_{w,fr} = \mathrm{sgn}(\dot{\mathrm{y}})\cdot \big( \mathrm{sgn}(\dot{\mathrm{\theta}})\,\frac{\mathrm{\Delta v}_f}{2} + v \big) \cdot \frac{2}{\mathrm{D}_w}\\
   & \mathrm{\omega}_{w,rl} = \mathrm{sgn}(\dot{\mathrm{y}})\cdot \big(- \mathrm{sgn}(\dot{\mathrm{\theta}})\,\frac{\mathrm{\Delta v}_r}{2} + v \big) \cdot \frac{2}{\mathrm{D}_w}\\
   & \mathrm{\omega}_{w,rr} = \mathrm{sgn}(\dot{\mathrm{y}})\cdot \big( \mathrm{sgn}(\dot{\mathrm{\theta}})\,\frac{\mathrm{\Delta v}_r}{2} + v \big) \cdot \frac{2}{\mathrm{D}_w} \\
   & \mathrm{\epsilon}_{fl} = \arctan \left( \frac{\left|\mathrm{AE}\right|}{\left|\mathrm{OE}\right|} \right) \\
   & \mathrm{\epsilon}_{fr} = \arctan \left( \frac{\left|\mathrm{BF}\right|}{\left|\mathrm{OF}\right|} \right) \\
   & \mathrm{\epsilon}_{rl} = \arctan \left( - \frac{\left|\mathrm{EC}\right|}{\left|\mathrm{OE}\right|} \right) \\
   & \mathrm{\epsilon}_{rr} = \arctan \left( - \frac{\left|\mathrm{FD}\right|}{\left|\mathrm{OF}\right|} \right)

Ackerman steering
^^^^^^^^^^^^^^^^^

Ackerman steering is a simplified version of a 4WD4WS robot.
The ICR moves the centre of mass M in *Figure 2*.
Consequently, points E and F coincide with points C and D, and :math:`\left|\mathrm{EC}\right| = \left|\mathrm{FD}\right| = 0` equation :math:`\epsilon_{rr} = \epsilon_{rl} = 0` apply.

.. figure:: images/fig_kinematics_ackerman.png
   :width: 70%
   :align: center

   **Figure 2.** Kinematics Ackerman vehicle configuration, with M as the center of mass.

The bicycle model is used to derive the Ackerman steering kinematics, which are represented in *Figure 2* by the dashed green lines.
Thereby applies :math:`cot(\epsilon) = \frac{cot(\epsilon_{fl}) + cot(\epsilon_{fr})}{2}` and  :math:`R = \sqrt{a^2 + l^2\, cot^2(\epsilon)}`.

Assume that only the front-right steering angle and velocity of the rear-left wheel were measured for this example.

The *direct kinematic model*

.. math::
   f(v_{rl}, \epsilon_{fr}) = (v_x, v_y, \omega_z)


.. math::
   v_{rl} &= R_{w} \cdot \omega_{w,rl} \\
   v_{y} &= \frac{R\,cos(\epsilon)}{R\,cos(\epsilon) - (b/2)}  \cdot v_{rl}\\
   v_{x} &= v_{y} \cdot tan(\epsilon) \\
   v &= \sqrt{v^2_x + v^2_y} \\
   \omega_{z} & =  \frac{v}{R}

The *inverse kinematic model*

.. math::
   g(v, \omega_z) = (v_{rl}, \epsilon_{fr})

Using formulas:

.. math::
   v &= \sqrt{v^2_x + v^2_y} \\
   v &= \omega_z \, R \\
   R &= \sqrt{a^2 + l^2\, cot^2(\epsilon)} \\
   cot(\epsilon) &= \frac{cot(\epsilon_{fl}) + cot(\epsilon_{fr})}{2} \\

We can calculate:

.. math::
   \epsilon &= acot\bigg(\sqrt{\frac{\frac{v^2}{\omega^2_z} - a^2}{l^2}}\bigg) \\
   \epsilon_{fr} &= acot(2\cdot cot(\epsilon) - cot(\epsilon_{fl})) \\
   v_{rl} &= \frac{R\,cos(\epsilon) - (b/2)}{R\,cos(\epsilon)} \cdot v \, cos(\epsilon)

Skid steering
^^^^^^^^^^^^^

Figure 3 was used to derive the kinematic model of a skid steering robot.



.. figure:: images/fig_kinematics_skid_steering.png
   :width: 70%
   :align: center

   **Figure 3.** Kinematics skid steering vehicle configuration

The *direct kinematic model*

.. math::
   f(v_{l}, v_{r}) = (v_x, v_y, \omega_z)

From *Figure 3* we can derive

.. math::
   v_x &= 0 \\
   v_y &= \frac{v_r + v_l}{2} \\
   \omega_z &= \frac{v_r - v_l}{b}

The *inverse kinematic model*


.. math::
   g(v_x, v_y, \omega_z) = (v_{l}, v_{r})


Imagine the robot driving in a circle. All parts of the robot move with the same rotational velocity around the Instanious Center of Rotation (ICR) (point O in *Figure 3*), consequently the equality :math:`\omega = \omega_r = \omega_l`.

.. math::
   \omega_z &= \frac{v}{R} = \frac{v_l}{R-a} = \frac{v_r}{R+a}  \\
   \Leftrightarrow v_l &= \frac{v}{\frac{R}{R-a}} = \frac{2v}{\frac{R+a}{R-a} + 1} \,\,\, , \,\,\,
   v_r = \frac{v}{\frac{R}{R+a}} = \frac{2v}{\frac{R-a}{R+a} + 1}