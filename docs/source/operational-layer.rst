
Operational layer
=================

Introduction
------------

All processes in the operational layer is implemented in Cplusplus (CPP).
The implementation can be found back in the `operational-layer <https://github.com/artof-ilvo>`_ project.

Requirements
------------

#. **RTK GNSS localisation (with heading)**  : The framework includes a driver for the `Septentrio Mosaic-H <https://www.septentrio.com/en/products/gnss-receivers/gnss-receiver-modules/mosaic-h>`_ module.

   + *Example components*: `SimpleRTK3B Heading <https://www.ardusimple.com/product/simplertk3b-heading/>`_

#. **Computer**: The computer runs the operational layer and the add-ons.

   + *Tested on*:

+-------------+--------------+--------------------------------------------+----------+
| Brand       | Model        | CPU                                        | Memory   |
+=============+==============+============================================+==========+
| Asus        | Gigabyte     | Intel(R) Core(TM) i5-10210U CPU @ 1.60GHz  | 60 GB    |
+-------------+--------------+--------------------------------------------+----------+
| Intel       | NUC          | Intel(R) Core(TM) i7-                      |          |
+-------------+--------------+--------------------------------------------+----------+
| Intel       | NUC          | ...                                        |          |
+-------------+--------------+--------------------------------------------+----------+

#. **Operating system**:

   + The operational layer is currently only compiled for *Ubuntu 22.04*.

   + The operational layer can run in a docker container, which is usefully for testing and development. This is not recommended for operation as this is not yet validated on inference.

Processes
---------


VariableManager
^^^^^^^^^^^^^^^

All processes in the operational layer inherit the :cpp:class:`VariableManager`, which performs basically process housekeeping.

During initialization, it loads a :cpp:member:`VariableMap::variableMap` of ``typedef std::map<std::string, VariablePtr> VariableMap``


The pseudocode below illustrates what it does

.. code-block:: cpp
   :caption: Pseudocode :cpp:class:`VariableManager` main loop

      while (true) {
         // start a clk cycle (f = 20 Hz => T = 50 ms)
         clk.start();
         // Read all Redis variables
         readRedisVariables();
         // Execute process specific code
         serverTick();
         // Write Redis variables (only those who were altered during the cycle)
         writeRedisVariables();
         // Communicate process health
         toggleHeartBeatPulse();
         publishProcessingTime(clk.poll());
         // Stop the clk cycle (sleep for the remaining time)
         clk.stop();
      }

#. The :cpp:member:`Clk::clk` member of the :cpp:class:`VariableManager` class supports process execution at a fixed frequency, which is by default configured to ``20 Hz``. When the main loop executes more quickly the additional time is waited at the :cpp:func:`void Clk::stop()` method, otherwise the process is immediately continued.
#. The :cpp:func:`void readRedisVariables()` and :cpp:func:`void writeRedisVariables()` methods perform block read and write the Redis variables described in the redis configuration file at once. For the :cpp:func:`void writeRedisVariables()` only the

.. _system_manager:

SystemManager
^^^^^^^^^^^^^


Blabla