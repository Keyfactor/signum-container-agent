# Signing with OpenSSL

Fully up-to date documentation can be found in the [oficial documentation](https://docs.keyfactor.com/Signum-SaaS/latest/using-signum-with-openssl/)

Obtain the pkcs11 private and public URLs for your certificate by following [obtain-pkcs11-url.md](obtain-pkcs11-url.md), then set the `privateUrl` and `publicUrl` variables before continuing.


## Using Dgst

```sh
echo "some stuff to sign" > test.txt
```

 ```sh
 openssl dgst -engine pkcs11 -keyform engine -sha256 -sign $privateURL test.txt > sign.bin
  ```
```sh
Engine "pkcs11" set.
```

```sh
 openssl dgst -engine pkcs11 -keyform engine -sha256 -verify $publicURL -signature sign.bin < test.txt
 ```

 ```sh
Engine "pkcs11" set.
Verified OK
```

 ## Using CMS

```sh
echo "some stuff to sign" > example.txt
```

Download your signer certificate from Signum
```sh
openssl cms -sign -binary -engine pkcs11 -keyform engine -in example.txt -signer /mnt/certs/signum-cert.pem -inkey 50d63698ef043051efb0b7e5280edacf35a09b29 -outform PEM -out signature.p7m
```
```sh
Engine "pkcs11" set.
```

Verify with your CA
```sh
openssl cms -verify -binary -in signature.p7m -inform PEM -content example.txt  -CAfile /mnt/certs/BenDemoRootG2-chain.pem -purpose any
```

```sh
some stuff to sign
CMS Verification successful
```
