# Client Attestation

## Client Attester Root Private Key (PEM)

```
openssl genpkey -algorithm EC -pkeyopt ec_paramgen_curve:P-256 -out client-attester-root-private.pem
```

## Client Attester Root Private Key (JWK)

```
eckles client-attester-root-private.pem > client-attester-root-private.jwk
```

## Client Attester Root Public Key (PEM)

```
openssl pkey -in client-attester-root-private.pem -pubout -out client-attester-root-public.pem
```

## Client Attester Root Certificate (PEM)

```
openssl req -x509 \
  -key client-attester-root-private.pem \
  -out client-attester-root-certificate.pem \
  -days 1000000 \
  -subj "/CN=Client Attester Root" \
  -addext "basicConstraints=critical,CA:TRUE" \
  -addext "keyUsage=critical,keyCertSign,cRLSign"
```

## Client Attester Private Key (PEM)

```
openssl genpkey -algorithm EC -pkeyopt ec_paramgen_curve:P-256 -out client-attester-private.pem
```

## Client Attester Private Key (JWK)

```
eckles client-attester-private.pem > client-attester-private.jwk
```

## Client Attester Public Key (PEM)

```
openssl pkey -in client-attester-private.pem -pubout -out client-attester-public.pem
```

## Client Attester Public Key (JWK)

```
eckles client-attester-public.pem > client-attester-public.jwk
```

## Client Attester CSR (PEM)

```
openssl req -new \
  -key client-attester-private.pem \
  -out client-attester.csr \
  -subj "/CN=Client Attester"
```

## Client Attester Certificate (PEM)

```
openssl x509 -req \
  -in client-attester.csr \
  -CA client-attester-root-certificate.pem \
  -CAkey client-attester-root-private.pem \
  -CAcreateserial \
  -out client-attester-certificate.pem \
  -days 1000000 \
  -extfile <(printf "basicConstraints=critical,CA:FALSE\nkeyUsage=critical,digitalSignature")
```

## Client Attestation JWT

```
CLIENT_ATTESTATION=`./generate-client-attestation \
  --attester-key=keys/client-attester-private.jwk \
  --client-id=trial_client \
  --client-key=client.jwk \
  --x5c=keys/client-attester-certificate.pem`
```

## Client Attestation PoP JWT

```
CLIENT_ATTESTATION_POP=`./generate-client-attestation-pop \
  --as-id=https://trial.authlete.net \
  --client-key=client.jwk`
```

## Service Settings

```json
{
  "clientAttesterRoots": [
    "-----BEGIN CERTIFICATE-----\nMIIBpTCCAUugAwIBAgIUUTw+Ep5ZOdplQAjzE/68Z9IUzzgwCgYIKoZIzj0EAwIw\nHzEdMBsGA1UEAwwUQ2xpZW50IEF0dGVzdGVyIFJvb3QwIBcNMjYwNDE0MTU0MzQ1\nWhgPNDc2NDAzMTExNTQzNDVaMB8xHTAbBgNVBAMMFENsaWVudCBBdHRlc3RlciBS\nb290MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAEyRlNlUPzTt6jF5s0ltIjoGHx\nSFQu6GE+gZOMm5sOrNGmAaI6i48CSI6yrXpr/F0KV6t4IH9FcKsZWrpIA7evz6Nj\nMGEwHQYDVR0OBBYEFDmGsWAPAWFDwW4IlARNSXJ3amPSMB8GA1UdIwQYMBaAFDmG\nsWAPAWFDwW4IlARNSXJ3amPSMA8GA1UdEwEB/wQFMAMBAf8wDgYDVR0PAQH/BAQD\nAgEGMAoGCCqGSM49BAMCA0gAMEUCIQCXxFHg356YWT2gEIdU0Vdm2znUMRwm7Wgb\nJf9GbLDvggIgK98PHmvP8WRqPfQr1cNpSSBoogDhZhLcelvm/L0coLA=\n-----END CERTIFICATE-----"
  ],
  "clientAttesterRootsEnabled": true
}
```
