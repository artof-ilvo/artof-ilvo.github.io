
.. _basic_concepts:

Basic concepts
==============

Field
-----

A field describes the entire field operation in the :ref:`system add-on web app <addon_system_admin_settings>`.
It exists out of shapefiles with the Coordinate Reference system (CRS) WGS84 or UTM.

A field includes:

1. ``traject``: A trajectory is mandatory when executing a field operation. The shapefile is of the shape type Linestring.
2. ``geofence``: A geofence is also mandatory when executing a field operation. The shapefile is of the shape type polygon.
3. ``tasks``: Tasks are optional and describe the robot's actions at specific locations. The following *task types* exist

   + *hitch*: The hitch is activated (lowered) when a polygon of the task map contains the hitch centre.

   + *continuous*:  The section is activated when it intersects with a polygon of the task map. A particular type of this is a *cardan* task, which activates the Power Take Off (PTO) when intersecting the polygon (s) in the task map.

   + *intermittent*: The section is activated when it contains a point on the task map.

   + *discrete*: The discrete action (mostly a measurement) is performed when the section is the closest to the point of action on the task map.

.. figure:: images/fig_task_types.png
   :width: 70%
   :align: center

   **Figure 1.** Task types

Every feature (polygon or point) of a task carries its own attribute:

+ ``rate`` (float, default ``100``) for *continuous* and *cardan* tasks: the application rate of the sections that intersect the polygon.
+ ``routine`` (int, default ``1``) for *discrete* and *intermittent* tasks: the action number the implement executes at the point.

The defaults are written when a task is saved, and the values can be edited per feature in the :ref:`system add-on web app <addon_system>`.

On the robot, a field is stored as a folder ``$ILVO_PATH/field/<name>/`` with an ``info.json`` file describing its tasks, and the shapefiles of the geofence, the traject and each task.
Field names may only contain letters, digits and underscores.
Shapefiles can be prepared in a GIS program such as QGIS, e.g. with the :ref:`sampling route generator <tools_sampling_route>`.

.. _basic_concepts_as_applied:

As-applied map
^^^^^^^^^^^^^^

An as-applied map records what the implement sections actually did during a field operation.
Recordings are made by the :ref:`task-map add-on <addon_taskmap>` as *sessions*, one folder per recording:

.. code-block:: text

   $ILVO_PATH/field/<field>/as_applied/<YYYYmmdd-HHMMSS>/
       sections.gpkg   area swept by every implement section (implement, section, rate, time)
       points.gpkg     robot.ref.state point with all Redis variables (+ time, lat, lon, ref.yaw)

Both GeoPackages open directly in QGIS. Recording is controlled by the Redis variables ``pc.as_applied.record``, ``pc.as_applied.session`` and ``pc.as_applied.heartbeat``.


.. _basic_concepts_platform:

Platform
--------
The platform abstracts the robot's capabilities and includes information such as dimensions, the number and position of the hitches, the vehicle configuration, etc.
The ``settings.json`` file describes the platform configuration and can be edited in the :ref:`system add-on web app <addon_system_admin_settings>` JSON editor.

.. literalinclude:: files/configuration/settings.json
   :language: JSON
   :caption: **Listing 1.** ``settings.json`` file of the CIMAT robot

The fields in the ``settings.json`` file are:

1. ``name``: The name of the robot platform

2. ``robot``: The robot platform configuration

   2.1. The dimensions of the robot platform (``width``, ``length``, ``wheel_diameter``) in meters.

   2.2. ``config``: The vehicle configuration (``ackermann``, ``4wd4ws``, ``diff`` or ``omni``).

   2.3. ``transform``: The translation (in meters) and rotation in Euler angles (degrees) of the different robot components to the antenna reference point on the robot. The robot reference point is measured from the ground on and is used by the navigation algorithms, to calculate the robots position in respect to the trajectory.

      + For a 4WD4WS robot vehicle configuration, the robot reference coincides with its geometric center.
      + For an Ackerman robot, the robot reference coincides with the centre of mass. In the case this does not equal the robot center, this transformation is written under the ``transform_center`` field of the ``robot`` configuration object.

.. figure:: images/fig_robot_transformations.png
   :width: 70%
   :align: center

   **Figure 2.** Robot component transformations, expressed relative to the antenna reference

3. ``auto_max_accel``: maximum ``linear`` (m/s²) and ``angular`` (rad/s²) acceleration in autonomous mode.

4. ``auto_velocity``: ``min`` and ``max`` velocity of the platform (m/s).

