============================
TRSII = Command Exploration
============================

This collection of information is gathered from the web to try to understand the TRSII protocol from WIT without purchasing it.

---------------
Initialization
---------------

It is quite typical to iniciate the device to Force Mode TRSII by sending an ESCAPE sequence to gain device's attention followed by ``W06``.

Example:

<-- ``CONNECT 9600<0D>``
*wait 1 second*
--> ``<1B>W06<1B>W06``
*wait 1 second*
--> *Send requests...*
<--
-->
<--
...
--> ``MODEM!<09>V<09>E<0D>``

MODEM!
======

- Write Command to remote Modem
  + ``V`` - 86 - Hangup ? 

Transmit Characters 
--------------------

  ``MODEM!\t`` *<Command>* ``\t`` *<ChkSum>* ``\n\0``

------------------------------
TRSII Request/Response Format
------------------------------

All requests and responses have the following specifics:

- *Words* are separated by tabs ``\t``
- request strings end with a *CheckSum Word* followed by ``\n``
  + Checksum is a 6 bit truncated sum, converted into a single character between 0x40 to 0x7F
  + ``\n`` can be *<0D><0A>* (``CR+LF``) or simply *<0D>* (``CR``)


Check Sum Calculation
======================

::

    unsigned int cksm = 0;
    int i = strlen(str);
    while(i>0){
      cksm += str[--i];
    }
    cksm = 64 - (cksm & 0x3F);
    cksm = 64 + (cksm & 0x3F);
    return cksm;


STATUT
======

- When a request can be considered a command, such as ``ACCESS`` *<password>* , the device will reply with ``STATUT`` symbolizing message recieved and providing a reponse code.

Transmit Characters 
--------------------

  ``STATUT\t`` *<response code>* ``\t`` *<response description>* ``\t`` *<ChkSum>* ``\n\0``

Example:
--------

Requesting an unexistant trace:

::

    T?<09>19<09>q<0D>
    STATUT<09>133<09>Err.Parameter<09>q<0D><0A>


---------------
TRSII Requests
---------------

ACCES
=====

- Allows to provide the access password
- Remote equipment should respond with a ``STATUT`` response where
  + Status code Values that are returned:
    * 0 = Password is good
    * 132 = Access denied 

Format
-------

  ``ACCESS <password>``

Transmit Characters 
--------------------

  ``ACCESS\t`` *<password>* ``\t`` *<ChkSum>* ``\n\0``


Example:
--------

::

    ACCES<09>A12541_TA<09>]<0D>
    STATUT<09>000<09>Ok<09>v<0D><0A>


IDENT?
======

- Request to read information about the remote system identification, the system version and the system type.
- Remote equipment responds with: 
  + System identification as 15 separate chars, *If the Id string < 15 chars --> padded with SPACEs*
  + The system version is returned as 3 separate chars.
  + The system type is returned as 2 separate chars.

System Types:
-------------

10 = MONET II
12 = BIMONET
20 = FORCE 100
24 = FORCE X
25 = FORCE BI-MODEM
26 = FORCE MINI
27 = FORCE PLUS
28 = FORCE ECO 2A = CLIP
29 = CLIP NANO
2A = CLIP
?2C = CLIP EMULATED?
30 = MODEM RTC
31 = MODEM BUS LS
32 = MODEM RADIO
33 = MODEM LS

Format
------

  ``IDENT?``

Transmit Characters 
--------------------

  ``IDENT?\t`` *<ChkSum>* ``\n\0``

Example:
--------

::

    IDENT?<09>D<0D>
		IDENT=<09>A12541_TA      <09>8.0.1<09>2C<09>O<0D><0A>


ETAT?
=====

- Request the Status of one or more resources
- Remote equipment responds...
  + Resource type defined as
    * ``.`` - Digital Resource
    * ``=`` - Analogic Resource
  + Resource Units and Current Value is returned


Format
------

  ``ETAT? <first resource number> [number of resources]``

