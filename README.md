# MSTeamsPJSIPNAT
USE AT YOUR OWN RISK

Compiled PJSIP NAT modules for Asterisk 21, 22 and 23 on Debian 12



Usually lives in /usr/lib/x86_64-linux-gnu/asterisk/modules/ on FreePBX, Asterisk 21/22/23, Debian 12 Bookworm: https://github.com/FreePBX/sng_freepbx_debian_install .

You may load it like this (example for Asterisk 22; adjust the version in the URL for 21 or 23):

Download from this repo into the Asterisk modules directory:

	    # For Asterisk 21
	    wget -O /usr/lib/x86_64-linux-gnu/asterisk/modules/res_pjsip_nat.so.MSTEAMS \
	      https://github.com/Vince-0/MSTeamsPJSIPNAT_Debian12/raw/main/prebuilt/debian12-amd64/asterisk-21/res_pjsip_nat.so

	    # For Asterisk 22
	    wget -O /usr/lib/x86_64-linux-gnu/asterisk/modules/res_pjsip_nat.so.MSTEAMS \
	      https://github.com/Vince-0/MSTeamsPJSIPNAT_Debian12/raw/main/prebuilt/debian12-amd64/asterisk-22/res_pjsip_nat.so

	    # For Asterisk 23
	    wget -O /usr/lib/x86_64-linux-gnu/asterisk/modules/res_pjsip_nat.so.MSTEAMS \
	      https://github.com/Vince-0/MSTeamsPJSIPNAT_Debian12/raw/main/prebuilt/debian12-amd64/asterisk-23/res_pjsip_nat.so

Backup original:

	    mv /usr/lib/x86_64-linux-gnu/asterisk/modules/res_pjsip_nat.so /usr/lib/x86_64-linux-gnu/asterisk/modules/res_pjsip_nat.so.ORIG

Copy custom PJSIP NAT module:

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
