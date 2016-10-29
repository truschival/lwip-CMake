This is a repackaged version of lwip v1.4.1 using CMake 

The original source code is taken from Texas Instruments TivaWare 2.1.3 

This configuration is customized to run with FreeRTOS and always
includes the TCP stack.

The following components have been removed:
* ipv6
* snmp

Additionally this project contains:

* Texas Instruments TM4C12x controller specific port
* A Texas Instruments lwip helperlibrary in `src/lwiplib.c`