Transmit Characters 
--------------------

  ``ETAT?\t`` *<FirstResourceNumber>* ``\t`` *<NumberResources>* ``\t``*<ChkSum>* ``\n\0``

Example:
--------

::

    ETAT?<09>0<09>5<09>S<0D>
    ETAT=<09>000<09><09><09>.<09>OFF             <09><09><09>=<09>= hm3:  0,000256<09>=<09>= l/s:       0,0<09>}<0D><0A>


RESS?
=====

- Request the parameters of one or more resources
- Remote equipment responds...

Format
------

  ``RESS? <first resource number> [number of resources]``

TP?
===

- Request the configuration of one or more Traces
- Remote equipment responds...

Format
------

  ``TP? <first resource number> [number of resources]``

Transmit Characters 
--------------------

  ``TP?\t`` *<NumTrace>* ``\t`` *<ChkSum>* ``\n\0``

or

  ``TP?\t`` *<NumTrace>* ``\t`` *<Number>* ``\t`` *<ChkSum>* ``\n\0``


T?
==

- Request the history of a trace
- You can request 
  + history since specific date
  + last x values
- Remote unit replies...

Format
------

  ``T? <numTrace> [Since Date / Number of Values]``

- Number of values is simply a number in ascii character e.g. ``53``
- Since Date has the format ``YYYYMMDDHHMMSS`` e.g. ``20160523090000``

Example:
--------

::

    T?<09>17<09><09>20160523090000<09>e<0D>
    T=<09>17<09>2<09>M<09>0<09>23<09>05<09>2016<09>09<09>00<09>00<09>20160523090000=0,000255<09>o<0D><0A>


Response Decifer
^^^^^^^^^^^^^^^^

 ``T=	17	2	M	0	23	05	2016	09	00	00	20160523090000=0,000255	o<0D><0A>``

- ``17`` - Trace 17
- ``2`` - Analog trace , 1 is for Digital Trace
- ``M`` - 77d -> Running
- ``0`` - interval between samples in seconds
- ``23`` - most recent sample day
- ``05`` - most recent sample moth
- ``2016`` - most recent sample year
- ``09`` - most recent sample hour
- ``00`` - most recent sample minute
- ``00`` - most recent sample second
- ``20160523090000=0,000255`` - Value of 0,000255 for first sample with Date/Time *2016/05/23 09:00:00*
- ``o`` - Checksum of message

In this case the most recent sample coincides with the requested since date and therefore there is only one sample delivered, otherwise there would be several samples delivered separated by tab character before completing the response with the checksum value.

DATE?
=====

- Requests the date of the remote equipment

Format
------

  ``DATE?``

Transmit Characters 
--------------------

  ``DATE?\t`` *<ChkSum>* ``\n\0``

TIME?
=====

- Requests the time of the remote equipment

Format
------

  ``TIME?``

Transmit Characters 
--------------------

  ``TIME?\t`` *<ChkSum>* ``\n\0``

D?
==

- Requests the date and time of the remote equipment

Format
------

  ``D?``

Transmit Characters 
--------------------

  ``D?\t`` *<ChkSum>* ``\n\0``

Example:
--------

::

    D?<09>t<0D>
    D=<09>18/05/2016<09>13:58:55<09>@<0D><0A>


------------
Other Stuff
------------


DATE ! : Permet de changer la date de l’e@sy
TIME ! : Permet de changer l’heure de l’e@sy
D ! : Permet de changer la date et l’heure de l’e@sy
ACTIF? : Permet d’obtenir l’activité d'une ressource de l’e@sy
ETAT ! : Permet de changer l’état d'une ressource de l’e@sy
ACTIF! : Permet de changer l’activité d'une ressource de l’e@sy
J? : Permet d’obtenir les évènements de l’e@sy
MEMOIRE! : Permet de sauver le paramétrage de l’e@sy

::

    str = rqCMD + "\t";
	  str += param1 + "\t";		//Not in IDENT?,DATE?,TIME?,J? iif Param exists
	  str += calcChkSum(str);
	  str += "\r\n"		// "\n" +\0 Nul terminated string