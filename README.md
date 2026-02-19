# MSTeams-PJSIPNAT

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

These modules are typically installed in the Asterisk modules directory. The path depends on your CPU architecture:

| Architecture | Modules directory |
|---|---|
| x86_64 (amd64) | `/usr/lib/x86_64-linux-gnu/asterisk/modules/` |
| aarch64 (arm64) | `/usr/lib/aarch64-linux-gnu/asterisk/modules/` |
| armv7l (armhf)  | `/usr/lib/arm-linux-gnueabihf/asterisk/modules/` |

On a standard FreePBX Debian 12 install: https://github.com/FreePBX/sng_freepbx_debian_install

### Option A: Use the automated installer (recommended)

The [MSTeams-FreePBX](https://github.com/Vince-0/MSTeams-FreePBX) installer script handles downloading or building the module, SSL certificate setup, and configuration automatically. It auto-detects your architecture. See that repo for full details.

### Option B: Download prebuilt module manually

#### Step 1: Download the module

Replace `<ARCH>` with your Debian architecture (`amd64`, `arm64`, `armhf`, `i386`, `ppc64el`) and `<VER>` with your Asterisk major version (`21`, `22`, or `23`):

```bash
# Example: amd64, Asterisk 22
MODULES=/usr/lib/x86_64-linux-gnu/asterisk/modules
wget -O $MODULES/res_pjsip_nat.so.MSTEAMS \
  https://github.com/Vince-0/MSTeamsPJSIPNAT_Debian12/raw/main/prebuilt/debian12-amd64/asterisk-22/res_pjsip_nat.so

# Example: arm64, Asterisk 22
MODULES=/usr/lib/aarch64-linux-gnu/asterisk/modules
wget -O $MODULES/res_pjsip_nat.so.MSTEAMS \
  https://github.com/Vince-0/MSTeamsPJSIPNAT_Debian12/raw/main/prebuilt/debian12-arm64/asterisk-22/res_pjsip_nat.so
```

#### Step 2: Backup the original module

```bash
mv $MODULES/res_pjsip_nat.so $MODULES/res_pjsip_nat.so.ORIG
```

#### Step 3: Install the custom module

```bash
cp -v $MODULES/res_pjsip_nat.so.MSTEAMS $MODULES/res_pjsip_nat.so
```

### Option C: Build from source

If no prebuilt module is available for your architecture, or you want to build it yourself, clone the Asterisk sources, apply the patch from [MSTeams-FreePBX](https://github.com/Vince-0/MSTeams-FreePBX), and build. After building, copy only the module (do **not** run `make install` on a FreePBX system — that would overwrite FreePBX-managed binaries):

```bash
# After ./configure && make inside the Asterisk source tree:
cp -v res/res_pjsip_nat.so $MODULES/res_pjsip_nat.so
```

### After installation: Reload the module

```bash
asterisk -rx 'module unload res_pjsip_nat.so'
asterisk -rx 'module load res_pjsip_nat.so'
```

### After installation: Verify the module loaded

```bash
asterisk -rx 'module show like res_pjsip_nat.so'
```

Expected output:
```
Module                         Description                              Use Count  Status      Support Level
res_pjsip_nat.so               PJSIP NAT Support                        0          Running              core
1 modules loaded
```

## SSL Certificate Requirement

MS Teams Direct Routing **requires** a valid TLS certificate issued by a trusted public CA for your FQDN. Self-signed certificates are not accepted by Microsoft.

[Let's Encrypt](https://letsencrypt.org/) (via **certbot**) is the recommended free option:

```bash
# Install certbot
apt-get install -y certbot

# Obtain a certificate (stop apache2 first if it's running on port 80)
systemctl stop apache2
certbot certonly --standalone --non-interactive --agree-tos \
  --email admin@example.com -d sbc.example.com
systemctl start apache2
```

Certbot installs certificates to `/etc/letsencrypt/live/<your-fqdn>/`. Copy the relevant files to Asterisk's SSL directory:

```bash
mkdir -p /etc/asterisk/ssl
cp /etc/letsencrypt/live/sbc.example.com/fullchain.pem /etc/asterisk/ssl/cert.crt
cp /etc/letsencrypt/live/sbc.example.com/cert.pem      /etc/asterisk/ssl/ca.crt
cp /etc/letsencrypt/live/sbc.example.com/privkey.pem   /etc/asterisk/ssl/privkey.crt
```

Certbot automatically renews certificates via a systemd timer or cron job. After renewal, re-copy the updated files and reload Asterisk.

> **Note:** The [MSTeams-FreePBX](https://github.com/Vince-0/MSTeams-FreePBX) installer automates all of this — certificate issuance, file copying, and configuration — using certbot.

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

## Further Development

Operating system version and distribution options.

## Reference Links

### Run Time Patch

[Jose's](https://github.com/eagle26) run time [patch](https://github.com/asterisk/asterisk/compare/master...eagle26:asterisk:master)

### Related projects

For automated installation and building from source, see:

- [MSTeams-FreePBX](https://github.com/Vince-0/MSTeams-FreePBX) - Installation script with source building and patch application
