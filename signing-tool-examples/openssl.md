# Signing with OpenSSL

Fully up-to date documentation can be found in the [oficial documentation](https://docs.keyfactor.com/Signum-SaaS/latest/using-signum-with-openssl/)

The images are built with OpenSSL 4, which no longer supports engines. The [dockerfile-openssl](../dockerfile-examples/dockerfile-openssl) example installs the [pkcs11-provider](https://github.com/latchset/pkcs11-provider) and sets `OPENSSL_CONF`, so the Signum PKCS#11 library is loaded automatically and the `-engine pkcs11 -keyform engine` flags are no longer needed.

> **Note:** signing with ECDSA keys through OpenSSL is not supported on Signum Agent Container versions before 5.0.0.

Obtain the pkcs11 private and public URLs for your certificate by following [obtain-pkcs11-url.md](obtain-pkcs11-url.md), then set the `privateUrl` and `publicUrl` variables before continuing.


## Using Dgst

```sh
echo "some stuff to sign" > test.txt
```

 ```sh
 openssl dgst -sha256 -sign $privateURL test.txt > sign.bin
  ```
```sh
 openssl dgst -sha256 -verify $publicURL -signature sign.bin < test.txt
 ```

 ```sh
Verified OK
```

 ## Using CMS

```sh
echo "some stuff to sign" > example.txt
```

Download your signer certificate from Signum
```sh
openssl cms -sign -binary -in example.txt -signer /mnt/certs/signum-cert.pem -inkey 50d63698ef043051efb0b7e5280edacf35a09b29 -outform PEM -out signature.p7m
```
Verify with your CA
```sh
openssl cms -verify -binary -in signature.p7m -inform PEM -content example.txt  -CAfile /mnt/certs/BenDemoRootG2-chain.pem -purpose any
```

```sh
some stuff to sign
CMS Verification successful
```
