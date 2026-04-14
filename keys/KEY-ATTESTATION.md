# Key Attestation

## Key Attester Root Private Key (PEM)

```
openssl genpkey -algorithm EC -pkeyopt ec_paramgen_curve:P-256 -out key-attester-root-private.pem
```

## Key Attester Root Private Key (JWK)

```
eckles key-attester-root-private.pem > key-attester-root-private.jwk
```

## Key Attester Root Public Key (PEM)

```
openssl pkey -in key-attester-root-private.pem -pubout -out key-attester-root-public.pem
```

## Key Attester Root Certificate (PEM)

```
openssl req -x509 \
  -key key-attester-root-private.pem \
  -out key-attester-root-certificate.pem \
  -days 1000000 \
  -subj "/CN=Key Attester Root" \
  -addext "basicConstraints=critical,CA:TRUE" \
  -addext "keyUsage=critical,keyCertSign,cRLSign"
```

## Key Attester Private Key (PEM)

```
openssl genpkey -algorithm EC -pkeyopt ec_paramgen_curve:P-256 -out key-attester-private.pem
```

## Key Attester Private Key (JWK)

```
eckles key-attester-private.pem > key-attester-private.jwk
```

## Key Attester Public Key (PEM)

```
openssl pkey -in key-attester-private.pem -pubout -out key-attester-public.pem
```

## Key Attester Public Key (JWK)

```
eckles key-attester-public.pem > key-attester-public.jwk
```

## Key Attester CSR (PEM)

```
openssl req -new \
  -key key-attester-private.pem \
  -out key-attester.csr \
  -subj "/CN=Key Attester"
```

## Key Attester Certificate (PEM)

```
openssl x509 -req \
  -in key-attester.csr \
  -CA key-attester-root-certificate.pem \
  -CAkey key-attester-root-private.pem \
  -CAcreateserial \
  -out key-attester-certificate.pem \
  -days 1000000 \
  -extfile <(printf "basicConstraints=critical,CA:FALSE\nkeyUsage=critical,digitalSignature")
```

## Key Attestation JWT

```
NONCE=`curl http://localhost:8080/api/nonce -X POST | jq -r .c_nonce`
```

```
KEY_ATTESTATION=`./generate-key-attestation \
  --attester-key=keys/key-attester-private.jwk \
  --attested-key=client.jwk \
  --nonce=$NONCE \
  --x5c=keys/key-attester-certificate.pem`
```

## JWT Key Proof

```
JWT_KEY_PROOF=`./generate-key-proof \
  --client-id=trial_client \
  --issuer=https://trial.authlete.net \
  --key=client.jwk \
  --nonce=$NONCE \
  --key-attestation=$KEY_ATTESTATION`
```

## Credential Request with JWT Key Proof

```
DPOP_PROOF=`./generate-dpop-proof \
  -m POST \
  -u https://trial.authlete.net/api/credential \
  -k client.jwk \
  -a $ACCESS_TOKEN`
```

```
curl \
  --oauth2-bearer $ACCESS_TOKEN \
  -H "DPoP: $DPOP_PROOF" \
  http://localhost:8080/api/credential --json '{
  "credential_configuration_id": "DigitalCredential",
  "proofs": {
    "jwt": ["'$JWT_KEY_PROOF'"]
  }
}'
```

## Credential Request with Attestation Key Proof

```
curl \
  --oauth2-bearer $ACCESS_TOKEN \
  -H "DPoP: $DPOP_PROOF" \
  http://localhost:8080/api/credential --json '{
  "credential_configuration_id": "DigitalCredential",
  "proofs": {
    "attestation": ["'$KEY_ATTESTATION'"]
  }
}'
```

## Service Settings

```json
{
  "keyAttesterRoots": [
    "-----BEGIN CERTIFICATE-----\nMIIBoDCCAUWgAwIBAgIUWRZNOcKwPKKFYiMHguS6ZS81IPUwCgYIKoZIzj0EAwIw\nHDEaMBgGA1UEAwwRS2V5IEF0dGVzdGVyIFJvb3QwIBcNMjYwNDE0MTkwNzUxWhgP\nNDc2NDAzMTExOTA3NTFaMBwxGjAYBgNVBAMMEUtleSBBdHRlc3RlciBSb290MFkw\nEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAE7RNVRube8DDaZ1zYjBe3vPQFB8WVVqJ/\nAAIxUd2HdgLd+7x6EU1T7aHQ5r/ARQpLeYsqiou02jwQoDW+rtNozaNjMGEwHQYD\nVR0OBBYEFEKPG7DDAb8ML3IjnInTugPThi8rMB8GA1UdIwQYMBaAFEKPG7DDAb8M\nL3IjnInTugPThi8rMA8GA1UdEwEB/wQFMAMBAf8wDgYDVR0PAQH/BAQDAgEGMAoG\nCCqGSM49BAMCA0kAMEYCIQCtbIlRnFUs7QSuQZkv6NCxiK0Ui8V/P+gcq/70nWBk\nyQIhAOowEiHzVvY1iTGYJ2at2Z1UigC3rq2q/E2A70AS8gc5\n-----END CERTIFICATE-----"
  ],
  "keyAttesterRootsEnabled": true
}
```