5. ``nav_modes``: The allowed navigation modes. Depending on the robot platforms' vehicle configuration, different headland turns are possible. The navigation modes determine these headland turns and are shown in *Figure 3* with their corresponding ID.

   The ids are fixed: ``1`` *90° spinning*, ``2`` *180° spinning*, ``3`` *pure pursuit*, ``4`` *rollback* and ``5`` *external*. Only list the modes your vehicle configuration supports.

   + *90° spinning*: When the robot reaches a corner, it rotates around its axis until its orientation matches the direction of the next trajectory line it needs to follow. This basic turning process applies to both regular corners and headland turns.

   + *180° spinning*: When the robot reaches a headland turn, it first rotates around its axis until it aligns with the direction of the next row it needs to follow. After aligning with the new row's trajectory, the robot moves sideways toward the corner where this new row begins. Once it reaches the corner, the robot continues driving along the trajectory line of the new row. The navigation mode *90° spinning* is used when the turns are regular corners.

   + *pure pursuit*: When the robot encounters a corner, it will navigate the turn by driving in a circular path. The radius of this circle is determined by the turning radius specified in the platform.json configuration file. If the corner is part of a headland turn (where the two rows of the field are closer together than twice the minimum turning radius), the robot will instead drive along a circular arc that connects the two rows. This arc will have the minimum turning radius, allowing the robot to smoothly transition from one row to the next within the limited space.

   + *rollback*: When the robot reaches a headland turn, it initially turns by driving in a quarter-circle, using the minimum turning radius specified for the platform. If the robot doesn't have enough space to align with the next trajectory line, it will move backward to create the necessary space. Once there is enough room, the robot makes another quarter-circle turn with the minimum turning radius, aligning itself with the new trajectory line and continuing on its path.

   + *external*: An external controller can take control of the robot's navigation by updating the navigation_control parameters. This allows the external controller to direct the robot's movements, overriding the robot's default navigation system.

.. figure:: images/fig_navigation_modes.png
   :width: 70%
   :align: center

   **Figure 3.** Navigation modes with their navigation id


6. ``auto_modes``: The different modes for autonomous navigation.

   + *Full auto*: The robot operates fully autonomous, with autonomous velocity- and steering control.

   + *Auto steer*: The robot only performs autonomous steering, and no autonomous velocity control. This mode provides "steering guidance" functionality.

   + *Auto throttle*: The robot only performs autonomous velocity control, and no autonomous steering control. This mode provides "cruise control" functionality.

   The names of the auto modes match the ``plc.monitor.state.<name>`` variables. They appear in the state selector of the web app and decide when as-applied recording starts automatically.

7. ``hitches``: The specifications of the different hitches on the platform.

   7.1. ``name``: name of the hitch, one of ``FF`` (front-front), ``FB`` (front-back), ``RF`` (rear-front), ``RB`` (rear-back), ``R`` (right) or ``L`` (left). The name must also exist in ``config.json`` as ``plc.control.hitch_<name>`` and ``plc.monitor.hitch_<name>`` (in lower case). Hitches no longer have an ``id``.

   7.2. ``min``: min height of the hitch (cm)

   7.3. ``max``: max height of the hitch (cm)

   7.4. ``transform`` to the hitch hinge (see *Figure 2*)

   7.5. ``link_length``: length of the hitch link from the hinge to the hitch pen (m)

   7.6. ``types``: compatible task types with the hitch

8. ``gps``: The RTK GNSS configuration.

   8.1. ``device``: ``serial`` for a Septentrio receiver (e.g. SimpleRTK3b) on USB, or ``socket`` for a Stonex receiver on Ethernet.

   8.2. ``utm_zone``: The Universal Transverse Mercator (UTM) projection zone to complete the EPSG:326xx code whereby xx is the UTM zone (31 for Belgium).

   8.3. ``usb_port`` for a ``serial`` device (e.g. ``/dev/septentrio0``), or ``ip`` and ``udp_port`` for a ``socket`` device.

   8.4. NTRIP configuration ``ntrip_server``, ``ntrip_mountpoint``, ``ntrip_uname``, ``ntrip_pwd``

   8.5. ``antenna_rotation``: The antenna rotation describes an additional rotation of the antennas around the z-axis (in degrees).

      + 0 or 180 if the antennas are in the longitudinal direction of the platform.

      + 90 or -90 if the antennas are in the lateral direction of the platform.

   8.6. ``transform``: The transform field describes the transformation to the antenna reference (see *Figure 2*).


Implements
----------

