Welcome to ARTOF's documentation!
===================================

The **Agricultural Robot Taskmap Operation Framework (ARTOF)**
provides the common functionality of task map execution
for a wide range of robot configurations and farming applications
based on Global Navigation Satellite System (GNSS) positioning.

The robot supports the four-wheel-drive four-wheel-steer (4WD4WS), skid- and Ackerman steering vehicle configuration for different agricultural tasks in arable farming and horticulture.
The framework can be integrated on a robot platform during the development and construction or can be added with a customized after-market kit.
The ARTOF framework was integrated in several robots until now.


.. note::

   This ARTOF project started in 2019 at `ILVO <https://ilvo.vlaanderen.be/en>`_ within the `European Interreg-Vlaanderen CIMAT <https://www.cimat.be/>`_ project and is still under active development to support in other EU projects.

Integrations
------------
CIMAT robot
^^^^^^^^^^^
The CIMAT robot is a 4WD4WS robot used for crop care applications, e.g. hoeing, ridging and flame weeding, and to do soil pressure sampling.

|pic1| |pic2|
|pic3|

.. |pic1| image:: images/fig_cimat_flame_weeding.jpg
   :width: 45%

.. |pic2| image:: images/fig_cimat_soil_compaction.jpg
   :width: 45%

.. |pic3| image:: images/fig_cimat_hoeing.jpg
   :width: 90.7%

The characteristics of the CIMAT robot are listed below.

+-----------------------+---------------------------------------------------------------+
|                                       **CIMAT robot**                                 |
+=======================+===============================================================+
| Vehicle configuration | 4WD4WS                                                        |
+-----------------------+---------------------------------------------------------------+
| Weight                | 1200 kg                                                       |
+-----------------------+---------------------------------------------------------------+
| Max velocity          | 15 km/h                                                       |
+-----------------------+---------------------------------------------------------------+
|                        **Energy source (LiFePO4 battery)**                            |
+-----------------------+---------------------------------------------------------------+
| Voltage               | 52 V                                                          |
+-----------------------+---------------------------------------------------------------+
| Capacity              | 20 kWh                                                        |
+-----------------------+---------------------------------------------------------------+
|                          **Transmission (electric)**                                  |
+-----------------------+---------------------------------------------------------------+
| Nominal power         | 4 x 6 kW                                                      |
+-----------------------+---------------------------------------------------------------+
| Gear ratio            | 40                                                            |
+-----------------------+---------------------------------------------------------------+
|                             **Steering (electric)**                                   |
+-----------------------+---------------------------------------------------------------+
| Nominal power (Steer) | 4 x 440 W                                                     |
+-----------------------+---------------------------------------------------------------+
| Gear ratio (Steer)    | 208                                                           |
+-----------------------+---------------------------------------------------------------+
|                                **Hydraulics**                                         |
+-----------------------+---------------------------------------------------------------+
| Nominal power         | 2.4 kW                                                        |
+-----------------------+---------------------------------------------------------------+
| RPM                   | 3000                                                          |
+-----------------------+---------------------------------------------------------------+
| Pump size             | 3cc                                                           |
+-----------------------+---------------------------------------------------------------+
|                             **Implement interface**                                   |
+-----------------------+---------------------------------------------------------------+
| Mounting              | Center three-point hitch cat.1, rear three-point hitch cat. 2 |
+-----------------------+---------------------------------------------------------------+
| Hydraulic couplers    | 3                                                             |
+-----------------------+---------------------------------------------------------------+
| Electric connectors   | 24V, 48V                                                      |
+-----------------------+---------------------------------------------------------------+


Electro-mechanical design and construction was done by `Lambers LMB <https://lambers-lmb.be/>`_, `ILVO <https://ilvo.vlaanderen.be/en>`_ and other CIMAT consortium partners.
The initial version of the ARTOF framework was integrated.
During the CIMAT project the

Contents
--------

.. toctree::

   usage
   api
