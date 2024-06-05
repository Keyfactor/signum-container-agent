# Signing with Cosign

As an proof of concept you can run a local registry to try things out or use an existing OCI compliant registry. Here is an example using [Docker Registry](https://hub.docker.com/_/registry).

You will need to use Cosign built with pkcs11 (cosign-linux-pivkey-pkcs11key)
support https://github.com/sigstore/cosign/releases. The certificate being used also needs to include an email address as a SAN value which EJBCA can issue from an [end entity profile](https://doc.primekey.com/ejbca/ejbca-operations/ejbca-ca-concept-guide/end-entities-overview/end-entity-profiles-overview/end-entity-profiles-fields#EndEntityProfilesFields-RFC822Name(e-mailaddress)). 

Start a local Docker Registry
```sh
docker run -d -p 5000:5000 --restart always --name registry registry:2
```

With the registry running pull an image to sign.
```sh
docker pull keyfactor/ejbca-ce
```

Push that image to the local registry
```sh
docker tag keyfactor/ejbca-ce localhost:5000/ejbca-ce:latest && docker push localhost:5000/ejbca-ce
```
Can use Registry API to view.

```sh
curl -X GET localhost:5000/v2/_catalog | jq .
```

```json
{
  "repositories": [
    "ejbca-ce"
  ]
}
```
If running a local registry on the default Docker network we will need the ip to reference later.

```sh
docker ps -q | xargs docker inspect --format '{{ .Name }} - {{ .NetworkSettings.IPAddress }}'
```

Exec into the container and use Cosign to list the Key Objects. 
```sh
 cosign pkcs11-tool list-keys-uris --module-path /usr/lib/libkeyfactorpkcs11.so --slot-id 1
```

```sh
Object 0
	Label: 50D63698EF043051EFB0B7E5280EDACF35A09B29 - Private key
	ID: 50d63698ef043051efb0b7e5280edacf35a09b29
	URI: pkcs11:token=Keyfactor%20for%20Linux%00%20%20%20%20%20%20%20%20%20%20%20%00;slot-id=1;id=%50%d6%36%98%ef%04%30%51%ef%b0%b7%e5%28%0e%da%cf%35%a0%9b%29;object=50D63698EF043051EFB0B7E5280EDACF35A09B29%20-%20Private%20key?module-path=/usr/lib/libkeyfactorpkcs11.so
```

Setting the URI to a variable to make things easier to read later.
```sh
uri="pkcs11:token=Keyfactor%20for%20Linux%00%20%20%20%20%20%20%20%20%20%20%20%00;slot-id=1;id=%50%d6%36%98%ef%04%30%51%ef%b0%b7%e5%28%0e%da%cf%35%a0%9b%29;object=50D63698EF043051EFB0B7E5280EDACF35A09B29%20-%20Private%20key?module-path=/usr/lib/libkeyfactorpkcs11.so"
```

Note the image tags before signing.
```sh
curl -X GET http://localhost:5000/v2/ejbca-ce/tags/list | jq .
```
```json
{
  "name": "ejbca-ce",
  "tags": [
    "latest"
  ]
}
```

Getting the image digest. 
```sh
docker inspect --format='{{index .RepoDigests 0}}' localhost:5000/ejbca-ce:latest
```

Sign the Image.
```sh
cosign sign --key $uri --tlog-upload=false 172.17.0.2:5000/ejbca-ce@sha256:5a2dd15b2d85a6094ea3cd65444e2b62fcc6afd8a18e9f299f22d71c021b92f5
```

Notice the signature tag that has been added. 
```json
{
  "name": "ejbca-ce",
  "tags": [
    "sha256-5a2dd15b2d85a6094ea3cd65444e2b62fcc6afd8a18e9f299f22d71c021b92f5.sig",
    "latest"
  ]
}
```

Verify the Signature.
```sh
cosign verify --key $uri --insecure-ignore-tlog 172.17.0.2:5000/ejbca-ce@sha256:5a2dd15b2d85a6094ea3cd65444e2b62fcc6afd8a18e9f299f22d71c021b92f5 | jq .
```
```json
[
  {
    "critical": {
      "identity": {
        "docker-reference": "172.17.0.2:5000/ejbca-ce"
      },
      "image": {
        "docker-manifest-digest": "sha256:5a2dd15b2d85a6094ea3cd65444e2b62fcc6afd8a18e9f299f22d71c021b92f5"
      },
      "type": "cosign container image signature"
    },
    "optional": {
      "Subject": "ben@signum.com"
    }
  }
]
```