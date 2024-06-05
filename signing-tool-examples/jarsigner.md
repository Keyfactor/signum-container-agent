# Signing with Jarsigner
An example of using Jarsigner to sign a Jar file. 

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
jarsigner -storetype PKCS11 -providerClass sun.security.pkcs11.SunPKCS11 -providerArg keyfactorpkcs11.cfg -storepass NONE -tsa http://bensignserver.com/signserver/process?workerId=4 /mnt/filestosign/HelloWorld.jar "3AB5BFB91DFBB46CF765D5BEE51429618C4857DD - Certificate"
```

Verifying the Signature.
```sh
jarsigner -verify -verbose -keystore /mnt/filestosign/BenDemoRootG2-chain.jks /mnt/filestosign/HelloWorld.jar
```

```

s        183 Fri May 24 20:50:32 UTC 2024 META-INF/MANIFEST.MF
         335 Fri May 24 20:50:32 UTC 2024 META-INF/3AB5BFB9.SF
        5914 Fri May 24 20:50:32 UTC 2024 META-INF/3AB5BFB9.RSA
           0 Thu Oct 19 12:47:52 UTC 2023 META-INF/
           0 Thu Oct 19 12:47:52 UTC 2023 com/
           0 Thu Oct 19 12:47:52 UTC 2023 com/example/
           0 Thu Oct 19 12:47:52 UTC 2023 com/example/helloworld/
sm       581 Thu Oct 19 12:47:52 UTC 2023 com/example/helloworld/HelloWorld.class

  s = signature was verified
  m = entry is listed in manifest
  k = at least one certificate was found in keystore

- Signed by "CN=Signum-RSA-4096"
    Digest algorithm: SHA-256
    Signature algorithm: SHA384withRSA, 4096-bit key
  Timestamped by "CN=BenTSACert" on Fri May 24 20:50:33 UTC 2024
    Timestamp digest algorithm: SHA-256
    Timestamp signature algorithm: SHA256withRSA, 2048-bit key

jar verified.

Warning:
This jar contains signed entries that are not signed by alias in this keystore.

Re-run with the -verbose and -certs options for more details.

The signer certificate will expire on 2029-04-23.
The timestamp will expire on 2034-04-23.
```