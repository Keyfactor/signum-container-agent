# Signing with OpenSSL

Listing the Key Objects.
```sh
pkcs11-tool --module /usr/lib/libkeyfactorpkcs11.so --list-objects --type cert
```

```sh
Using slot 0 with a present token (0x0)
Certificate Object; type = X.509 cert
  label:      50D63698EF043051EFB0B7E5280EDACF35A09B29 - Certificate
  subject:    DN: CN=Signum-RSA-2048
  ID:         50d63698ef043051efb0b7e5280edacf35a09b29
  Unique ID:
 ```
## Using Dgst 

```sh
echo "some stuff to sign" > test.txt
```

 ```sh
 openssl dgst -engine pkcs11 -keyform engine -sha256 -sign 50D63698EF043051EFB0B7E5280EDACF35A09B29 test.txt > sign.bin
  ```
```sh
Engine "pkcs11" set.
```

```sh
 openssl dgst -engine pkcs11 -keyform engine -sha256 -verify 50D63698EF043051EFB0B7E5280EDACF35A09B29 -signature sign.bin < test.txt
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