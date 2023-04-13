=============================
 PPP for Z055 HDLC Adapters
=============================

This document describes the software and procedures necessary to implement
a PPP connection using Linux and the MEN Z055 HDLC adapter.


------------
PPP Software
------------

Linux uses the PPPD package to provide PPP communications. A good
description of the PPPD packages is provided in the PPP-HOWTO document
included with most Linux distributions.

The pppd package consists of two main parts:

        * PPP line discipline driver (ppp.o)
        * pppd user mode program

The PPP line discipline driver allows tty devices to be used for PPP
communications. The pppd program does most of the work including opening a
tty device, setting the device line discipline to PPP, and negotiating the
PPP connection. PPP options are set by using command line arguments to the
pppd program.


---------
Using PPP
---------

Once the PPP software is installed, a PPP connection is made by running
the pppd user mode program. PPP options are specified as command line
arguments to pppd.

The command line argument 'sync' selects synchronous PPP mode. Without
this option, pppd defaults to asynchronous PPP mode. All other pppd
options operate as described in the pppd documentation.

*NOTE* Both ends of a PPP connection *MUST* operate in the same mode
       (synchronous HDLC).

Z055_HDLC device options, such as serial encoding (NRZ, NRZI, Manchester)
and data clock selection, are controlled through the z055_hdlc_util
program or a custom program accessing the device directly.
These options should be set before running the pppd program.

A sample shell script 'start-ppp.sh' is provided with the Z055 HDLC
software that demonstrates setting the serial options and then invokes the
pppd program to establish a connection. The 'stop-ppp.sh' script
deactivates the connection.

*NOTE* When operating in synchronous mode, it does not make sense to use
       the 'chat' program or equivalent to send and receive AT commands.

*NOTE* When transferring bigger blocks of data, the ICMP driver generates
       a single ping (multiple packets). Because the txqueuelen of the ppp
       device defaults to a value of 3, the communication may fail.
       In this case you have to set the txqueuelen to a bigger value. A
       txqueuelen of around 36 works well on most applications.
       "ifconfig ppp0 txqueuelen 36"


--------------------
PPP for Linux Kernel
--------------------

The device file /dev/ppp must exist. This file is created automatically by
the latest version of pppd, or can be created manually with the command
(as root):

mknod /dev/ppp c 108 0

If the ppp drivers are built as modules, add the following lines to
/etc/modules.conf:

alias char-major-108    ppp_generic
alias /dev/ppp          ppp_generic
alias tty-ldisc-3       ppp_async
alias tty-ldisc-14      ppp_synctty
alias ppp-compress-21   bsd_comp
alias ppp-compress-24   ppp_deflate
alias ppp-compress-26   ppp_deflate

If using devfsd with ppp support built as modules, add the following to
/etc/devfsd.conf:

LOOKUP        PPP        MODLOAD

When rebuilding the kernel, enable the following kernel configuration
options in the 'Network device support' menu:

PPP Support (required)
PPP support for sync tty ports (required for synchronous (HDLC) PPP)
PPP Deflate compression (optional compression support)
PPP BSD-Compress compression (optional compression support)

These options may be built into the kernel or built as modules.

