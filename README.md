# Splunk UF to Cribl Forwarding

Repository of Splunk Universal Forwarder (UF) app packages and example configuration to securely forward data to Cribl Stream and/or Splunk indexers. The apps ship sensible TLS defaults, optional mTLS, and an app-bundled CA trust path so deployments are predictable and self-contained.

## Repository structure
- **Cribl/CFG_uf_forward_cribl_windows**
  - Splunk UF app for Windows hosts.
  - Includes TLS defaults, optional mTLS, and Windows-specific SSL behavior.
  - See: `Cribl/CFG_uf_forward_cribl_windows/README.md` for details and commands.
- **Cribl/CFG_uf_forward_cribl_linux**
  - Splunk UF app for Linux hosts.
  - Includes TLS defaults and optional mTLS.
  - See: `Cribl/CFG_uf_forward_cribl_linux/README.md` for details and commands.

## What these apps configure
- `outputs.conf`
  - Define `tcpout` groups for Cribl (and optionally Splunk indexers) with TLS and server name verification.
  - Commented examples for selecting `defaultGroup` and enabling mTLS.
- `server.conf`
  - Point UF to an app-bundled CA bundle.
  - Windows app sets `sslRootCAPathHonoredOnWindows = true`.
- `certs/`
  - Place your CA bundle under `certs/ca/` (PEM).
  - Optional client certificate+key for mTLS.

## Prerequisites
- Splunk Universal Forwarder installed on target hosts.
- Cribl Stream receiver(s) reachable over the chosen TLS port (commonly 9997) and presenting a valid server certificate.
- A CA bundle that chains to the server certificate(s). Optional client certificate+key if enabling mTLS.

## Quick start
1. **Choose the app for your OS**
   - Windows: `Cribl/CFG_uf_forward_cribl_windows`
   - Linux: `Cribl/CFG_uf_forward_cribl_linux`
2. **Edit forwarding targets** in `default/outputs.conf` (or better, in `local/outputs.conf`):
   - Set `server = <host1:port>, <host2:port>` under the `cribl` (and/or `splunk`) group(s).
   - Select your `defaultGroup` (`cribl`, `splunk`, or `cribl,splunk` for dual-output if configured).
3. **Provide trust**
   - Place your CA bundle at:
     - Windows app: `certs\ca\ca_cert.pem`
     - Linux app: `certs/ca/ca_cert.pem`
   - If you change the path, update `sslRootCAPath` in `server.conf` (preferably in `local/server.conf`).
4. **Optional mTLS**
   - Place a PEM containing the client certificate and private key in the app's `certs/` path (see OS README for exact filename).
   - Uncomment and set `clientCert = ...` and `sslPassword = <passphrase>` (if the key is encrypted) in the relevant `tcpout` group(s).
5. **Deploy**
   - Copy the chosen app directory to the UF:
     - Windows: `%SPLUNK_HOME%\etc\apps\`
     - Linux: `$SPLUNK_HOME/etc/apps/`
   - Restart the Splunk Universal Forwarder service.

## Validate deployment
- Windows UF admin PowerShell:
  ```powershell
  "C:\Program Files\SplunkUniversalForwarder\bin\splunk.exe" btool outputs list --app=CFG_uf_forward_cribl_windows --debug
  "C:\Program Files\SplunkUniversalForwarder\bin\splunk.exe" list forward-server
  ```
- Linux UF admin shell:
  ```bash
  $SPLUNK_HOME/bin/splunk btool outputs list --app=CFG_uf_forward_cribl_linux --debug
  $SPLUNK_HOME/bin/splunk list forward-server
  ```
- Review UF logs for connectivity/SSL messages:
  - Windows: `%SPLUNK_HOME%\var\log\splunk\splunkd.log`
  - Linux: `$SPLUNK_HOME/var/log/splunk/splunkd.log`

## Notes
- Keep shipped defaults under `default/` and put site-specific changes in `local/` to preserve them across updates.
- Hostnames in `server =` should match the certificate CN/SAN when `sslVerifyServerName = true`.
- For detailed OS-specific guidance and examples, refer to the README within the selected app directory.
