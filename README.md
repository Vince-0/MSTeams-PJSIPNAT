# MSTeamsPJSIPNAT

**USE AT YOUR OWN RISK**

Precompiled PJSIP NAT modules for Asterisk 21, 22 and 23 on Debian 12 with **runtime FQDN configuration** support.

## What's included

These modules are built from Asterisk sources patched with the `ms_signaling_address` runtime configuration feature (v2 and later). This allows you to configure the FQDN for MS Teams Direct Routing at runtime via `pjsip.conf` instead of hard-coding it at compile time.

**Key features:**
- ✅ **No hard-coded FQDN** - the modules are generic and work for any deployment
- ✅ **Runtime configuration** - set your FQDN in `pjsip.conf` using the `ms_signaling_address` parameter
- ✅ **No recompilation needed** - change your FQDN by editing config and reloading PJSIP

**IMPORTANT:** These modules **require** you to configure `ms_signaling_address` in your PJSIP transport configuration. See the configuration section below.

## Installation

These modules are typically installed in `/usr/lib/x86_64-linux-gnu/asterisk/modules/` on FreePBX, Asterisk 21/22/23, Debian 12 Bookworm: https://github.com/FreePBX/sng_freepbx_debian_install

### Step 1: Download the module

Download the appropriate version for your Asterisk installation:

	    # For Asterisk 21
	    wget -O /usr/lib/x86_64-linux-gnu/asterisk/modules/res_pjsip_nat.so.MSTEAMS \
	      https://github.com/Vince-0/MSTeamsPJSIPNAT_Debian12/raw/main/prebuilt/debian12-amd64/asterisk-21/res_pjsip_nat.so

	    # For Asterisk 22
	    wget -O /usr/lib/x86_64-linux-gnu/asterisk/modules/res_pjsip_nat.so.MSTEAMS \
	      https://github.com/Vince-0/MSTeamsPJSIPNAT_Debian12/raw/main/prebuilt/debian12-amd64/asterisk-22/res_pjsip_nat.so

	    # For Asterisk 23
	    wget -O /usr/lib/x86_64-linux-gnu/asterisk/modules/res_pjsip_nat.so.MSTEAMS \
	      https://github.com/Vince-0/MSTeamsPJSIPNAT_Debian12/raw/main/prebuilt/debian12-amd64/asterisk-23/res_pjsip_nat.so

### Step 2: Backup the original module

```bash
mv /usr/lib/x86_64-linux-gnu/asterisk/modules/res_pjsip_nat.so \
   /usr/lib/x86_64-linux-gnu/asterisk/modules/res_pjsip_nat.so.ORIG
```

### Step 3: Install the custom module

```bash
cp -v /usr/lib/x86_64-linux-gnu/asterisk/modules/res_pjsip_nat.so.MSTEAMS \
      /usr/lib/x86_64-linux-gnu/asterisk/modules/res_pjsip_nat.so
```

### Step 4: Reload the module

```bash
asterisk -rx 'module unload res_pjsip_nat.so'
asterisk -rx 'module load res_pjsip_nat.so'
```

### Step 5: Verify the module loaded

```bash
asterisk -rx 'module show like res_pjsip_nat.so'
```

Expected output:
```
Module                         Description                              Use Count  Status      Support Level
res_pjsip_nat.so               PJSIP NAT Support                        0          Running              core
1 modules loaded
```

## Required Configuration

**CRITICAL:** These modules use runtime FQDN configuration. You **must** configure the `ms_signaling_address` parameter in your PJSIP transport configuration.

### For FreePBX systems

Create or edit `/etc/asterisk/pjsip.transports_custom.conf`:

```ini
[transport-ms-teams]
type=transport
protocol=tls
bind=0.0.0.0:5061
external_signaling_address=YOUR.PUBLIC.IP    ; Your public IP address
external_signaling_port=5061                 ; External SIP TLS port
ms_signaling_address=sbc.example.com         ; Your FQDN hostname (REQUIRED)
```

### For standalone Asterisk systems

Edit `/etc/asterisk/pjsip.conf` and add or update your transport section:

```ini
[transport-ms-teams]
type=transport
protocol=tls
bind=0.0.0.0:5061
external_signaling_address=YOUR.PUBLIC.IP    ; Your public IP address
external_signaling_port=5061                 ; External SIP TLS port
ms_signaling_address=sbc.example.com         ; Your FQDN hostname (REQUIRED)
```

### Configuration parameters explained

- **`external_signaling_address`** - Set to your **public IP address** (not a hostname)
- **`external_signaling_port`** - External SIP port (typically `5061` for TLS)
- **`ms_signaling_address`** - Set to your **FQDN hostname** (not an IP address)
  - This FQDN must match your TLS certificate
  - This FQDN must match your MS Teams SBC configuration in Microsoft 365
  - Example: `sbc.example.com`, `teams-sbc.yourcompany.com`

### After configuration

Reload PJSIP to apply the changes:

```bash
# Via Asterisk CLI
asterisk -rx 'pjsip reload'

# Or via FreePBX
fwconsole restart
```

Verify your transport is configured:

```bash
asterisk -rx 'pjsip show transports'
```

## How it works

Unlike older versions that required hard-coding the FQDN into the source code and recompiling, these modules read the FQDN from your `pjsip.conf` configuration at runtime:

1. When Asterisk sends SIP messages to MS Teams, the PJSIP NAT module checks if `ms_signaling_address` is set on the transport
2. If set, it uses that FQDN in the SIP Contact and Via headers
3. If not set, it falls back to using `external_signaling_address` (IP-based behavior)

This means:
- ✅ The same compiled module works for everyone
- ✅ You can change your FQDN without recompiling
- ✅ Multiple deployments can use the same binary

## Version history

- **v2 and later**: Includes `ms_signaling_address` runtime configuration patch (current)
- **v1 (deprecated)**: Had hard-coded FQDNs - do not use

## Related projects

For automated installation and building from source, see:
- [MSTeams-FreePBX](https://github.com/Vince-0/MSTeams-FreePBX) - Installation script with source building and patch application

## Support

For issues, questions, or contributions, please open an issue on GitHub.
