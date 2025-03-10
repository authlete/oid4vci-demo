# Memo about VC Issuer Key Examples

| file                        | description |
|:----------------------------|:------------|
| `vc-issuer_private-key.pem` | A private key in PEM format |
| `vc-issuer_private-key.jwk` | A private key in JWK format |
| `vc-issuer_public-key.pem`  | The paired public key in PEM format |
| `vc-issuer_public-key.jwk`  | The paired public key in JWK format |
| `vc-issuer_certificate.pem` | A self-signed certificate for the public key in PEM format |
| `vc-issuer_jwk-set.json`    | A JWK Set containing the private key |

## Create a private key in PEM format

```
openssl genpkey -algorithm EC -pkeyopt ec_paramgen_curve:P-256 > vc-issuer_private.pem
```

## Create a public key in PEM format

```
openssl pkey -pubout -in vc-issuer_private.pem > vc-issuer_public.pem
```

## Create a self-signed certificate in PEM format

```
openssl req -x509 -days 100000 -key vc-issuer_private-key.pem -subj "/C=JP/ST=Tokyo/L=Chiyoda/O=Authlete, Inc./CN=trial.authlete.net" -addext subjectAltName=URI:https://trial.authlete.net > vc-issuer_certificate.pem
```

## Create a private key in JWK format

```
npm install -g eckles
eckles vc-issuer_private-key.pem > vc-issuer_private-key.jwk
```

## Create a JWK Set manually

- The JWK Set must be a JSON object.
- The JSON object must contain a `keys` property.
- The value of the `keys` property must be a JSON array.
- Each entry in the `keys` array must be a JWK ([RFC 7517][RFC_7517]).
- The `alg` property of each JWK should be explicitly set (e.g., `"alg":"ES256"`). Otherwise, Authlete may report an error, saying _"The signing key does not have an algorithm."_
- The `kid` property of each JWK should be a JWK Thumbprint ([RFC 7638][RFC_7638]).
- It's better for each JWK to have the `x5c` property if the certificate for the JWK is available.
- A JWK Set containing at least one private key must be registered into Authlete as the JWK Set for Verifiable Credentials. Authlete uses the private key to sign verifiable credentials.

The value for the first element of the `x5c` array can be generated by the following command line.

```
sed /-/d vc-issuer_certificate.pem | tr -d \\n
```

[RFC_7517]: https://www.rfc-editor.org/rfc/rfc7517.html
[RFC_7638]: https://www.rfc-editor.org/rfc/rfc7638.html
