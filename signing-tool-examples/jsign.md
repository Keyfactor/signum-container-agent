# Signing with Jsign
Jsign is a Java implementation of Microsoft's Authenticode check out the project at https://github.com/ebourg/jsign .

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
java -jar jsign-6.0.jar --keystore /etc/keyfactor/keyfactorpkcs11.cfg --storetype PKCS11 --alias "3AB5BFB91DFBB46CF765D5BEE51429618C4857DD - Certificate" /mnt/filestosign/example-script.ps1
```
```sh
Adding Authenticode signature to /mnt/filestosign/example-script.ps1
bash-5.1$ cat /mnt/filestosign/example-script.ps1
some scripting things....

# SIG # Begin signature block
# MIIS2AYJKoZIhvcNAQcCoIISyTCCEsUCAQExDTALBglghkgBZQMEAgEweQYKKwYB
# BAGCNwIBBKBrMGkwNAYKKwYBBAGCNwIBHjAmAgMBAAAEEB/MO2BZSwhOtyTSxil+
.....
# p/iTFXNyrAKiM3wLD1LyJq2J4hcitxIXQw7fRG2ZSQwF57SkdPq7nnkA3L630zTv
# YCOz0F8RUFaiQFn7P43f6cb9g7uaG+FAwm+oEQ==
# SIG # End signature block
```
