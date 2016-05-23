============================
TRSII = Command Exploration
============================

This collection of information is gathered from the web to try to understand the TRSII protocol from WIT without purchasing it.

------------------------------
TRSII Request/Response Format
------------------------------

All requests and responses have the following specifics
- *Words* are separated by tabs ``\t``
- request strings end with a *CheckSum Word* followed by ``\n``
  + Checksum is a 6 bit truncated sum, converted into a single character between 0x40 to 0x7F
  + ``\n`` can be *<0D><0A>* (``CR+LF``) or simply *<0D>* (``CR``)

Check Sum Calculation
======================

.. math::

    cksm = \sum_{i=0}^{i=strlen(str)} str[i]


::
    
    cksm &= 63;
    cksm = 64 - cksm;
    cksm &= 63;
    cksm += 64;
    return cksm;


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
::

    ACCES <password>


Literal String to send as request:

 ``ACCESS\t`` *<password>* ``\t`` CheckSum("``ACCESS\t`` *<password>* ``\t``") ``\n\0``


Example:

::

    ACCES<09>A12541_TA<09>]<0D>
    STATUT<09>000<09>Ok<09>v<0D><0A>