The ``<implement-name>.json`` files describe the implement configuration and can be edited in the :ref:`system add-on web app <addon_system_admin_implements>` form or JSON editor.

The fields in the ``<implement-name>.json`` file are:

1. ``name``: The name of the implement.

2. ``on_taskmap``: This variable determines whether the task needs to be executed on the task map. If yes, each section takes the rate of the task map under its footprint. If not, the sections follow the manual section variables, so they can be operated by an add-on or program on another computer.

3. ``types``: The implement supports several types of operations: *hitch*, *continuous*, *cardan*, *intermittent* and *discrete*, as illustrated in *Figure 1*. A hitch only accepts the implement when they share at least one type. Some implements can handle multiple types of operations simultaneously. For example, the *auto-label* implement shown in *Listing 2* is designed for data collection using two cameras. The sections are defined by the cameras' fields of view. A camera can be activated in two scenarios: when the sections overlap with a specified polygon (a continuous task) or when a crop plant enters the field of view (an intermittent task).

.. literalinclude:: files/implements/auto-label.json
   :language: JSON
   :caption: **Listing 2.** Implement supporting multiple types

4. ``sections``: The sections are an array of the geometries and transformations of the implement sections.

   4.1. ``id``: The id of the section (string).

   4.2. The dimensions ``width``, ``up``, ``down`` in meters: the footprint is ``±width/2`` across and ``[-down, up]`` along the driving direction.

   4.3. ``transform``: The transformation of the center of the section to the hitch pen, visualized *Figure 2* (+x right, +y forward, +z up).

   4.4. ``link_length`` (optional): a parallel link that hangs the section further back from the hitch pen (m).

   4.5. A section can be repeated over a specified distance by using the ``repeats`` and ``offset`` fields. This functionality allows for the configuration of implements with repetitive sections, such as the spray boom described in Listing 3. The spray boom consists of 24 sections, but only the transformation for the first section needs to be defined. The ``repeats`` and ``offset`` fields define how these sections are repeated along the boom: copy ``i`` is shifted ``i·offset`` along x and gets the id ``<id><i>``.

.. literalinclude:: files/implements/spray-boom.json
   :language: JSON
   :caption: **Listing 3.** Implement with an array of sections configured by the ``repeats`` and ``offset`` fields



.. _basic_concepts_interfaces:

Interfaces
----------

The ``config.json`` file describes the system's Redis variables and the *mechatronic/operational layer interface configuration*.
You can edit the configuration file in the :ref:`system add-on web app <addon_system_admin_settings>` JSON editor.

.. literalinclude:: files/configuration/config.json
   :language: JSON
   :caption: **Listing 4.** The ``config.json`` file of the CIMAT robot

The fields in the ``config.json`` file are:

1. The ``protocols`` object defines the protocol configuration.

   1.1. The ``plc`` object provides the *mechatronic/operational layer interface* parameters:

      1.1.1. ``protocol``: ``s7`` (Siemens S7-communication, Snap7) or ``udp``

      1.1.2. the ``ip`` address of the PLC

      1.1.3. for ``s7``: the ``read_db`` and ``write_db`` data block (DB), which correspond with the *higherLevelMonitor* and *higherLevelControl*, and the ``rack`` and ``slot`` of the PLC

      1.1.4. for ``udp``: the ``read_port`` and ``write_port``

   For a UDP PLC the object looks like:

   .. code-block:: JSON

      "plc": {
          "protocol": "udp",
          "ip": "192.168.2.243",
          "read_port": 1200,
          "write_port": 1201
      }

   1.2 The ``redis`` object provides the *Redis ARTOF interface* parameters:

      1.2.1 ``ip`` and ``port``: the IP address and port of the Redis server

2. The ``variables`` object defines the ``plc`` and ``pc`` Redis variables on the system.

   2.1. ``plc`` variables: These variables are continuously synced between the *operational and mechatronic layer*.

      2.1.1 the ``monitor`` variables are read from the PLC by the :cpp:class:`RobotPlc`

      2.1.2 the ``control`` variables are written to the PLC by the :cpp:class:`RobotPlc`

   2.2. ``pc`` variables are solely used by the operational layer and require no synchronization with the mechatronic layer.

The configuration depends on the platform configuration (vehicle configuration, number of hitches, energy source, etc.).

The ``types.json`` file describes recurrent composite types in the ``config.json`` file.
These types can be nested as can be seen in *Listing 5*.


.. literalinclude:: files/configuration/types.json
   :language: JSON
   :caption: **Listing 5.** ``types.json`` file

