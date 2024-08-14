
Configuration files
===================

.. _layer_interface:

Interface
---------

The ``config.json`` configuration file describes the Redis variables in the system as well as the *mechatronic/operational layer interface* configuration

.. literalinclude:: files/configuration/config.json
   :language: JSON
   :caption: The ``config.json`` file of the CIMAT robot

The ``protocols`` object defines the protocol configuration.

#. ``snap7``: provides the *mechatronic/operational layer interface* parameters:

   + the IP address of the PLC
   + the read and write data block (DB), which correspond with the *higherLevelMonitor* and *higherLevelControl*
   + the rack and slot of these data blocks

#. ``redis``: provides the *Redis ARTOF interface* parameters:

   + the IP address and port of the Redis server

The ``variables`` object defines the ``plc`` and ``pc`` Redis varialbes on the system.

#. ``plc`` variables: the ``monitor`` variables are read and the ``control`` variables are written to the PLC using the *S7-communication protocol (Snap7)* by the :cpp:class:`RobotPLC`. These variables are continuously synced between the *operational and mechatronic layer*.

#. ``pc`` variables are solely used by the operational layer and require no synchronisation with the mechatronic layer.


The configuration depends on the platform configuration (vehicle configuration, how many hitches, what energy source, ...).

The recurrent, composite types are described in the ``types.json`` file. An extract of the ``types.json`` configuration file is shown below.
Not shown in the extract is that types also can be nested.

.. literalinclude:: files/configuration/types.json
   :language: JSON
   :caption: ``types.json`` file

Redis variables that need configuration are listed in the ``redis.init.json`` file, such as shown in the ``variables`` object in the code block below.
Next to information about the initialization of the Redis variables in contains information related to the processes and jobs that need to run.
The latter is later discussed in :ref:`system_manager`.

.. literalinclude:: files/configuration/redis.init.json
   :language: JSON
   :caption: ``redis.init.json`` file of the CIMAT robot

Platform
--------

The ``settings.json`` file describes the platform configuration of the robot platform.

.. literalinclude:: files/configuration/settings.json
   :language: JSON
   :caption: ``settings.json`` file of the CIMAT robot

These settings include:

+ ``name``: The name of the robot platform

+ ``robot``: The robot platform configuration

   + The dimensions of the robot platform (``width``, ``height``, ``wheel_diameter``)
   + The ``transform`` (**T**: translation in m, **R**: rotation in euler angles) to the reference of the robot.
      + For a 4WD4WS robot this coincides with the geometric center of the robot.
      + For an Ackerman robot this is the center of the rear axle. In that case also the ``transform_center`` needs to be added in the ``robot`` configuration object.

+ ``auto_velocity``: ``min`` and ``max`` velocity of the platform.

+ ``nav_modes``: The navigation modes of the robot. This depends on the vehicle configuration, different navigation modes are possible at the headacre. In the image below the navigation modes are shown with there corresponding id:

   #. 90°: When a corner is reached, the robot turns around its axis for the angle of the corner.

   #. 180°: When a headacre is reached, the robot turns around its axis until it reaches the same orientation of the new trajectory line to follow. Than it navigates sideways until reaching the corner of the next trajectory line.

   #. pure pursuit: When a headacre is reached, the robot drives a circle with the platform turning radius specified in the ``platform.json`` configuration file.

   #. rollback: When a headacre is reached, the robot drives a quarter of a circle with the platform turning radius. It rolls back when the corner with the next trajectory line was passed and turns in again to follow the new trajectory line.

   #. external: An external controllor can take over and perform the navigation, by updating the ``navigation_control`` parameters.

.. image:: images/fig_navigation_modes.png
   :width: 70%
   :alt: Navigation modes with navigation id
   :align: center


+ ``nav_modes``: The modes for autonomous navigation.

   #. Full auto: performs autonomous velocity control and steering
   #. Auto steer: performs only autonomous steering (no velocity control)
   #. Auto throttle: performs only autonomous velocity control (no steering)

+ ``hitches``: The specifications of the different hitches on the platform

   #. ``id``: hitch id

   #. ``name``: name of the hitch

   #. ``min``: min height of the hitch (cm)

   #. ``max``: max height of the hitch (cm)

   #. ``transform`` to the hitch pen

   #. ``types``: Compatible task types with the hitch.

      #. *hitch*: The hitch is activated (lowered) when the hitch centre is contained by a polygon of the task map.

      #. *continuous*:  The section is activated when it intersects with a polygon of the task map. A special type of this is a *cardan* task, which activates the cardan when intersecting the polygon(s) in the task map.

      #. *intermittent*: The section is activated when it contains a point of the task map.

      #. *discrete*: The discrete action (measurement) is performed when the section is the closest to the point of the task map.

.. image:: images/fig_task_types.png
   :width: 70%
   :alt: Task types compatible with the hitch
   :align: center

