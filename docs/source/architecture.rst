Architecture
============

Overview
--------

The framework architecture is shown in the figure below.

.. image:: images/fig_framework_architecture.png
    :width: 80%
    :alt: Robot framework architecture
    :align: center

The ARTOF Redis API maintains the communication between the operational layer, the add-ons and other network devices or implements.

#. The **mechatronic layer** controlled the hardware components, integrated the inverse and direct kinematic models, interacting with the remote controller, enabled hitch operation, interfaced implements and integrated the machine safety state diagram by interaction with the SRP/CS.
#. The **operational layer** provided the functionality related to real-time autonomous task map execution based on GNSS navigation.
#. The **add-ons** run in docker containers and provide additional functionality. The system is shipped with

   + The *system add-on* provides a web page for user interaction.
   + The *node-red add-on* provides flow programming functionality, used for logging and implement interaction.

Communication
-------------

ARTOF Redis interface
^^^^^^^^^^^^^^^^^^^^^

Redis was selected to perform the interprocess communication between the real-time processes in the *operational layer* and inter-network communication between other implements or remote operators.


Mechatronic-operational layer interface
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The **Siemens S7-communication protocol (Snap7)** is currently the only supported interface between the *mechatronic-operational layer interface* that is implemented in the ARTOF framework.
At the PLC a *higherLevelMonitor* and *higherLevelControl* data block are maintained, from which the content is read and written respectively at every program cycle of the *operational layer* process *Robot PLC*.

At the PLC side the checkbox ``Permit Access with PUT/GET Communication from Remote Partner`` need to be set. This can be found back in the `Siemens documentation <https://cache.industry.siemens.com/dl/files/115/82212115/att_108330/v2/82212115_s7_communication_s7-1500_en.pdf>`_.

More information on this interface can be found in section :ref:`layer_interface`.