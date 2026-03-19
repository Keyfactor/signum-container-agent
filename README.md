# About the Signum Container Agent
The Signum Container Agent is a base image that runs the Signum Agent service, and it can be modified as shown in the examples with additional signing tools for handling a variety of different scenarios.

Usage examples of several popular signing tools can be found in [signing-tool-examples](/signing-tool-examples/):
- [Cosign](/signing-tool-examples/cosign.md) — OCI/container image signing
- [Jarsigner](/signing-tool-examples/jarsigner.md) — Java JAR file signing
- [Jsign](/signing-tool-examples/jsign.md) — Windows Authenticode signing
- [OpenSSL](/signing-tool-examples/openssl.md) — General-purpose cryptographic operations
- [XMLSecTool](/signing-tool-examples/xmlsectool.md) — XML digital signatures

---

## Support for Signum Container Agent with Signing Tools
---
Signum Container Agent with Signing Tools is open source and supported on best effort level for this set of examples.  This means customers can report Bugs, Feature Requests, Documentation amendment or questions, as well as requests for customer information required for setup that needs Keyfactor access to obtain. Such requests do not follow normal SLA commitments for response or resolution. If you have a support issue, please open a support ticket via the Keyfactor Support Portal at https://support.keyfactor.com/

To report a problem or suggest a new feature, use the **[Issues](../../issues)** tab. If you want to contribute actual bug fixes or proposed enhancements, use the **[Pull requests](../../pulls)** tab.

---

## Running the Signum Container Agent Base Image
```sh
docker run --name signum-agent \
  -e "SIGNUM_HOSTNAME=A URL" \
  -e "SIGNUM_USERNAME=myuser@somedomain" \
  -e "SIGNUM_PASSWORD=$mycreds" \
  -e "SIGNUM_LOGLEVEL=HIGH" \
  -e "SIGNUM_LOGTYPE=FILE" \
  repo.keyfactor.com/images/signum-agent:4.80.1
```
```sh
docker exec -it signum-agent /bin/bash
```
```
signum-util lc
```
```
Subject CN     : Signum-RSA-3072
    Issuer CN      : DemoRoot-G2
    Valid Until    : 2029-04-23
    Valid From     : 2024-04-24
    Thumbprint     : 170570A1D56FBB5A4CC780B69ACAEF94010D5DAA
Subject CN     : Signum-RSA-4096
    Issuer CN      : DemoRoot-G2
    Valid Until    : 2029-04-23
    Valid From     : 2024-04-24
    Thumbprint     : 3AB5BFB91DFBB46CF765D5BEE51429618C4857DD
Subject CN     : Signum-RSA-2048
    Issuer CN      : DemoRoot-G2
    Valid Until    : 2030-02-05
    Valid From     : 2025-02-06
    Thumbprint     : F78AE7871FEF1D0CF3EFFB58E9CC85F261438D2B
```

---

## Environment Variable Reference

