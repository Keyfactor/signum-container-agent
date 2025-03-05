# Signing with XMLSecTool
Instructions for using the 3rd party xmlsectool script for generating signatures for XML files.

Listing the Key Objects.
```sh
keytool -list -storetype PKCS11 -providerClass sun.security.pkcs11.SunPKCS11 -providerArg keyfactorpkcs11.cfg -storepass NONE
```
```sh
Keystore type: PKCS11
Keystore provider: SunPKCS11-KeyfactorPKCS11

Your keystore contains 1 entry

3AB5BFB91DFBB46CF765D5BEE51429618C4857DD - Certificate, PrivateKeyEntry,
Certificate fingerprint (SHA-256): 97:58:8B:1B:C4:D5:19:3C:C6:5F:3F:4A:73:11:53:17:98:D4:A7:E9:FD:A3:3D:88:B0:9F:09:EB:77:D9:23:F0
```

```sh
Using slot 0 with a present token (0x0)
Certificate Object; type = X.509 cert
  label:      50D63698EF043051EFB0B7E5280EDACF35A09B29 - Certificate
  subject:    DN: CN=Signum-RSA-2048
  ID:         50d63698ef043051efb0b7e5280edacf35a09b29
  Unique ID:
 ```

## Signing

```sh
./xmlsectool.sh --sign --pkcs11Config /etc/keyfactor/keyfactorpkcs11.cfg --keyAlias "3AB5BFB91DFBB46CF765D5BEE51429618C4857DD - Certificate" --keyPassword NONE --inFile sample.xml --outFile sample.xml.signed
```

## Verification

```
./xmlsectool.sh --verifySignature --pkcs11Config /etc/keyfactor/keyfactorpkcs11.cfg --keyAlias "3AB5BFB91DFBB46CF765D5BEE51429618C4857DD - Certificate" --keyPassword NONE --inFile sample.xml.signed 
```