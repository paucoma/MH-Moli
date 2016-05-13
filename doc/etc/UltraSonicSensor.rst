==================
Ultrasonic Sensor 
==================

------------------------
Device Characteristics
------------------------
The ultrasonic sensor is the U500.DA0-11110575 from Baumer. This device is capable of both:
  - 4-20 mA Current Output
  - 0-10 V Voltage Output

This adaptive output is recognized automatically on device power-up. During Startup, the device measures the impedance, *Z* , connected between its output (black cable/pin 4) and Ground (Blue cable/pin 3).
  - if Z > 10 kOhm --> Output === Voltage output (0-10 Vdc)
  - if Z < 10 kOhm --> Output === Current Output (4-20 mAdc)

Sensor Placement
=================
The sensor is placed at a height of approximately 925 mm from the flume floor. Unfortunately it appears as if the flume inclination has approximately a 10mm difference from head to tail. When calculating the flume height we would ideally subtract the measured distance by the sensor from 925mm but given the inclination of the flume we have to compensate for this inclination.

+----------------------+----------+
| Flume Floor Distance |  917 mm  |
+----------------------+----------+

----------------------------------
Connected Device Characteristics
----------------------------------
The connected device that senses the output of the sensor is a multiplexed-input type. This means that as the device powers up, the input port represents an open circuit until the configuration for the input type has been read.

Incompatible configuration 
============================
As durring startup the device represents a high impedance for the ultrasonic sensor, the ultrasonic sensor believes that the output should be voltage. After the device configures its input to read mA and then we have created an incompatibility where the sensor outputs voltage and the main device is expecting current.

Resolution to incompatibility
-------------------------------
To resolve this incompatible startup condition, I placed two 8.2 kOhm resistors in parallel and in parallel with the ultrasonic sensor output so that there is always a low impedance (Z < 10 kOhm) in startup/reboot conditions.

::

  analog out    o-----+--------------.
                     .|.        -----|-----
                     |R|       |Main Sensor|
                     |1|        -----------
                     '|'             |
    0V (gnd)    o-----+--------------'


Calibration adjustment to resolution
--------------------------------------
The resolution has side effects, the most immediate one being that the current is actually split between the Main sensor and the paralleled resistances. 
The current seen by the Main Sensor will be less than actually the ultrasonic sensor is providing.

To compensate for this, on site calibration was necessary.
  1. I placed a |DMM| in series with the analog output and the begining of the measurement chain and set it to measure the mA.
  2. Monitoring the reading from both the |DMM| and that from the main sensor, adjusting a simple *multiplication constant* should correct the reading.

.. |DMM| replace:: Digital Multi-Meter

+-----------------------+------------+
| Compensation Constant | 1.0573     |
+-----------------------+------------+
| Calibration Date      | 2016/05/07 |
+-----------------------+------------+