| Variable | Required | Description |
|---|---|---|
| `SIGNUM_HOSTNAME` | Yes | URL of the Signum server (e.g. `https://signum.example.com`) |
| `SIGNUM_USERNAME` | Yes* | Username for authenticating to the Signum server. |
| `SIGNUM_PASSWORD` | Yes* | Password for authenticating to the Signum server, if using certificate authentication it's the password of the .p12 file. |
| `SIGNUM_LOGLEVEL` | No | Controls log verbosity. Valid values: `LOW`, `MEDIUM`, `HIGH`. `HIGH` produces the most output and should be only used for troubleshooting. |
| `SIGNUM_LOGTYPE` | No | Controls log destination. Valid values: `FILE` (writes to a log file inside the container), `STDOUT` (writes to stdout). |
| `SIGNUM_AGENTID` | No | Allows to pre-define the agentID that will be reported to the server. The default value is `AAAAA-BBBBB-CCCCC-DDDDD`. The provided AgentID must have the same format. |
| `SIGNUM_HTTPS_PROXY` | No | The proxy to be used for connecting to the Signum instance. Check the [official documentation](https://docs.keyfactor.com/Signum-SaaS/latest/linux-agent#id-(4.70.1)LinuxAgent-Setup/) for more information |
| `SIGNUM_CERTIFICATE_PATH` | No* | The absolute certificate path inside the container used to connect to the Signum instance. Must be a .p12 file. Check the [official documentation](https://docs.keyfactor.com/Signum-SaaS/latest/linux-agent#id-(4.70.1)LinuxAgent-AuthenticatewithCertificate/) for more information |
| `SIGNUM_WAF_PORT` | No* | Provide the WAF port configured in the Administration Console. Required only when using certificate-based login behind a WAF. |

---
*If using certificate authentication, you need to provide 'SIGNUM_CERTIFICATE_PATH', 'SIGNUM_WAF_PORT', and 'SIGNUM_PASSWORD'. For user-password login, provide 'SIGNUM_USERNAME' and 'SIGNUM_PASSWORD'.

### User-Password Authentication
```bash
docker run --name signum-agent -d \
  -e "SIGNUM_HOSTNAME=A URL" \
  -e "SIGNUM_USERNAME=myuser@somedomain" \
  -e "SIGNUM_PASSWORD=$mycreds" \
  -e "SIGNUM_LOGLEVEL=HIGH" \
  -e "SIGNUM_LOGTYPE=FILE" \
  repo.keyfactor.com/images/signum-agent:4.80.1
```

### Certificate Authentication

```bash
docker run --name signum-agent -d \
  -v $PWD/loginCertificate.p12:/mnt/loginCertificate.p12 \
  -e "SIGNUM_HOSTNAME=A URL" \
  -e "SIGNUM_CERTIFICATE_PATH=/mnt/loginCertificate.p12" \
  -e "SIGNUM_PASSWORD=$myCertificatePassword" \
  -e "SIGNUM_LOGLEVEL=HIGH" \
  -e "SIGNUM_LOGTYPE=FILE" \
  -e "SIGNUM_WAF_PORT=$myWafPort" \
  repo.keyfactor.com/images/signum-agent:4.80.1
```

## Modifying the Base Image
Add the PKCS#11-based signing tools your team would like to use.
See [dockerfile-examples](/dockerfile-examples/) for some examples. In production you should verify the sources of external repositories.

### PKCS#11 Configuration File

Most signing tools require a PKCS#11 configuration file that points to the Keyfactor PKCS#11 library. This file is created during image build and placed at `/etc/keyfactor/signumpkcs11.cfg`. Its contents are:

```
name = SignumPKCS11
library = /usr/lib/libsignumpkcs11.so
description = Keyfactor PKCS#11 interface for SmartCard
```

This file is referenced by signing tools (e.g. `keytool`, `jarsigner`, `jsign`) via the `-providerArg` or `--keystore` flags. See the individual signing tool examples for usage.

### Example: Adding Jsign to the Base Image

Build the extended image:
```bash
docker buildx build -f dockerfile-examples/dockerfile-jsign -t signum-container-agent:jsign .
```

Run a container with a local directory mounted for files to sign:
```bash
docker run --name signum-agent -d \
  -v $PWD/filestosign/:/mnt/filestosign \
  -e "SIGNUM_HOSTNAME=A URL" \
  -e "SIGNUM_USERNAME=myuser@somedomain" \
  -e "SIGNUM_PASSWORD=$mycreds" \
  -e "SIGNUM_LOGLEVEL=HIGH" \
  -e "SIGNUM_LOGTYPE=FILE" \
  signum-container-agent:jsign
```

Open a shell in the running container:
```bash
docker exec -it signum-agent /bin/bash
```

List key objects stored in the PKCS#11 keystore:
```sh
keytool -list -storetype PKCS11 -providerClass sun.security.pkcs11.SunPKCS11 -providerArg signumpkcs11.cfg -storepass NONE
```
```
Keystore type: PKCS11
Keystore provider: SunPKCS11-SignumPKCS11

Your keystore contains 5 entries

170570A1D56FBB5A4CC780B69ACAEF94010D5DAA - Certificate, PrivateKeyEntry,
Certificate fingerprint (SHA-256): 1C:3B:0B:5E:B7:7F:29:29:87:4E:7D:BC:77:11:D9:7F:FF:06:0B:C3:F2:F9:DE:02:8E:72:C6:87:4E:CE:B2:94
3AB5BFB91DFBB46CF765D5BEE51429618C4857DD - Certificate, PrivateKeyEntry,
Certificate fingerprint (SHA-256): 97:58:8B:1B:C4:D5:19:3C:C6:5F:3F:4A:73:11:53:17:98:D4:A7:E9:FD:A3:3D:88:B0:9F:09:EB:77:D9:23:F0
3BFA85A455F54CE76D74B52F6B4226C00299CF7D - Certificate, PrivateKeyEntry,
Certificate fingerprint (SHA-256): 35:5C:22:86:9F:92:19:CA:80:38:F3:A9:D0:7A:20:BD:0B:53:5E:20:1C:29:A9:39:40:71:F0:68:12:88:E3:26
50D63698EF043051EFB0B7E5280EDACF35A09B29 - Certificate, PrivateKeyEntry,
Certificate fingerprint (SHA-256): B9:D3:D7:70:1E:DA:11:3C:2B:27:65:9E:64:73:6F:9F:0B:FB:7A:F5:77:9D:81:BF:95:A5:71:D2:96:0B:D0:1A
```

The alias used in signing commands is the certificate thumbprint followed by ` - Certificate` (e.g. `3AB5BFB91DFBB46CF765D5BEE51429618C4857DD - Certificate`).

Sign a file using Jsign:
```
java -jar jsign-6.0.jar --keystore /etc/keyfactor/signumpkcs11.cfg --storetype PKCS11 --alias "3AB5BFB91DFBB46CF765D5BEE51429618C4857DD - Certificate" /mnt/filestosign/example-script.ps1
```

---

## Troubleshooting

### Agent fails to start / cannot connect to Signum server
- Verify `SIGNUM_HOSTNAME` is reachable from within the container: `curl -v $SIGNUM_HOSTNAME`
- Check that `SIGNUM_USERNAME` and `SIGNUM_PASSWORD` are correct
- Set `SIGNUM_LOGLEVEL=HIGH` to get detailed logs

### `signum-util lc` returns no certificates
- Confirm the agent has successfully authenticated (check logs with `SIGNUM_LOGLEVEL=HIGH`)
- Verify the user account has certificates assigned to it in the Signum server

### PKCS#11 library not found
- Confirm `/usr/lib/libsignumpkcs11.so` exists inside the container: `ls -la /usr/lib/libsignumpkcs11.so`
- Confirm `/etc/keyfactor/signumpkcs11.cfg` exists and contains the correct library path
- Confirm `/usr/share/p11-kit/modules/signum.module` exists and contains the correct library path

### `keytool` returns no entries
- The agent must be running and authenticated before the PKCS#11 keystore is populated
- Wait a few seconds after container start for the agent to connect, then retry

### Permission denied errors
- The container runs as user `10001` (non-root). Ensure any host-mounted volumes grant read/write access to this user: `chown -R 10001 ./filestosign`
  
### user doesn't have configuration file or credentials are not valid
- The container is started and configured for the user `10001` (non-root). Ensure any time you are running a command with `docker exec $containerName` you are not overriding the default user for the container.
