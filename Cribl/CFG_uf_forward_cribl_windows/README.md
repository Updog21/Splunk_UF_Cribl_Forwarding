# CFG_uf_forward_cribl_windows

## Purpose
Configure a Splunk Universal Forwarder (Windows) to securely forward data to Cribl Stream and/or Splunk indexers. The app provides sane TLS defaults and optional mTLS, with an app-bundled CA trust path.

## What this app configures
- **`outputs.conf`**
  - `[tcpout]` with commented `defaultGroup` options so you can choose where to send data:
    - `cribl` only, `splunk` only, or `cribl,splunk` for dual-output.
  - `[tcpout:cribl]` example using TLS to Cribl nodes on port 9997 with server certificate and server name verification enabled. Optional mTLS lines are provided (commented).
  - `[tcpout:splunk]` example using TLS to Splunk indexers with certificate and name verification. `compressed = false` is set to avoid double-compression in some environments.
- **`server.conf`**
  - `[sslConfig]` that points Splunk to a CA certificate bundled in the app at `certs\ca\ca_cert.pem`.
  - `caTrustStore = splunk`.
  - `sslRootCAPathHonoredOnWindows = true` to ensure the app-provided CA file is honored on Windows.

## File layout
- `default/outputs.conf` – Forwarding destinations and TLS/mTLS settings.
- `default/server.conf` – Global SSL settings (CA trust path, Windows override).
- `certs/ca/ca_cert.pem` – Place your trusted CA bundle here (PEM). Create directories as needed.
- `certs/client/client_cert.pem` – Optional client certificate+key PEM if enabling mTLS.
- `local/` – Put site-specific overrides here to preserve them across updates.

## How to use
1. **Pick destinations**: In `outputs.conf`, uncomment the relevant `defaultGroup` line under `[tcpout]` to choose `cribl`, `splunk`, or `cribl,splunk`.
2. **Set servers**: Update the `server =` lines under `[tcpout:cribl]` and `[tcpout:splunk]` with your actual hosts and ports.
3. **Trust the CA**: Place your CA bundle at `certs\ca\ca_cert.pem` (PEM). If you use a different path/name, update `sslRootCAPath` in `default/server.conf` (or better, in `local/server.conf`).
4. **Optional mTLS**:
   - Place a PEM file containing the client certificate and private key at `certs\client\client_cert.pem`.
   - In `[tcpout:cribl]` (and/or `[tcpout:splunk]` if you add it), uncomment `clientCert = ...` and set `sslPassword = <passphrase>` if the key is encrypted.
5. **Deploy**: Copy the entire `CFG_uf_forward_cribl_windows` directory to `%SPLUNK_HOME%\etc\apps\` on the UF and restart the Splunk Universal Forwarder service.

## Validate
- From a Splunk UF admin CLI:
  ```powershell
  "C:\Program Files\SplunkUniversalForwarder\bin\splunk.exe" btool outputs list --app=CFG_uf_forward_cribl_windows --debug
  "C:\Program Files\SplunkUniversalForwarder\bin\splunk.exe" list forward-server
  ```
- Check `%SPLUNK_HOME%\var\log\splunk\splunkd.log` for SSL handshake or connectivity messages.

## Notes
- Keep the shipped settings in `default/` and put environment-specific changes in `local/` so updates to this app don’t overwrite your changes.
- The example hostnames and ports are placeholders. Replace them with your environment’s details.
