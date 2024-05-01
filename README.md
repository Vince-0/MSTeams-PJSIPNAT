# MSTeamsPJSIPNAT_Debian12
Compiled PJSIP NAT module for Asterisk 21 on Debian 12

Usually lives in /usr/lib/x86_64-linux-gnu/asterisk/modules/ on FreePBX, Asterisk 21, Debian 12 Bookworm: https://github.com/FreePBX/sng_freepbx_debian_install .

You may load it like this:

Download from this repo into Asterisk modules directory:

        wget -O res_pjsip_nat.so.MSTEAMS -P /usr/lib/x86_64-linux-gnu/asterisk/modules/ https://github.com/Vince-0/MSTeamsPJSIPNAT_Debian12/blob/5dcd26ef97841268ac030a39643b1b074cff362d/res_pjsip_nat.so
        
Backup original:

        mv /usr/lib/x86_64-linux-gnu/asterisk/modules/res_pjsip_nat.so  /usr/lib/x86_64-linux-gnu/asterisk/modules/res_pjsip_nat.so.ORIG

Copy custom PJSIP nat module:

        cp -v /usr/lib/x86_64-linux-gnu/asterisk/modules/res_pjsip_nat.so.MSTEAMS /usr/lib/x86_64-linux-gnu/asterisk/modules/res_pjsip_nat.so

Unload original module and load custom module:

        asterisk -rx 'module unload res_pjsip_nat.so'
        asterisk -rx 'module load res_pjsip_nat.so'

Check:

        asterisk -rx 'module show like res_pjsip_nat.so'

Shows this if successful:

        Module                         Description                              Use Count  Status      Support Level
        res_pjsip_nat.so               PJSIP NAT Support                        0          Running              core
        1 modules loaded
