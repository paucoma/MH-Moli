==============================
Variable analógica "Flume mm"
==============================

This resource treats the ultrasonic sensor output.

  - Input Channels:
    + Sensor mA
    + Flume Floor mm
    + Flume min mm
    + Flume max mm
  - Ouput Channels:
    + Flume Height mm
    + Flume Height m
    + Within Bounds

----------------------
Sensor mA Calibration 
----------------------
.. |etc/UltraSonicSensor| replace:: the config documentation of the ultrasonic sensor
.. _etc/UltraSonicSensor: ./UltraSonicSensor.rst

Due to a slight start-up condition adaptation, detailed in |etc/UltraSonicSensor|_, we have to adapt the sensed value to reflect the real output given by the sensor.

This basically consists of a multiplication block of a constant by the input value.

::

                  _____
  Sensor mA ---> |     |
                 |  X  |--> Real mA
     1.0573 ---> |_____|


----------------
Sensor mA to mm
----------------
The Sensor has a linear transfer function between its output and the distance measured.

::

                 10V/20mA
                ----------
               /  1000mm
     0V/4mA   /
    ---------/
     100mm

Depending on whether there is a voltage output or current output the formula to apply for the conversion is :
  - Distance(mm) = 56.25*Current(mA) - 125
  - Distance(mm) = 0.09*Voltage(mV) + 100


Code Optimized Caluclation
===========================
Working with current an integer multiplication /division setup could be as follows:
  - Distance(mm) [16bit] = 225*current(mA)/4 - 125

Considering that current will be obtained with 2 decimal point accuracy:
  - e.g. 18.85 mA --> (( (225*18) + (225*85)/100 )/2 + 1)/2 - 125

WIT Linearization Function
===========================
The Easy has a way of linearlizing a varible where you define two points (X_1,Y_1),(X_2,Y_2) and the translation from X (input) to Y(output) is done.

+-----+-----+-----+-----+
| X_1 | Y_1 | X_2 | Y_2 |
+=====+=====+=====+=====+
| 4   | 100 | 20  | 1000|
+-----+-----+-----+-----+


