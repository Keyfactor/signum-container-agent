# About the Signum Container Agent
The Signum Container Agent is a base image that runs the Signum Agent service, and it can be modified as shown in the examples with additional signing tools for handling a variety of different scenarios.

Usage examples of several popular signing tools can be found in [signing-tool-examples](/signing-tool-examples/):
- [Cosign](/signing-tool-examples/cosign.md) — OCI/container image signing
- [Jarsigner](/signing-tool-examples/jarsigner.md) — Java JAR file signing
- [Jsign](/signing-tool-examples/jsign.md) — Windows Authenticode signing
- [OpenSC (pkcs11-tool)](/signing-tool-examples/opensc.md) — PKCS#11 token inspection and debugging
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
  repo.keyfactor.com/images/signum-agent:4.80.2
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
| `SIGNUM_HOSTNAME` | Yes | URL of the Signum or SignServer host (e.g. `https://signum.example.com`) |
| `SIGNUM_BACKEND` | No | Backend type: `SIGNUM` (default) or `SIGNSERVER`. Requires image `4.80.2` or later. |
| `SIGNUM_USERNAME` | Yes* | Username for authenticating to the Signum server (Signum backend only). |
| `SIGNUM_PASSWORD` | Yes* | Password for authenticating to the Signum server, if using certificate authentication it's the password of the .p12 file. |
| `SIGNUM_LOGLEVEL` | No | Controls log verbosity. Valid values: `LOW`, `MEDIUM`, `HIGH`. `HIGH` produces the most output and should be only used for troubleshooting. |
| `SIGNUM_LOGTYPE` | No | Controls log destination. Valid values: `FILE` (writes to a log file inside the container), `STDOUT` (writes to stdout). |
| `SIGNUM_AGENTID` | No | Allows to pre-define the agentID that will be reported to the server. The default value is `AAAAA-BBBBB-CCCCC-DDDDD`. The provided AgentID must have the same format. |
| `SIGNUM_HTTPS_PROXY` | No | The proxy to be used for connecting to the Signum instance. Check the [official documentation](https://docs.keyfactor.com/Signum-SaaS/latest/linux-agent#id-(4.90.1)LinuxAgent-Setup/) for more information |
| `SIGNUM_CERTIFICATE_PATH` | No* | The absolute certificate path inside the container used to connect to the Signum instance. Must be a .p12 file. Check the [official documentation](https://docs.keyfactor.com/Signum-SaaS/latest/linux-agent#id-(4.90.1)LinuxAgent-AuthenticatewithCertificate/) for more information |
| `SIGNUM_WAF_PORT` | No* | Provide the WAF port configured in the Administration Console. Required only when using certificate-based login behind a WAF. |
| `SIGNUM_TLS_TRUSTED_CA` | No | The absolute path inside the container to a PEM file containing the trusted CA certificate for the server's TLS certificate. When set, the TLS validation mode automatically switches to `PinnedCA`. See [Trusted CA (Pinned CA)](#trusted-ca-pinned-ca). |

---
*If using certificate authentication, you need to provide 'SIGNUM_CERTIFICATE_PATH', 'SIGNUM_WAF_PORT', and 'SIGNUM_PASSWORD'. For user-password login, provide 'SIGNUM_USERNAME' and 'SIGNUM_PASSWORD'. The `SIGNSERVER` backend is certificate-based and requires 'SIGNUM_CERTIFICATE_PATH' and 'SIGNUM_PASSWORD' (username/password login is not supported).

### User-Password Authentication
```bash
docker run --name signum-agent -d \
  -e "SIGNUM_HOSTNAME=A URL" \
  -e "SIGNUM_USERNAME=myuser@somedomain" \
  -e "SIGNUM_PASSWORD=$mycreds" \
  -e "SIGNUM_LOGLEVEL=HIGH" \
  -e "SIGNUM_LOGTYPE=FILE" \
  repo.keyfactor.com/images/signum-agent:4.80.2
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
  repo.keyfactor.com/images/signum-agent:4.80.2
```

### SignServer Authentication

> Requires image `4.80.2` or later.

To connect to a SignServer backend instead of Signum, set `SIGNUM_BACKEND=SIGNSERVER`. This backend authenticates with a client certificate (`.p12`); username/password login is not supported. Mount the certificate into the container and point `SIGNUM_CERTIFICATE_PATH` at it.

```bash
docker run --name signum-agent -d \
  -v $PWD/loginCertificate.p12:/mnt/loginCertificate.p12 \
  -e "SIGNUM_HOSTNAME=A URL" \
  -e "SIGNUM_BACKEND=SIGNSERVER" \
  -e "SIGNUM_CERTIFICATE_PATH=/mnt/loginCertificate.p12" \
  -e "SIGNUM_PASSWORD=$myCertificatePassword" \
  -e "SIGNUM_LOGLEVEL=HIGH" \
  -e "SIGNUM_LOGTYPE=FILE" \
  repo.keyfactor.com/images/signum-agent:4.80.2
```

### Trusted CA (Pinned CA)

To make the agent validate the server's TLS certificate against a specific CA, mount the CA certificate (PEM) into the container and point `SIGNUM_TLS_TRUSTED_CA` at it. Providing a trusted CA automatically changes the TLS validation mode to `PinnedCA`; no additional setting is required.

```bash
docker run --name signum-agent -d \
  -v $PWD/trustedCa.pem:/tmp/trustedCa.pem \
  -e "SIGNUM_HOSTNAME=A URL" \
  -e "SIGNUM_USERNAME=myuser@somedomain" \
  -e "SIGNUM_PASSWORD=$mycreds" \
  -e "SIGNUM_TLS_TRUSTED_CA=/tmp/trustedCa.pem" \
  -e "SIGNUM_LOGLEVEL=HIGH" \
  -e "SIGNUM_LOGTYPE=FILE" \
  repo.keyfactor.com/images/signum-agent:4.80.2
```

For more information about the validation modes, check the [official documentation](https://docs.keyfactor.com/Signum-SaaS/latest/linux-agent#id-(4.90.1)).

## Deploying on Kubernetes

Two things are easy to miss when writing a Kubernetes manifest for the Signum Container Agent:

1. **You must set `securityContext.runAsUser: 10001` on the app container.** The image is built to run as UID `10001`, but Kubernetes does not infer this from the image — if you omit it the pod may start as a different user and the agent will not find its configuration or credentials.
2. **If you need to add a custom CA to the trust store** (e.g. the Signum server presents a certificate chained to an internal CA), you have to run `update-ca-trust extract` as root. The image runs as non-root, so do it in an init container and share the results with the app container via `emptyDir` volumes mounted at `/etc/pki/ca-trust/source/anchors/` and `/etc/pki/ca-trust/extracted/`. Writing into the image's baked-in trust store from the app container will fail or will not persist.

### Example manifest

The manifest below deploys the agent with both points addressed. It injects a CA certificate from a `ConfigMap` named `my-custom-ca-cert` and reads `SIGNUM_USERNAME` / `SIGNUM_PASSWORD` from a `Secret` named `<DEPLOYMENT_NAME>-signum-creds`. Replace the `<PLACEHOLDERS>` before applying.

<details>
<summary>Click to expand full manifest</summary>

```json
{
  "apiVersion": "apps/v1",
  "kind": "Deployment",
  "metadata": {
    "name": "<DEPLOYMENT_NAME>",
    "labels": {
      "app.kubernetes.io/name": "signum-agent",
      "app.kubernetes.io/instance": "<INSTANCE_NAME>",
      "app.kubernetes.io/version": "<AGENT_VERSION>",
      "app.kubernetes.io/part-of": "signum",
      "deployment-name": "<DEPLOYMENT_NAME>"
    }
  },
  "spec": {
    "replicas": 1,
    "selector": {
      "matchLabels": {
        "app.kubernetes.io/instance": "<INSTANCE_NAME>",
        "deployment-name": "<DEPLOYMENT_NAME>"
      }
    },
    "template": {
      "metadata": {
        "labels": {
          "app.kubernetes.io/name": "signum-agent",
          "app.kubernetes.io/instance": "<INSTANCE_NAME>",
          "app.kubernetes.io/version": "<AGENT_VERSION>",
          "app.kubernetes.io/part-of": "signum",
          "deployment-name": "<DEPLOYMENT_NAME>"
        }
      },
      "spec": {
        "imagePullSecrets": [
          { "name": "image-creds" }
        ],
        "volumes": [
          {
            "name": "ca-cert",
            "configMap": { "name": "my-custom-ca-cert" }
          },
          { "name": "ca-trust-anchors",   "emptyDir": {} },
          { "name": "ca-trust-extracted", "emptyDir": {} }
        ],
        "initContainers": [
          {
            "name": "update-ca-trust",
            "image": "<IMAGE_HOST>/<IMAGE_PATH>:<AGENT_VERSION>",
            "securityContext": { "runAsUser": 0 },
            "command": [
              "/bin/sh",
              "-c",
              "cp /ca-cert/*.crt /etc/pki/ca-trust/source/anchors/; mkdir /etc/pki/ca-trust/extracted/{edk2,java,openssl,pem}; update-ca-trust extract;"
            ],
            "volumeMounts": [
              { "name": "ca-cert",            "mountPath": "/ca-cert", "readOnly": true },
              { "name": "ca-trust-anchors",   "mountPath": "/etc/pki/ca-trust/source/anchors/" },
              { "name": "ca-trust-extracted", "mountPath": "/etc/pki/ca-trust/extracted/" }
            ],
            "resources": {
              "limits": { "cpu": "50m", "memory": "128Mi" }
            }
          }
        ],
        "containers": [
          {
            "name": "signum-agent",
            "image": "<IMAGE_HOST>/<IMAGE_PATH>:<AGENT_VERSION>",
            "imagePullPolicy": "IfNotPresent",
            "securityContext": { "runAsUser": 10001 },
            "env": [
              { "name": "SIGNUM_HOSTNAME", "value": "<SIGNUM_HOSTNAME>" },
              {
                "name": "SIGNUM_USERNAME",
                "valueFrom": {
                  "secretKeyRef": { "name": "<DEPLOYMENT_NAME>-signum-creds", "key": "username" }
                }
              },
              {
                "name": "SIGNUM_PASSWORD",
                "valueFrom": {
                  "secretKeyRef": { "name": "<DEPLOYMENT_NAME>-signum-creds", "key": "password" }
                }
              },
              { "name": "SIGNUM_LOGLEVEL", "value": "HIGH" },
              { "name": "SIGNUM_LOGTYPE",  "value": "FILE" }
            ],
            "volumeMounts": [
              { "name": "ca-trust-anchors",   "mountPath": "/etc/pki/ca-trust/source/anchors/" },
              { "name": "ca-trust-extracted", "mountPath": "/etc/pki/ca-trust/extracted/" }
            ],
            "resources": {
              "requests": { "cpu": "100m", "memory": "128Mi" },
              "limits":   { "cpu": "250m", "memory": "256Mi" }
            }
          }
        ]
      }
    }
  }
}
```

</details>

If you don't need a custom CA, drop the `ca-cert` / `ca-trust-anchors` / `ca-trust-extracted` volumes, their mounts, and the `update-ca-trust` init container — `securityContext.runAsUser: 10001` on the app container is still required.

## Modifying the Base Image
Add the PKCS#11-based signing tools your team would like to use.
See [dockerfile-examples](/dockerfile-examples/) for some examples. In production you should verify the sources of external repositories.

> **Note (4.80.1):** `pkcs11-tool` (from OpenSC) is no longer preinstalled in the base image. If you need it to inspect or debug the PKCS#11 token from inside the container, layer the [dockerfile-opensc](/dockerfile-examples/dockerfile-opensc) example on top of the base image. See [signing-tool-examples/opensc.md](/signing-tool-examples/opensc.md) for usage.

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

### Example: Adding OpenSC (pkcs11-tool) to the Base Image

As of 4.80.1 the base image no longer bundles `pkcs11-tool`. Build the `dockerfile-opensc` example to layer it on:

```bash
docker buildx build -f dockerfile-examples/dockerfile-opensc -t signum-container-agent:opensc .
```

Run the container and exec into it:

```bash
docker run --name signum-agent -d \
  -e "SIGNUM_HOSTNAME=A URL" \
  -e "SIGNUM_USERNAME=myuser@somedomain" \
  -e "SIGNUM_PASSWORD=$mycreds" \
  -e "SIGNUM_LOGLEVEL=HIGH" \
  -e "SIGNUM_LOGTYPE=FILE" \
  signum-container-agent:opensc

docker exec -it signum-agent /bin/bash
```

List slots and objects on the Signum token:

```sh
pkcs11-tool --module /usr/lib/libsignumpkcs11.so --list-slots
pkcs11-tool --module /usr/lib/libsignumpkcs11.so --list-objects
```

See [signing-tool-examples/opensc.md](/signing-tool-examples/opensc.md) for the full usage example.

---

## Troubleshooting

### Agent fails to start / cannot connect to Signum server
- Verify `SIGNUM_HOSTNAME` is reachable from within the container: `curl -v $SIGNUM_HOSTNAME`
- Check that `SIGNUM_USERNAME` and `SIGNUM_PASSWORD` are correct
- Set `SIGNUM_LOGLEVEL=HIGH` to get detailed logs
- If the Signum server uses a certificate chained to an internal/private CA, verify the container trusts the issuing CA. From inside the container:
  - `curl -w '%{ssl_verify_result}\n' -s -o /dev/null $SIGNUM_HOSTNAME` — `0` means verification succeeded, any non-zero value indicates a TLS trust failure
  - `curl -vI $SIGNUM_HOSTNAME` — prints the server certificate chain and verification details
  - On Kubernetes, the CA must be added to the trust store via an init container that runs `update-ca-trust extract` as root. See the [Deploying on Kubernetes](#deploying-on-kubernetes) example.

### `signum-util lc` returns no certificates
- Confirm the agent has successfully authenticated (check logs with `SIGNUM_LOGLEVEL=HIGH`)
- Verify the user account has certificates assigned to it in the Signum server

### PKCS#11 library not found
- Confirm `/usr/lib/libsignumpkcs11.so` exists inside the container: `ls -la /usr/lib/libsignumpkcs11.so`
- Confirm `/etc/keyfactor/signumpkcs11.cfg` exists and contains the correct library path
- Confirm `/usr/share/p11-kit/modules/signum.module` exists and contains the correct library path

### `pkcs11-tool: command not found`
- As of 4.80.1 OpenSC is no longer bundled in the base image. Build the [dockerfile-opensc](/dockerfile-examples/dockerfile-opensc) example and use that image, or install `opensc` in your own derived Dockerfile.

### `keytool` returns no entries
- The agent must be running and authenticated before the PKCS#11 keystore is populated
- Wait a few seconds after container start for the agent to connect, then retry

### Permission denied errors
- The container runs as user `10001` (non-root). Ensure any host-mounted volumes grant read/write access to this user: `chown -R 10001 ./filestosign`
  
### user doesn't have configuration file or credentials are not valid
- The container is started and configured for the user `10001` (non-root). Ensure any time you are running a command with `docker exec $containerName` you are not overriding the default user for the container.
