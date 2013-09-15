Setup CPRS Demo on GT. M
========================

pre-requisite
-------------

* Already installed GT. M, if not refer to ::

    install GT.M

* Already has a local copy of OSEHRA VistA repository, if not ::

    git clone git://github.com/OSEHRA/VistA.git

* Setup OSERHA Automated Testing Environment, if not, refer to documentation ::

    https://github.com/OSEHRA/VistA/blob/master/Documentation/SetupTestingEnvironment.rst

Obtain CPRS Demo globals and routines
-------------------------------------

* Clone CPRSDemo branch from following github repository ::

    git clone git://github.com/jasonli2000/VistA-M.git -b CPRSDemo

* Alternatively you can also Obtain CPRS Demo from VA FOIA Site

  * Download cprsdemo cache.dat ::

      https://downloads.va.gov/files/FOIA/Software/DBA_VistA_FOIA_System_Files/cprsdemoCACHE.zip

  * Extract globals and routines

Setup CPRS Demo instance
------------------------

* Import CPRS Demo globals and routines, refer to ::

    https://github.com/OSEHRA/VistA/blob/master/Documentation/ImportGTM.rst

* Add a GTM-UNIX-TELNET device as following ::

    NAME: GTM-UNIX-TELNET                   $I: /dev/pts/
      ASK DEVICE: YES                       SIGN-ON/SYSTEM DEVICE: YES
        QUEUING: ALLOWED                      LOCATION OF TERMINAL: TELNET
          ASK HOST FILE: YES                    OPEN COUNT: 4
          MNEMONIC: GTM-LINUX-TELNET
          MNEMONIC: TELNET
            SUBTYPE: C-VT320                      TYPE: VIRTUAL TERMINAL

* Change NULL DEVICE $I from /.null to /dev/null

* Post setup configuration

    From a GT.M terminal ::

      Run D ^ZTMGRSET to set the GT.M platform specific routines
      Run D ^DINIT to initialize FileMan files
      Run D ^ZUSET to change the right ZU routine for GT.M

      Run K ^XTV(8989.3,1,”XUCP”) to disable the audit
      Run K ^XTV(8989.,1,4,”ROU”,1)
      Run S ^XTV(8989.,1,4,”PLA”,1)=””
      Run S ^XTV(8989.,1,4,1,0)=”PLA^^500”

* Setup RPC Broker via xinetd

    1. Create a cprsdemp-vista-9431 file as following
       ::

         #Assume designated username is cprsdemo, port number is 9431
         #please change accordingly
         service cprs-vista
         {
           port        = 9431
           socket_type = stream
           protocol    = tcp
           type        = UNLISTED
           user        = cprsdemo
           server      = /home/cprsdemo/xinetd/vista-rpcbroker.sh
           wait        = no
           disable     = no
           per_source  = UNLIMITED
           instances   = UNLIMITED
         }

    2. Copy cprsdemo-vista-9431 to /etc/xinetd.d ::

         sudo cp cprsdemo-vista-9431 /etc/xinetd.d/

    3. Create vista-rpcbroker.sh to setup xinetd service call as following::

         #! /bin/bash

         HOMEDIR=/home/cprsdemo/xinetd
         cd ${HOMEDIR}

         # please modify the path accordingly
         export gtm_dist="<path/to/gtm_dist/"
         export gtm_prompt="GTM>"
         export gtmver=<gtm_version>
         export gtmgbldir="<path/to/cprsdemo/global/dir>"
         export gtmroutines="<cprsdemo routines setup>"
         export gtm_zinterrupt='I $$JOBEXAM^ZU($ZPOSITION)'

         if [ ! -d log ] ; then
           mkdir log
         fi

         LOG=${HOMEDIR}/log/cprs-xinetd.log
         echo "$$ Job begin `date`"                                      >>  ${LOG}
         echo "$$  ${gtm_dist}/mumps -run GTMLNX^XWBTCPM"                >>  ${LOG}

         ${gtm_dist}/mumps -run GTMLNX^XWBTCPM "${CPRS_BANNER}"         2>>  ${LOG}
         echo "$$  RPCBroker stopped with exit code $?"                  >>  ${LOG}
         echo "$$ Job ended `date`"                                      >>  ${LOG}

    4. Restart xinetd service
       ::

         sudo service xinetd restart


    5. Try telnet localhostt 9431 to make sure xinetd routing works

* Setup FMQL

        Setup FMQL environment ::

          https://github.com/caregraf/FMQL/wiki/Install-Instructions
