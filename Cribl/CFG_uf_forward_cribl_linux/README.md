# CFG_uf_forward_cribl_linux

## Purpose
Configure a Splunk Universal Forwarder (Linux) to securely forward data to Cribl Stream. The app ships TLS defaults and optional mTLS, with an app-bundled CA trust path.

## What this app configures
- **`outputs.conf`**
  - `[tcpout]` with commented `defaultGroup` examples.
  - `[tcpout:cribl]` example using TLS to Cribl nodes on port 9997 with server certificate and server name verification enabled. Optional mTLS lines are provided (commented).
  - Note: This Linux app includes only the `cribl` group by default. If you also need to forward to Splunk indexers directly, add a `[tcpout:splunk]` stanza (mirroring your environment) and set `defaultGroup = splunk` or `cribl,splunk` as appropriate.
- **`server.conf`**
  - `[sslConfig]` that points Splunk to a CA certificate bundled in the app at `certs/ca/ca_cert.pem` and sets `caTrustStore = splunk`.

## File layout
- `default/outputs.conf` – Forwarding destination(s) and TLS/mTLS settings.
- `default/server.conf` – Global SSL settings (CA trust path).
- `certs/ca/ca_cert.pem` – Place your trusted CA bundle here (PEM). Create directories as needed.
- `certs/client_cert.pem` – Optional client certificate+key PEM if enabling mTLS.
- `local/` – Put site-specific overrides here to preserve them across updates.

## How to use
1. **Pick destination(s)**: In `outputs.conf`, uncomment the relevant `defaultGroup` line under `[tcpout]`. With the shipped file, `cribl` is the defined group.
2. **Set servers**: Update the `server =` line under `[tcpout:cribl]` with your actual Cribl worker/receiver hosts and ports.
3. **Trust the CA**: Place your CA bundle at `certs/ca/ca_cert.pem` (PEM). If you use a different path/name, update `sslRootCAPath` in `default/server.conf` (or better, in `local/server.conf`).
4. **Optional mTLS**:
   - Place a PEM with the client certificate and private key at `certs/client_cert.pem`.
   - In `[tcpout:cribl]`, uncomment `clientCert = ...` and set `sslPassword = <passphrase>` if the key is encrypted.
5. **Deploy**: Copy the entire `CFG_uf_forward_cribl_linux` directory to `$SPLUNK_HOME/etc/apps/` on the UF host and restart the Splunk Universal Forwarder service.

## Environment-specific settings to update
- **Cribl targets (hosts and ports)**
  - Edit `default/outputs.conf` in `[tcpout:cribl]` and set `server = host1.example.com:9997, host2.example.com:9997` to match your Cribl workers/receiver(s).
  - You can specify a comma-separated list for load balancing/failover.
  - Keep `useSSL = true`, `sslVerifyServerCert = true`, and `sslVerifyServerName = true` unless you are testing. Ensure the FQDNs you use match the CN/SAN on the server certificates.
  - Adjust the port if your Cribl receiver listens on a non-default port.
- **Default output group**
  - Under `[tcpout]`, uncomment `defaultGroup = cribl` (or add `,splunk` if you configure a Splunk group as well). Dual-output will duplicate events to both destinations.
- **Compression**
  - If needed, toggle `compressed = false` under `[tcpout:cribl]` to avoid double compression in some environments. Leave it commented or set appropriately for your pipeline.
- **mTLS (optional)**
  - Place the client cert+key PEM at `certs/client_cert.pem`, then uncomment `clientCert = $SPLUNK_HOME/etc/apps/CFG_uf_forward_cribl_linux/certs/client_cert.pem` and set `sslPassword = <passphrase>` if the key is encrypted.
- **CA trust**
  - Put your CA bundle at `certs/ca/ca_cert.pem`. If you use a different path/name, update `sslRootCAPath` in `server.conf` (preferably in `local/server.conf`).

## Validate
- From a Splunk UF admin CLI:
  ```bash
  $SPLUNK_HOME/bin/splunk btool outputs list --app=CFG_uf_forward_cribl_linux --debug
  $SPLUNK_HOME/bin/splunk list forward-server
  ```
- Check `$SPLUNK_HOME/var/log/splunk/splunkd.log` for SSL handshake or connectivity messages.

## Notes
- Keep the shipped settings in `default/` and put environment-specific changes in `local/` so updates to this app don’t overwrite your changes.
- The example hostnames and ports are placeholders. Replace them with your environment’s details.
