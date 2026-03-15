# MSTeams-PJSIPNAT

**USE AT YOUR OWN RISK**

Precompiled patched PJSIP modules (`res_pjsip.so` + `res_pjsip_nat.so`) for Asterisk 21, 22 and 23 on Debian 12 with **runtime FQDN configuration** support.

## What's included

Both `res_pjsip.so` and `res_pjsip_nat.so` are built from Asterisk sources patched with the `ms_signaling_address` runtime configuration feature. These two modules share the `ast_sip_transport` struct and must always be deployed together as a matched pair — deploying only one will cause crashes or load failures.

The patch modifies three source files:
- `include/asterisk/res_pjsip.h` — adds the `ms_signaling_address` string field to `ast_sip_transport`
- `res/res_pjsip/config_transport.c` — registers the `ms_signaling_address` config option (compiled into `res_pjsip.so`)
- `res/res_pjsip_nat.c` — reads the field and uses it in SIP Contact/Via headers (compiled into `res_pjsip_nat.so`)

This allows you to configure the FQDN for MS Teams Direct Routing at runtime via `pjsip.conf` instead of hard-coding it at compile time.

**Key features:**
- ✅ **No hard-coded FQDN** — the modules are generic and work for any deployment
- ✅ **Runtime configuration** — set your FQDN in `pjsip.conf` using the `ms_signaling_address` parameter
- ✅ **No recompilation needed** — change your FQDN by editing config and reloading PJSIP

**IMPORTANT:** These modules **require** you to configure `ms_signaling_address` in your PJSIP transport configuration. See the configuration section below.

## Available prebuilt bundles

Each bundle contains both `res_pjsip.so` and `res_pjsip_nat.so` built from the exact same source tree.

| Platform | Available versions |
|---|---|
| `debian12-amd64` | `asterisk-21.12.1`, `asterisk-22.7.0`, `asterisk-22.8.2`, `asterisk-23.2.2` |

