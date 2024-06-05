# About the Signum Container Agent
The Signum Container Agent is a base image that runs the Signum Agent service and it can be modified as shown in the examples with additional signing tools for handling a variety of different scenarios.

Usage examples of several popular signing tools can be found in [signing-tool-examples](/signing-tool-examples/).

---

## Support for Signum Container Agent with Signing Tools
---
Signum Container Agent with Signing Tools is open source and supported on best effort level for this set of examples.  This means customers can report Bugs, Feature Requests, Documentation amendment or questions as well as requests for customer information required for setup that needs Keyfactor access to obtain. Such requests do not follow normal SLA commitments for response or resolution. If you have a support issue, please open a support ticket via the Keyfactor Support Portal at https://support.keyfactor.com/

To report a problem or suggest a new feature, use the **[Issues](../../issues)** tab. If you want to contribute actual bug fixes or proposed enhancements, use the **[Pull requests](../../pulls)** tab.

---

## Running the Signum Container Agent Base Image
```sh
docker run --name signum-agent -e "SIGNUM_HOSTNAME=A URL" -e "SIGNUM_CLIENTID=TheClientID" -e "SIGNUM_USERNAME=myuser@somedomain" -e "SIGNUM_PASSWORD=$mycreds" -e "SIGNUM_LOGLEVEL=HIGH" -e "SIGNUM_LOGTYPE=FILE" signum-agent-3.80.4-2024052302
```
```sh
docker exec -it signum-agent /bin/bash
```
```sh
pkcs11-tool --module /usr/lib/libkeyfactorpkcs11.so --list-objects --type cert
```
```
Using slot 0 with a present token (0x0)
Certificate Object; type = X.509 cert
  label:      50D63698EF043051EFB0B7E5280EDACF35A09B29 - Certificate
  subject:    DN: CN=Signum-RSA-2048
  ID:         50d63698ef043051efb0b7e5280edacf35a09b29
  Unique ID:
Certificate Object; type = X.509 cert
  label:      170570A1D56FBB5A4CC780B69ACAEF94010D5DAA - Certificate
  subject:    DN: CN=Signum-RSA-3072
  ID:         170570a1d56fbb5a4cc780b69acaef94010d5daa
  Unique ID:
Certificate Object; type = X.509 cert
  label:      3AB5BFB91DFBB46CF765D5BEE51429618C4857DD - Certificate
  subject:    DN: CN=Signum-RSA-4096
  ID:         3ab5bfb91dfbb46cf765d5bee51429618c4857dd
  Unique ID:
```

## Modifying the Base Image
Add the pkcs11 based signing tools your team would like to use
see, [dockerfile-examples](/dockerfile-examples/) for some examples. In production you should verify the sources of external repositories. 

An Example adding Jsign to the base Signum Agent Image and then launching a container. 
```bash
docker buildx build -f dockerfile-jsign -t signum-container-agent:jsign .
```

```bash
docker run --name signum-agent -d -v $PWD/filestosign/:/mnt/filestosign -e "SIGNUM_HOSTNAME=A URL" -e "SIGNUM_CLIENTID=TheClientID" -e "SIGNUM_USERNAME=myuser@somedomain" -e "SIGNUM_PASSWORD=$mycreds" -e "SIGNUM_LOGLEVEL=HIGH" -e "SIGNUM_LOGTYPE=FILE" signum-container-agent:jsign
```

```bash
docker exec -it signum-agent /bin/bash
```

Can run keytool and Jsign to list key objects and sign.
```sh
keytool -list -storetype PKCS11 -providerClass sun.security.pkcs11.SunPKCS11 -providerArg keyfactorpkcs11.cfg -storepass NONE
```
```
Keystore type: PKCS11
Keystore provider: SunPKCS11-KeyfactorPKCS11

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

```
java -jar jsign-6.0.jar --keystore /etc/keyfactor/keyfactorpkcs11.cfg --storetype PKCS11 --alias "3AB5BFB91DFBB46CF765D5BEE51429618C4857DD - Certificate" /mnt/filestosign/example-script.ps1
```

