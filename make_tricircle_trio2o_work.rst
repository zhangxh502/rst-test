==================================================
Make Tricircle and Trio2o work more close together
==================================================

Background
==========

Since the tricircle was separated from trio2o, the division of labor between
the management of network resources and computing resources in the data center
cascade, Tricircle specifically cascades network resources, while trio2o only
keeps resources associated with computing resources for cascading.
After separation, the two components have difference between the architecture
and the management of the region. In order for the two components to work
together, the management of the region needs to be adjusted, and tricircle
smoke test was modified.

Region manage
=============

Tricircle region is divided as follows: ::

                    +---------------------+
             neutron|                     |
            --------|--> CentralRegion    |
                    |        |  |         |
                    +---------------------+
                             |  |
    +---------------------+  |  |   +---------------------+
    |                     |  |  |   |                     |
    |      RegionOne      |<--  --> |      RegionTwo      |
    |                     |         |                     |
    +---------------------+         +---------------------+

                  Fig. 1. Tricircle Region manage

As shown in Fig. 1. Tricircle is responsible for the cascading management of
network resources, and divides CentralRegion, RegionOne, RegionTwo into three
regions, CentralRegion is responsible for unified management of network
resources, network resources related to neutron will pass to CentralRegion to
operates. As for which cascaded region to operate, it is specified by the
region name.

Trio2o region is divided as follows: ::

                    +---------------------+
             nova   |                     |
            --------|-->  RegionOne       |
                    |        |  |         |
                    +---------------------+
                             |  |
    +---------------------+  |  |   +---------------------+
    |                     |  |  |   |                     |
    |       Pod1          |<--  --> |          Pod2       |
    |                     |         |                     |
    +---------------------+         +---------------------+

                   Fig. 2. Trio2o Region manage

As shown in Fig. 2. Trio2o is responsible for the cascade management of
computing resources, and divides RegionOne, Pod1, and Pod2 into three regions.
RegionOne is responsible for unified management of computing resources.
The computing resources related to nova will be operated through RegionOne.
As for which cascaded region to operate, it is specified by the availability
zone.

After Tricircle and Trio2o work together, the region is divided into: ::

                      +---------------------+
               neutron|                     |
              --------|-->  CentralRegion   |
                      |        |  |         |
                      +---------------------+
                               |  |
      +---------------------+  |  |   +---------------------+
  nova|                     |  |  |   |                     |
  ----|-----> RegionOne     |<--  |   |                     |
      |+--------+  |  |     |     --->|      RegionTwo      |
      ||  Pod1  |<--  ----- |-------> |                     |
      |+--------+           |         |                     |
      +---------------------+         +---------------------+

            Fig. 3. Tricircle with Trio2o Region manage

As shown in Fig. 3. When tricircle and trio2o work together, tricircle is
divided into three regions: CentralRegion, RegionOne, and RegionTwo,
while trio2o is divided into RegionOne, Pod1, and RegionTwo, that is,
tricircle and trio2o share a RegionTwo. That is, the separation of computing
resources and network resources in RegionOne, Pod1 bears the calculation part.

Tricircle Pods: ::

    +----------------------------------------+
    |  CentralRegion     |                   |
    +----------------------------------------+
    |     RegionOne      |        az1        |
    +----------------------------------------+
    |     RegionTwo      |        az2        |
    +----------------------------------------+

Trio2o Pods: ::

    +----------------------------------------+
    |        Pod1        |        az1        |
    +----------------------------------------+
    |     RegionTwo      |        az2        |
    +----------------------------------------+

Tricircle Smoke test
====================

In the tricircle smoke test, for the virtual machine, since the trio2o uses az
to distinguish regions, so the nova client is used to create and delete the
virtual machine. For the query virtual machine, since the trio2o queries
the RegionOne, all the virtual machines of the az are obtained,
so Pod1 is used instead of RegionOne to query the virtual machine.

If tricircle is used with trio2o, you need to manually set the TRIO2O_ENABLE
variable in ::
    tricircle/tempestplugin/smoke_test.sh
to true before the smoke test, which is false by default. ::

    TRIO2O_ENABLE=true

Installation with DevStack
==========================

To add the installation of trio2o refer to the following documentation: ::

    https://github.com/openstack/trio2o/blob/master/doc/source/installation.rst

Data Model Impact
=================

None

Dependencies
============

None

Documentation Impact
====================

None

References
==========

None
