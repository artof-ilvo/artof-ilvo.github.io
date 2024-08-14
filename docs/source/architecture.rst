Architecture
============

Introduction
------------

The framework architecture is shown in the figure below.

.. image:: images/fig_framework_architecture.png
    :width: 600
    :alt: Robot framework architecture

The ARTOF Redis API maintains the communication between the operational layer, the add-ons and other network devices or implements.

#. The **mechatronic layer** controlled the hardware components, integrated the inverse and direct kinematic models, interacting with the remote controller, enabled hitch operation, interfaced implements and integrated the machine safety state diagram by interaction with the SRP/CS.
#. The **operational layer** provided the functionality related to real-time autonomous task map execution based on GNSS navigation.
#. The **add-ons** run in docker containers and provide additional functionality. The system is shipped with
    * The *system add-on* provides a web page for user interaction.
    * The *node-red add-on* provides flow programming functionality, used for logging and implement interaction.