The names of the Redis variables or keys are the composition of the nested object keys. Some examples of how these Redis keys look like are:
``plc.monitor.drive_fl.current``, ``plc.monitor.drive_fr.temperature``, ``plc.monitor.hitch_fb.feedback_sections.27``, ``plc.control.navigation.heartbeat``, ``plc.control.hitch_fb.activate_sections.30``, ``pc.navigation.turning_radius``, ``pc.purepursuit.carrot_distance``

Redis variables that need configuration are listed in the ``redis.init.json`` file, as shown in the ``variables`` object in *Listing 4*.
The initialization is only done at system installation. Redis variables are persistent. So these values are preserved on system reboot.

Besides, the ``redis.init.json`` file contains information related to the processes and jobs that need to run.
The latter is discussed in the :ref:`jobs` section.

.. literalinclude:: files/configuration/redis.init.json
   :language: JSON
   :caption: **Listing 6.** ``redis.init.json`` file of the CIMAT robot

.. _jobs:

Jobs
----

The :cpp:class:`SystemManager` class is responsible for managing all other processes (:cpp:class:`Process`) and the add-ons (:cpp:class:`Addon`) in the operational layer.
A :cpp:class:`Job` is a higher-level abstraction that implements the common functionality of the :cpp:class:`Process` and :cpp:class:`Addon` class.

A :cpp:class:`Process` is a component managed by the :cpp:class:`SystemManager` within the operational layer.
These processes are further discussed in the :ref:`operational_layer` section.

An :cpp:class:`Addon` is an entry for the :cpp:class:`SystemManager` process, which maintains the system's add-ons.
The add-on configuration uses the `Docker Engine API (1.46) <https://docs.docker.com/engine/api/v1.46/>`_ syntax.
These add-ons are further discussed in the :ref:`addons` section.

The ``redis.init.json`` file describes the job configurations at installation.
After installation, the add-on configuration can be adjusted using the :ref:`system add-on web app <addon_system_admin_settings>`.


.. _basic_concepts_states:

States
------

The ARTOF framework continuously updates the state of various platform components in Redis JSON variables, using the GNSS coordinates along with the robot and implement geometry. Each state consists of a position in UTM coordinates and an orientation. Internally, this information is represented as a 4x4 affine transformation matrix, as affines allow for efficient calculations.
For easier human readability, the affine matrix is converted into a translation vector (in meters) and an orientation vector (in degrees). This translation vector provides the position, while the orientation vector details the component's alignment in space.

+ ``gps.raw.state``: The GPS raw state provides the UTM position, heading, and roll as measured by the GNSS receiver. This is the state of the dominant GNSS antenna.

+ ``gps.ref.state``: The GPS raw state is transformed to the center point between the two antennas. This adjusted coordinate is referred to as the GPS reference state.

   + Configuration of ``gps`` ``transform`` in the ``settings.json`` file.

+ ``robot.ref.state``: The robot reference state, typically located at the robot's center of mass, serves as the reference point for calculating its relative position to the trajectory. This information is used for the navigation controller.

   + Configuration of ``robot`` ``transform`` in the ``settings.json`` file.

+ ``robot.center.state``: The geometric centre of the robot is used for visualization.

   + Configuration of ``robot`` ``center`` in the ``settings.json`` file.

+ ``robot.head.state``: The front of the robot, or the "head," is utilized by certain navigation controllers that require deviation measurements from a point at the robot's front.

   + Configuration of ``robot`` ``head`` in the ``settings.json`` file.

+ ``hitch.states``: The hitch states describe the states of the hitches. This is the state of the hitch hinge.

   + Configuration of ``hitches`` ``hitch_<name>`` ``transform`` in the ``settings.json`` file.

+ ``implement.states``: The implement states describe the states of the different implements and their sections.

   + Configuration of the ``<implement-name>.json`` file.

.. _basic_concepts_robot_status:

Robot status
^^^^^^^^^^^^

The ``robot.status`` JSON variable summarizes the robot for the web app: the errors, simulation state, GNSS fix, notification, heartbeat, power level (``plc.monitor.power_source.data.soc`` or ``.level``), and

+ ``current_state``: the active navigation state, i.e. the first ``plc.monitor.state.<auto mode>`` that is set (``normal``, ``auto``, ...).
+ ``current_substate``: the active PLC substate, the first set ``plc.monitor.substate.<name>`` of ``programming``, ``implement_control``, ``inhibited``, ``aware`` and ``stay_clear``. In simulation the substate is always ``programming``.

The substates are reported by the :ref:`mechatronic layer <mechatronic_layer_states>`.

