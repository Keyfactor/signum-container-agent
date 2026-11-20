# Using OpenSC (pkcs11-tool)

`pkcs11-tool` (from the [OpenSC](https://github.com/OpenSC/OpenSC) project) is useful for inspecting the Signum PKCS#11 token from inside the agent container — listing slots, objects, and mechanisms, and verifying that the agent is authenticated and populating the keystore correctly.

Starting with signum-agent `4.80.1`, `pkcs11-tool` is no longer bundled in the base image. Build the `dockerfile-opensc` example to layer it on top.

## Build the extended image

```bash
docker buildx build -f dockerfile-examples/dockerfile-opensc -t signum-container-agent:opensc .
```

## Run the container

```bash
docker run --name signum-agent -d \
  -e "SIGNUM_HOSTNAME=A URL" \
  -e "SIGNUM_USERNAME=myuser@somedomain" \
  -e "SIGNUM_PASSWORD=$mycreds" \
  -e "SIGNUM_LOGLEVEL=HIGH" \
  -e "SIGNUM_LOGTYPE=FILE" \
  signum-container-agent:opensc
```

Open a shell in the running container:

```bash
docker exec -it signum-agent /bin/bash
```

## List token slots

```sh
pkcs11-tool --module /usr/lib/libsignumpkcs11.so --list-slots
```

```
Available slots:
Slot 0 (0x1): Signum for Linux
  token label        : Signum for Linux
  token manufacturer : Keyfactor
  token model        : Linux
  token flags        : login required, rng, token initialized, PIN initialized
  hardware version   : 1.0
  firmware version   : 1.0
  serial num         : 1
  pin min/max        : 4/64
```

## List objects (certificates, public/private keys)

```sh
pkcs11-tool --module /usr/lib/libsignumpkcs11.so --list-objects
```

```
Certificate Object; type = X.509 cert
  label:      3AB5BFB91DFBB46CF765D5BEE51429618C4857DD - Certificate
  subject:    DN: CN=Signum-RSA-4096
  ID:         3ab5bfb91dfbb46cf765d5bee51429618c4857dd
Public Key Object; RSA 4096 bits
  label:      3AB5BFB91DFBB46CF765D5BEE51429618C4857DD - Public key
  ID:         3ab5bfb91dfbb46cf765d5bee51429618c4857dd
  Usage:      verify
Private Key Object; RSA
  label:      3AB5BFB91DFBB46CF765D5BEE51429618C4857DD - Private key
  ID:         3ab5bfb91dfbb46cf765d5bee51429618c4857dd
  Usage:      sign
```

## List supported mechanisms

```sh
pkcs11-tool --module /usr/lib/libsignumpkcs11.so --list-mechanisms
```

## Troubleshooting

- **`pkcs11-tool: command not found`** — the base image no longer ships it. Build the `dockerfile-opensc` extended image.
- **No objects returned** — the agent must be running and authenticated. Wait a few seconds after container start, then retry. Use `SIGNUM_LOGLEVEL=HIGH` to diagnose authentication failures.
- **`CKR_GENERAL_ERROR` or `CKR_TOKEN_NOT_PRESENT`** — confirm `/usr/lib/libsignumpkcs11.so` exists (`ls -la /usr/lib/libsignumpkcs11.so`) and that the agent service is listening on `localhost:51599`.