> If your exact running version is not listed, use the [MSTeams-FreePBX](https://github.com/Vince-0/MSTeams-FreePBX) installer to build from source for your version, or use `--downloadonly` which will fall back to the nearest available major-version bundle with ABI verification.

## Installation

These modules are typically installed in the Asterisk modules directory. The path depends on your CPU architecture:

| Architecture | Modules directory |
|---|---|
| x86_64 (amd64) | `/usr/lib/x86_64-linux-gnu/asterisk/modules/` |
| aarch64 (arm64) | `/usr/lib/aarch64-linux-gnu/asterisk/modules/` |
| armv7l (armhf)  | `/usr/lib/arm-linux-gnueabihf/asterisk/modules/` |

On a standard FreePBX Debian 12 install: https://github.com/FreePBX/sng_freepbx_debian_install

### Option A: Use the automated installer (recommended)

The [MSTeams-FreePBX](https://github.com/Vince-0/MSTeams-FreePBX) installer script handles downloading or building the module pair (`res_pjsip.so` + `res_pjsip_nat.so`), SSL certificate setup, and configuration automatically. It auto-detects your architecture. Use `--downloadonly` to fetch a prebuilt bundle from this repo, or run without that flag to build from source. See that repo for full details.

### Option B: Download prebuilt module pair manually

**Both `res_pjsip.so` and `res_pjsip_nat.so` must be downloaded and installed together.** Check the [Available prebuilt bundles](#available-prebuilt-bundles) table above for the exact version directory matching your running Asterisk (`asterisk -V`).

#### Step 1: Download both modules

Replace `<EXACT-VER>` with your exact running version (e.g. `22.8.2`) and `<ARCH>` with your Debian architecture (`amd64`):

```bash
# Example: amd64, Asterisk 22.8.2
MODULES=/usr/lib/x86_64-linux-gnu/asterisk/modules
BASE=https://github.com/Vince-0/MSTeams-PJSIPNAT/raw/main/prebuilt/debian12-amd64/asterisk-22.8.2

wget -O $MODULES/res_pjsip.so.MSTEAMS     $BASE/res_pjsip.so
wget -O $MODULES/res_pjsip_nat.so.MSTEAMS $BASE/res_pjsip_nat.so
```

#### Step 2: Back up the originals (first run only)

```bash
mv $MODULES/res_pjsip.so     $MODULES/res_pjsip.so.ORIG
mv $MODULES/res_pjsip_nat.so $MODULES/res_pjsip_nat.so.ORIG
```

#### Step 3: Install both patched modules

```bash
cp -v $MODULES/res_pjsip.so.MSTEAMS     $MODULES/res_pjsip.so
cp -v $MODULES/res_pjsip_nat.so.MSTEAMS $MODULES/res_pjsip_nat.so
```

### Option C: Build from source

If no prebuilt bundle is available for your architecture or exact version, apply the patch from [MSTeams-FreePBX](https://github.com/Vince-0/MSTeams-FreePBX) and build. After building, copy **both** modules (do **not** run `make install` on a FreePBX system — that would overwrite FreePBX-managed binaries):

```bash
# After patch, ./configure && make inside the Asterisk source tree:
cp -v res/res_pjsip.so     $MODULES/res_pjsip.so
cp -v res/res_pjsip_nat.so $MODULES/res_pjsip_nat.so
```

### After installation: Restart Asterisk/FreePBX

Because `res_pjsip.so` is a core module that cannot be hot-reloaded safely, a full restart is required:

```bash
# FreePBX
fwconsole restart

# Standalone Asterisk
systemctl restart asterisk
```

### After installation: Verify both modules loaded

```bash
asterisk -rx 'module show like res_pjsip.so'
asterisk -rx 'module show like res_pjsip_nat.so'
```

Expected output for each:
```
Module                         Description                              Use Count  Status      Support Level
res_pjsip.so                   Basic SIP resource                       ...        Running              core
res_pjsip_nat.so               PJSIP NAT Support                        0          Running              core
```

## SSL Certificate Requirement

MS Teams Direct Routing **requires** a valid TLS certificate issued by a trusted public CA for your FQDN. Self-signed certificates are not accepted by Microsoft.

> **⚠️ RSA certificates only.** MS Teams Direct Routing does **not** support ECDSA certificates. Using an ECDSA certificate (e.g. one issued by Let's Encrypt's E5/E6/E7 CA) will result in a TLS handshake failure: `no shared cipher`. You must explicitly request an RSA certificate.

[Let's Encrypt](https://letsencrypt.org/) (via **certbot**) is the recommended free option. Force RSA with `--key-type rsa`:

```bash
# Install certbot
apt-get install -y certbot

# Obtain an RSA certificate (stop apache2 first if it's running on port 80)
systemctl stop apache2
certbot certonly --standalone --non-interactive --agree-tos \
  --key-type rsa --rsa-key-size 2048 \
  --email admin@example.com -d sbc.example.com
systemctl start apache2
```

Verify the issued certificate is RSA (not ECDSA) before configuring Asterisk:

```bash
openssl x509 -in /etc/letsencrypt/live/sbc.example.com/cert.pem -noout -text \
  | grep "Public Key Algorithm"
# Expected: rsaEncryption
# If you see id-ecPublicKey, re-issue with --key-type rsa
```

Certbot installs certificates to `/etc/letsencrypt/live/<your-fqdn>/`. Reference them directly in `pjsip.conf` using `fullchain.pem` and `privkey.pem` (see configuration section below). Certbot renews certificates automatically via a systemd timer or cron job — no manual file copying is needed when referencing the live symlinks directly.

> **Note:** The [MSTeams-FreePBX](https://github.com/Vince-0/MSTeams-FreePBX) installer automates certificate issuance (RSA), verification, and configuration, and warns if an ECDSA certificate is detected.

## Required Configuration

**CRITICAL:** These modules use runtime FQDN configuration. You **must** configure the `ms_signaling_address` parameter in your PJSIP transport configuration.

### For FreePBX systems

Create or edit `/etc/asterisk/pjsip.transports_custom.conf`:

```ini
[transport-ms-teams]
type=transport
protocol=tls
bind=0.0.0.0:5061
external_signaling_address=203.0.113.10                              ; Your public IP address
external_signaling_port=5061                                         ; External SIP TLS port
ms_signaling_address=sbc.example.com                                 ; FQDN added by patch — must match TLS cert CN
cert_file=/etc/letsencrypt/live/sbc.example.com/fullchain.pem        ; RSA cert (fullchain includes intermediate)
priv_key_file=/etc/letsencrypt/live/sbc.example.com/privkey.pem      ; RSA private key
method=tlsv1_2                                                       ; MS Teams requires TLS 1.2
verify_server=no                                                     ; Do not verify MS Teams server cert
```

### For standalone Asterisk systems

Edit `/etc/asterisk/pjsip.conf` and add or update your transport section:

```ini
[transport-ms-teams]
type=transport
protocol=tls
bind=0.0.0.0:5061
external_signaling_address=203.0.113.10                              ; Your public IP address
external_signaling_port=5061                                         ; External SIP TLS port
ms_signaling_address=sbc.example.com                                 ; FQDN added by patch — must match TLS cert CN
cert_file=/etc/letsencrypt/live/sbc.example.com/fullchain.pem        ; RSA cert (fullchain includes intermediate)
priv_key_file=/etc/letsencrypt/live/sbc.example.com/privkey.pem      ; RSA private key
method=tlsv1_2                                                       ; MS Teams requires TLS 1.2
verify_server=no                                                     ; Do not verify MS Teams server cert
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
