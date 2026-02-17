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


## Related projects

For automated installation and building from source, see:
- [MSTeams-FreePBX](https://github.com/Vince-0/MSTeams-FreePBX) - Installation script with source building and patch application

## Runtime FQDN configuration (`ms_signaling_address`)



The patch adds a new transport option `ms_signaling_address`. Instead of hard-coding the FQDN into `res/res_pjsip_nat.c` at build time, the module reads the FQDN from your PJSIP transport configuration at runtime.

The key parameters on your `transport` object in `pjsip.conf` are:

- `external_signaling_address` – the **public IP address** of your SBC as seen by MS Teams (usually your WAN IP or load balancer VIP).
- `external_signaling_port` – the **external SIP port** forwarded from the internet to Asterisk (typically `5061` for TLS).
- `ms_signaling_address` – the **FQDN** MS Teams expects to see in SIP Contact and Via headers (this must match the FQDN you configure in Microsoft 365 and on your TLS certificate).

To configure these options:

1. Choose the FQDN that will represent your SBC to MS Teams, for example `sbc.example.com`. This FQDN:
   - Must resolve in public DNS to your SBC public IP.
   - Must appear in the certificate presented by Asterisk (CN or SAN).
   - Must match what you configure in the Microsoft Teams Direct Routing/SBC settings.
2. Determine the public IP and port that MS Teams will use to reach your SBC:
   - `external_signaling_address` = that public IP address.
   - `external_signaling_port` = the TLS SIP port you expose (commonly `5061`).
3. Edit your PJSIP transport configuration:
   - On a plain Asterisk system, edit `/etc/asterisk/pjsip.conf` and add or update the appropriate `transport` section.
   - On a FreePBX system, add the transport in the appropriate custom file (for example `/etc/asterisk/pjsip.transports_custom.conf`), rather than editing the FreePBX‑managed `pjsip.conf` directly.
4. Set `ms_signaling_address` to the SBC FQDN you chose in step 1.
5. Reload PJSIP (for example from the Asterisk CLI with `pjsip reload` or via FreePBX) so the new transport settings take effect.

Example transport stanza:

```ini
[transport-ms-teams]
type=transport
protocol=tls
external_signaling_address=203.0.113.10
external_signaling_port=5061
ms_signaling_address=sbc.example.com
```

If `ms_signaling_address` is not set, Asterisk continues to use the existing behaviour based on `external_signaling_address` and `external_signaling_port`.

The install script:

- Applies the `ms_signaling_address` patch to the Asterisk sources whenever it builds from source (both default FreePBX mode and `--asterisk-only`).
- Or downloads precompiled `res_pjsip_nat.so` modules that were built from patched sources (see below).

## Compiled PJSIP NAT modules for Asterisk 21, 22 and 23 on Debian 12

Precompiled `res_pjsip_nat.so` modules for multiple Asterisk versions are available in the following repository (organised by Asterisk major version). These modules are built from sources that include the `ms_signaling_address` runtime FQDN patch described above (v2 and later):


## Further Development

Operating system version and distribution options.

## Reference Links

## Run Time Patch

[Jose's](https://github.com/eagle26) run time [patch](https://github.com/asterisk/asterisk/compare/master...eagle26:asterisk:master)
