# Obtain the PKCS11 URL

List all objects in the Signum token to find the PKCS11 URLs for your key:

```sh
p11tool --list-all "pkcs11:model=Linux;manufacturer=Keyfactor;serial=1;token=Signum%20for%20Linux%00"
```

```sh
Object 0:
        URL: pkcs11:model=Linux;manufacturer=Keyfactor;serial=1;token=Signum%20for%20Linux%00;id=%33%2D%B3%5F%9C%6A%34%D7%80%4D%47%20%8B%E8%BC%0F%02%30%77%A8;object=332DB35F9C6A34D7804D47208BE8BC0F023077A8%20-%20Certificate;type=cert
        Type: X.509 Certificate
        Label: 332DB35F9C6A34D7804D47208BE8BC0F023077A8 - Certificate
        Flags: CKA_PRIVATE; CKA_TRUSTED;
        ID: 33:2d:b3:5f:9c:6a:34:d7:80:4d:47:20:8b:e8:bc:0f:02:30:77:a8

Object 1:
        URL: pkcs11:model=Linux;manufacturer=Keyfactor;serial=1;token=Signum%20for%20Linux%00;id=%33%2D%B3%5F%9C%6A%34%D7%80%4D%47%20%8B%E8%BC%0F%02%30%77%A8;object=332DB35F9C6A34D7804D47208BE8BC0F023077A8%20-%20Public%20key;type=public
        Type: Public key (RSA-4096)
        Label: 332DB35F9C6A34D7804D47208BE8BC0F023077A8 - Public key
        Flags: CKA_WRAP/UNWRAP; CKA_PRIVATE; CKA_TRUSTED;
        ID: 33:2d:b3:5f:9c:6a:34:d7:80:4d:47:20:8b:e8:bc:0f:02:30:77:a8

Object 2:
        URL: pkcs11:model=Linux;manufacturer=Keyfactor;serial=1;token=Signum%20for%20Linux%00;id=%33%2D%B3%5F%9C%6A%34%D7%80%4D%47%20%8B%E8%BC%0F%02%30%77%A8;object=332DB35F9C6A34D7804D47208BE8BC0F023077A8%20-%20Private%20key;type=private
        Type: Private key (RSA-4096)
        Label: 332DB35F9C6A34D7804D47208BE8BC0F023077A8 - Private key
        Flags: CKA_WRAP/UNWRAP; CKA_PRIVATE; CKA_TRUSTED; CKA_EXTRACTABLE; CKA_SENSITIVE;
        ID: 33:2d:b3:5f:9c:6a:34:d7:80:4d:47:20:8b:e8:bc:0f:02:30:77:a8
```

Locate the private and public URL to use to sign and verify:

```sh
privateUrl="pkcs11:model=Linux;manufacturer=Keyfactor;serial=1;token=Signum%20for%20Linux%00;id=%33%2D%B3%5F%9C%6A%34%D7%80%4D%47%20%8B%E8%BC%0F%02%30%77%A8;object=332DB35F9C6A34D7804D47208BE8BC0F023077A8%20-%20Private%20key;type=private"
publicUrl="pkcs11:model=Linux;manufacturer=Keyfactor;serial=1;token=Signum%20for%20Linux%00;id=%33%2D%B3%5F%9C%6A%34%D7%80%4D%47%20%8B%E8%BC%0F%02%30%77%A8;object=332DB35F9C6A34D7804D47208BE8BC0F023077A8%20-%20Public%20key;type=public"
```
