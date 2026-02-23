# OID4VCI 1.0 Final Setup

## Service Configuration

- Tokens and Claims
  - Advanced
    - Scope
      - Supported Authorization Details Types
        - Add `openid_credential`
      - Supported Scopes
        - Add `digital_credential`
        - Add `org.iso.18013.5.1.mDL`

- Key Management
  - JWK Set
    - Authorization Server
      - JWK Set Content -> as_jwk-set.json in https://github.com/authlete/oid4vci-demo/
      - JWK Set Endpoint URI -> https://{AUTHORIZATION_SERVER}/api/jwks
    - Verifiable Credentials
      - JWK Set Content -> vc-issuer_jwk-set.json in https://github.com/authlete/oid4vci-demo/
      - JWK Set Endpoint URI -> https://{CREDENTIAL_ISSUER}/api/vci/jwks

- Verifiable Credentials
  - General
    - Verifiable Credentials Feature -> Enable
    - Credential Issuance Version -> 1.0-Final
  - Credential Issuer Metadata
    - Credential Issuer -> https://{CREDENTIAL_ISSUER}
    - Credential Endpoint -> https://{CREDENTIAL_ISSUER}/api/credential
    - Deferred Credential Endpoint -> https://{CREDENTIAL_ISSUER}/api/deferred_credential
    - Nonce Endpoint -> https://{CREDENTIAL_ISSUER}/api/nonce
    - Credential Request Encryption JWK Set -> credential_request_encryption_jwk-set.json in https://github.com/authlete/oid4vci-demo/
    - Supported Credential Configurations -> credential_configurations_supported-final.json in https://github.com/authlete/oid4vci-demo/
    - Display Types -> vc-issuer_display.json in https://github.com/authlete/oid4vci-demo/

## Client Configuration

- Basic Settings
  - General
    - Client ID -> trial_client
    - Client Type -> PUBLIC
- Endpoints
  - Global Settings
    - Redirect URIs
      - Add `https://nextdev-api.authlete.net/api/mock/redirection`
- Tokens and Claims
  - ID Token
    - General
      - ID Token Algorithms
        - Signature Algorithm -> ES256
  - Advanced
    - Scope
      - Authorization Details Types
        - Add `openid_credential`

NOTE: It is mandatory to add `openid_credential` to the Authorization Details Types,
but the other configuration items may be changed as long as you do not use the
examples in this document as they are.

## Authlete Server

Use the new Authlete 3.0 server that includes the commit below.

```
commit a2b9c0baf3f61eba7fee36c4354f80f28c036e2f (HEAD -> 3.0, origin/3.0)
Merge: 83969551b b743f69ce
Author: Takahiko Kawasaki <taka@authlete.com>
Date:   Mon Feb 23 10:51:53 2026 +0000

    Merge branch 'v3.0/feat/oid4vci-proofs-requirement' into '3.0'
    
    feat: OID4VCI / Proofs Requirement
    
    See merge request authlete/server!2353

commit b743f69ce290c01e1f0dea0ca5a795363529bbdf
Author: Takahiko Kawasaki <taka@authlete.com>
Date:   Mon Feb 23 10:51:52 2026 +0000

    feat: OID4VCI / Proofs Requirement
    
    Approved-by: Hideki Ikeda <hide@authlete.com>
    
    Refs:

```

## java-oauth-server

Use the new `java-oauth-server` that includes the commit below.

```
commit 14e41cfdf962b074390172606194150c65ec488d (HEAD -> master, origin/master, origin/HEAD)
Merge: 23cf31d f9009f4
Author: Takahiko Kawasaki <taka@authlete.com>
Date:   Mon Feb 23 17:15:50 2026 +0900

    Merge pull request #82 from authlete/feat/oid4vci_final-order_processor
    
    feat: OrderProcessor Update for OID4VCI 1.0 Final

commit f9009f408c42e2091140f43d9805678cba4dd879 (origin/feat/oid4vci_final-order_processor, feat/oid4vci_final-order_processor)
Author: Takahiko Kawasaki <taka@authlete.com>
Date:   Mon Feb 23 17:14:12 2026 +0900

    feat: OrderProcessor Update for OID4VCI 1.0 Final

```

## Credential Issuer Configuration Request

```
https://{CREDENTIAL_ISSUER}/.well-known/openid-credential-issuer
```

### Credential Issuer Configuration Request Examples

Trial Server (trial.authlete.net)

```
https://trial.authlete.net/.well-known/openid-credential-issuer
```

Local Server (localhost)

```
http://localhost:8080/.well-known/openid-credential-issuer
```

## Authorization Request

```
https://{AUTHORIZATION_SERVER}/api/authorization?client_id={CLIENT_ID}&response_type=code&scope=email
authorization_details=%5B%7B%22type%22%3A%22openid_credential%22%2C%22credential_configuration_id%22%3A%22DigitalCredential%22%7D%2C%7B%22type%22%3A%22openid_credential%22%2C%22credential_configuration_id%22%3A%22org.iso.18013.5.1.mDL%22%7D%5D
```

| Input Field | Value  |
|:------------|:-------|
| Login ID    | `inga` |
| Password    | `inga` |

The value of the `authorization_details` request parameter represents the following JSON:

```json
[
  {
    "type": "openid_credential",
    "credential_configuration_id": "DigitalCredential"
  },
  {
    "type": "openid_credential",
    "credential_configuration_id": "org.iso.18013.5.1.mDL"
  }
]
```

### Authorization Request Examples

Trial Server (trial.authlete.net)

```
https://trial.authlete.net/api/authorization?client_id=trial_client&response_type=code&scope=email&authorization_details=%5B%7B%22type%22%3A%22openid_credential%22%2C%22credential_configuration_id%22%3A%22DigitalCredential%22%7D%2C%7B%22type%22%3A%22openid_credential%22%2C%22credential_configuration_id%22%3A%22org.iso.18013.5.1.mDL%22%7D%5D
```

Local Server (localhost)

```
http://localhost:8080/api/authorization?client_id=trial_client&response_type=code&scope=email&authorization_details=%5B%7B%22type%22%3A%22openid_credential%22%2C%22credential_configuration_id%22%3A%22DigitalCredential%22%7D%2C%7B%22type%22%3A%22openid_credential%22%2C%22credential_configuration_id%22%3A%22org.iso.18013.5.1.mDL%22%7D%5D
```

## Token Request

```
curl https://${AUTHORIZATION_SERVER}/api/token -d client_id=${CLIENT_ID} -d grant_type=authorization_code -d code=${AUTHORIZATION_CODE}
```

### Token Request Examples

Trial Server (trial.authlete.net)

```
curl https://trial.authlete.net/api/token -d client_id=trial_client -d grant_type=authorization_code -d code=${AUTHORIZATION_CODE}
```

Local Server (localhost)

```
curl http://localhost:8080/api/token -d client_id=trial_client -d grant_type=authorization_code -d code=${AUTHORIZATION_CODE}
```

### Token Response Examples

```json
{
  "access_token": "ZpbcEr6-WVhqFb3qdRlw2NuNJczmgLKqbwnVyEEbxwo",
  "token_type": "Bearer",
  "expires_in": 86400,
  "scope": "email",
  "refresh_token": "8vM0kzkLcCOQ9YjrFp0nzJxFK_Il0Bq_JxW-vDRB5oo",
  "authorization_details": [
    {
      "credential_configuration_id": "DigitalCredential",
      "credential_identifiers": [
        "DigitalCredential/Type1",
        "DigitalCredential/Type2"
      ],
      "type": "openid_credential"
    },
    {
      "credential_configuration_id": "org.iso.18013.5.1.mDL",
      "credential_identifiers": [
        "org.iso.18013.5.1.mDL"
      ],
      "type": "openid_credential"
    }
  ]
}
```

## Credential Request

### Credential Request with Credential Configuration ID

Key proof generation using the `generate-key-proof` script in https://github.com/authlete/oid4vci-demo/

```
./generate-key-proof --client-id=${CLIENT_ID} --issuer=https://${CREDENTIAL_ISSUER} --key=${JWK_FILE} --nonce=${NONCE}
```

Credential Request

```
curl --oauth2-bearer ${ACCESS_TOKEN} \
     https://${CREDENTIAL_ISSUER}/api/credential \
     --json '{
  "credential_configuration_id": "'${CREDENTIAL_CONFIGURATION_ID}'",
  "proofs": {
    "jwt": ["'${KEY_PROOF}'"]
  }
}'
```

#### Credential Request with Credential Configuration ID Examples for SD-JWT VC

Trial Server (trial.authlete.net)

```
NONCE=`curl https://trial.authlete.net/api/nonce -X POST | jq -r .c_nonce`
KEY_PROOF=`./generate-key-proof --client-id=trial_client --issuer=https://trial.authlete.net --key=holder.jwk --nonce=${NONCE}`
```

```
curl --oauth2-bearer ${ACCESS_TOKEN} \
     https://trial.authlete.net/api/credential \
     --json '{
  "credential_configuration_id": "DigitalCredential",
  "proofs": {
    "jwt": ["'${KEY_PROOF}'"]
  }
}'
```

Local Server (localhost)

```
NONCE=`curl http://localhost:8080/api/nonce -X POST | jq -r .c_nonce`
KEY_PROOF=`./generate-key-proof --client-id=trial_client --issuer=https://trial.authlete.net --key=holder.jwk --nonce=${NONCE}`
```

```
curl --oauth2-bearer ${ACCESS_TOKEN} \
     http://localhost:8080/api/credential \
     --json '{
  "credential_configuration_id": "DigitalCredential",
  "proofs": {
    "jwt": ["'${KEY_PROOF}'"]
  }
}'
```

#### Credential Response with Credential Configuration ID Examples for SD-JWT VC

```json
{
  "credentials": [
    {
      "credential": "eyJ4NWMiOlsiTUlJQ1NEQ0NBZTZnQXdJQkFnSVVLSlM1R21Ram5mY0JrM1piL2VXK0loblFrOHN3Q2dZSUtvWkl6ajBFQXdJd1pURUxNQWtHQTFVRUJoTUNTbEF4RGpBTUJnTlZCQWdNQlZSdmEzbHZNUkF3RGdZRFZRUUhEQWREYUdsNWIyUmhNUmN3RlFZRFZRUUtEQTVCZFhSb2JHVjBaU3dnU1c1akxqRWJNQmtHQTFVRUF3d1NkSEpwWVd3dVlYVjBhR3hsZEdVdWJtVjBNQ0FYRFRJMU1ESXlOVEExTXpJME9Wb1lEekl5T1RneE1qRXhNRFV6TWpRNVdqQmxNUXN3Q1FZRFZRUUdFd0pLVURFT01Bd0dBMVVFQ0F3RlZHOXJlVzh4RURBT0JnTlZCQWNNQjBOb2FYbHZaR0V4RnpBVkJnTlZCQW9NRGtGMWRHaHNaWFJsTENCSmJtTXVNUnN3R1FZRFZRUUREQkowY21saGJDNWhkWFJvYkdWMFpTNXVaWFF3V1RBVEJnY3Foa2pPUFFJQkJnZ3Foa2pPUFFNQkJ3TkNBQVFPZDZUQ3ducG5tWHFwczhSUUhxK0s5Tm9vRmJyamIxeXlGeEpsSGs3V2ZUVk9yWkR1OU5xK0lPYzl5cm83eStHOXJUZjB6dDhPV0o1aS9XaWpqSUxUbzNvd2VEQWRCZ05WSFE0RUZnUVVQTVlhNmZRSlA2TTZ4NHRZTTFmYmYzL0UweE13SHdZRFZSMGpCQmd3Rm9BVVBNWWE2ZlFKUDZNNng0dFlNMWZiZjMvRTB4TXdEd1lEVlIwVEFRSC9CQVV3QXdFQi96QWxCZ05WSFJFRUhqQWNoaHBvZEhSd2N6b3ZMM1J5YVdGc0xtRjFkR2hzWlhSbExtNWxkREFLQmdncWhrak9QUVFEQWdOSUFEQkZBaUVBeW8zQS9vUVgvTWU4bXlNV0wwMjVjVEJ3L2tlY2VIRUxmU1FIbWxlNVFBRUNJQzVnL3luT210L251a3NmSEFlTUFCZjd0bzUyelRSa1JkdXFRQ1pITzNlWSJdLCJraWQiOiJaWUdJT0hZdUE5SXBVaWpWd1FOdWwzbkU1MzZ4MUpTV0hpT2ZkUzdzYWRnIiwidHlwIjoiZGMrc2Qtand0IiwiYWxnIjoiRVMyNTYifQ.eyJpc3MiOiJodHRwczovL3RyaWFsLmF1dGhsZXRlLm5ldCIsIl9zZCI6WyJEWC1rVUJSeXRLb3phWHdOc2pvQVZEVDBnMEtXZk4xTDE0alFnTjk4dmNvIiwiRnFpVTJiLURMTGE4c2JtUGFVOU9ESmlqaUdmbWZXeWl0eGI0YW14RWpycyIsIkdQYVFndEt2NGRoUW5PV0VFc2dUTGRsal8tLW5qdlR0OUhmUkVtdEN1bjgiLCJVSGVZT3M3cHhMbXRpTWFUVmhza1gwYkJvUGxxMDJhSWM3R3RHamprSnRRIiwiV1h4MTRFdERvbHdObUFVVjBTeXVYM1JfaGZTZEpxUk1fTnI0aUY2NUN3VSIsIllLYmJNVTlKa01VZEdKaldEY1VBOTd5cVpvdDhsSkhmRkQzLTRKR0R0OEkiLCJfZnJwbXV0SHJMdmNLbm9jMHVkdmNrTkV1R1d4YTRRT19RZDR6a1did1djIiwiX3ZibjMwSDN5cGc5WTJTSURyTy10akhXR3dLaWJIenFOQ0ZGdmI0bVVHUSIsImY0V1dyYU9mZ1BVZ25MOUM5Y3JBc2ZJOFdHUXh3Yl9VRURBdzZ1dldtRzAiLCJnaUxyM0VxYzJUTmFvajJQVTlfeE1henNRalhSU1lPYWJJUVdwd1FUUmE4IiwibUZJcGdWT2QzSW03eFZBcktaanBHclBXNmMyYUxZOWNDZTZhaTljLXk4MCIsInpGdUl6ckM0NFZ3OG9kdWhZWHVwY19ibm45X3lBQTVJMkg4RkJ6UjNBUzAiXSwiY25mIjp7Imp3ayI6eyJrdHkiOiJFQyIsImNydiI6IlAtMjU2Iiwia2lkIjoiNE05a0lyQjlXWXp0MUdRZ0wxMmx6ZEJac0d5ZVYzbGdQS292MjhvVDVMNCIsIngiOiJQU3hRckQyemwwX21YY0FxejFtZ3FTZUJvQmhubXgyeXhCRXByQlk4RjIwIiwieSI6InhWOGZiaTFGU29zVXVuTGV1TE51TGtKaXFtWTZUS2lNbnVyLUduMndSMTAifX0sImlhdCI6MTc3MTg0MTU0NSwidmN0IjoiaHR0cHM6Ly9jcmVkZW50aWFscy5leGFtcGxlLmNvbS9kaWdpdGFsX2NyZWRlbnRpYWwiLCJfc2RfYWxnIjoic2hhLTI1NiJ9.76-75fTdyZw3Ij10EwH94skqabjnWvImE_svaOeb10vJKy9olJekjGuC8T4gxLHzrMGozh5JagolFVyenkYo1w~WyJsT0c2TFNMNXZ5Wk9ybmRRaFRkMkFBIiwic3ViIiwiMTAwNCJd~WyJQZDdmV01sMHR5M2hzV2s5LXltNzl3IiwiZ2l2ZW5fbmFtZSIsIkluZ2EiXQ~WyJIQVJXM2hveDRlbjg3MlZ4aUM1Ynl3IiwiZmFtaWx5X25hbWUiLCJTaWx2ZXJzdG9uZSJd~WyJPZGxIYS1YNzBVUmEwNXRUVFFjTmhBIiwiYmlydGhkYXRlIiwiMTk5MS0xMS0wNiJd~"
    }
  ]
}
```

#### Credential Request with Credential Configuration ID Examples for mdoc

Trial Server (trial.authlete.net)

```
curl --oauth2-bearer ${ACCESS_TOKEN} \
     https://trial.authlete.net/api/credential \
     --json '{
  "credential_configuration_id": "org.iso.18013.5.1.mDL"
}'
```

Local Server (localhost)

```
curl --oauth2-bearer ${ACCESS_TOKEN} \
     http://localhost:8080/api/credential \
     --json '{
  "credential_configuration_id": "org.iso.18013.5.1.mDL"
}'
```

#### Credential Response with Credential Configuration ID Examples for mdoc

```json
{
  "credentials": [
    {
      "credential": "ompuYW1lU3BhY2VzoXFvcmcuaXNvLjE4MDEzLjUuMYjYGFhbpGhkaWdlc3RJRAFmcmFuZG9tUDXXZGo1l02eUahWJ7JNT_txZWxlbWVudElkZW50aWZpZXJqaXNzdWVfZGF0ZWxlbGVtZW50VmFsdWXZA-xqMjAyNi0wMi0yM9gYWFykaGRpZ2VzdElEAmZyYW5kb21QTzYxvUEeiWUoxZCIqM5Pq3FlbGVtZW50SWRlbnRpZmllcmtleHBpcnlfZGF0ZWxlbGVtZW50VmFsdWXZA-xqMjAyNy0wMi0yM9gYWFqkaGRpZ2VzdElEA2ZyYW5kb21Qx_FRjIVQ3gP5g6neRBhFh3FlbGVtZW50SWRlbnRpZmllcmtmYW1pbHlfbmFtZWxlbGVtZW50VmFsdWVrU2lsdmVyc3RvbmXYGFhSpGhkaWdlc3RJRARmcmFuZG9tUBnXMkXccNHZlq0mLVlgvZlxZWxlbWVudElkZW50aWZpZXJqZ2l2ZW5fbmFtZWxlbGVtZW50VmFsdWVkSW5nYdgYWFukaGRpZ2VzdElEBWZyYW5kb21Q3o3zMwmpqn2x7B52JYToeXFlbGVtZW50SWRlbnRpZmllcmpiaXJ0aF9kYXRlbGVsZW1lbnRWYWx1ZdkD7GoxOTkxLTExLTA22BhYVaRoZGlnZXN0SUQGZnJhbmRvbVBgLiieEKSuRGFlwk5zf8g0cWVsZW1lbnRJZGVudGlmaWVyb2lzc3VpbmdfY291bnRyeWxlbGVtZW50VmFsdWViVVPYGFhbpGhkaWdlc3RJRAdmcmFuZG9tUEJsSHIFZaCkFSg3fXeUp9xxZWxlbWVudElkZW50aWZpZXJvZG9jdW1lbnRfbnVtYmVybGVsZW1lbnRWYWx1ZWgxMjM0NTY3ONgYWKKkaGRpZ2VzdElECGZyYW5kb21Q_0uXu2oldqRJ9XEF__M3VHFlbGVtZW50SWRlbnRpZmllcnJkcml2aW5nX3ByaXZpbGVnZXNsZWxlbWVudFZhbHVlgaN1dmVoaWNsZV9jYXRlZ29yeV9jb2RlYUFqaXNzdWVfZGF0ZdkD7GoyMDIzLTAxLTAxa2V4cGlyeV9kYXRl2QPsajIwNDMtMDEtMDFqaXNzdWVyQXV0aIRDoQEmoRghWQJMMIICSDCCAe6gAwIBAgIUKJS5GmQjnfcBk3Zb_eW-IhnQk8swCgYIKoZIzj0EAwIwZTELMAkGA1UEBhMCSlAxDjAMBgNVBAgMBVRva3lvMRAwDgYDVQQHDAdDaGl5b2RhMRcwFQYDVQQKDA5BdXRobGV0ZSwgSW5jLjEbMBkGA1UEAwwSdHJpYWwuYXV0aGxldGUubmV0MCAXDTI1MDIyNTA1MzI0OVoYDzIyOTgxMjExMDUzMjQ5WjBlMQswCQYDVQQGEwJKUDEOMAwGA1UECAwFVG9reW8xEDAOBgNVBAcMB0NoaXlvZGExFzAVBgNVBAoMDkF1dGhsZXRlLCBJbmMuMRswGQYDVQQDDBJ0cmlhbC5hdXRobGV0ZS5uZXQwWTATBgcqhkjOPQIBBggqhkjOPQMBBwNCAAQOd6TCwnpnmXqps8RQHq-K9NooFbrjb1yyFxJlHk7WfTVOrZDu9Nq-IOc9yro7y-G9rTf0zt8OWJ5i_WijjILTo3oweDAdBgNVHQ4EFgQUPMYa6fQJP6M6x4tYM1fbf3_E0xMwHwYDVR0jBBgwFoAUPMYa6fQJP6M6x4tYM1fbf3_E0xMwDwYDVR0TAQH_BAUwAwEB_zAlBgNVHREEHjAchhpodHRwczovL3RyaWFsLmF1dGhsZXRlLm5ldDAKBggqhkjOPQQDAgNIADBFAiEAyo3A_oQX_Me8myMWL025cTBw_keceHELfSQHmle5QAECIC5g_ynOmt_nuksfHAeMABf7to52zTRkRduqQCZHO3eYWQHt2BhZAeilZ3ZlcnNpb25jMS4wb2RpZ2VzdEFsZ29yaXRobWdTSEEtMjU2bHZhbHVlRGlnZXN0c6Fxb3JnLmlzby4xODAxMy41LjGoAVggbDeSWf4sa0E0EFdLWdayAuCWB0LaX-bAcR9Qvbybqz0CWCDq6sOBMqf5f8vcYqNCRsceW4qH74zCsas3ihPsxnB0OANYIGQGJYKeByals-5j60qIlHE6D3sHvetuVLUSbMxhZk9EBFggyuhZdvJAyaC0W19W6OOa8jd428BbaP9V20KfzncobTgFWCAGRPvjlnCeYrpl1ScQZP_OQe81gpc7cvQE77986_XmFAZYIIIaD5kwff2lIvf_G_BUYV9V0prHA8CGVOuer3TPPDLhB1ggZG6v_3MmB_ZdfBn2fmAh3DmUZjHF-HYVK-7KlnZG-ywIWCBDKAsRDNyeAHivXfBRUe3j3Vo5sw5E8MzYCykWNbtPlmdkb2NUeXBldW9yZy5pc28uMTgwMTMuNS4xLm1ETGx2YWxpZGl0eUluZm-jZnNpZ25lZMB0MjAyNi0wMi0yM1QwNzozMzoyNFppdmFsaWRGcm9twHQyMDI2LTAyLTIzVDA3OjMzOjI0Wmp2YWxpZFVudGlswHQyMDI3LTAyLTIzVDA3OjMzOjI0WlhAk-6zrAqUAJTZhxjV4W8Lksw5MPCwdiliJdK_AJ2s_beiJ6lGAB_0ly8wqP7dzI4pUuaHIN3RJcDxurlIT-1fdA"
    }
  ]
}
```

NOTE: The value of the `credential` property is a base64url-encoded CBOR data sequence.
Use https://cbor.zone/ to decode it to the diagnostic format (which is human-readable).

### Credential Request with Credential Identifier

Key proof generation using the `generate-key-proof` script in https://github.com/authlete/oid4vci-demo/

```
./generate-key-proof --client-id=${CLIENT_ID} --issuer=https://${CREDENTIAL_ISSUER} --key=${JWK_FILE} --nonce=${NONCE}
```

Credential Request

```
curl --oauth2-bearer ${ACCESS_TOKEN} \
     https://${CREDENTIAL_ISSUER}/api/credential \
     --json '{
  "credential_identifier": "'${CREDENTIAL_IDENTIFIER}'",
  "proofs": {
    "jwt": ["'${KEY_PROOF}'"]
  }
}'
```

#### Credential Request with Credential Identifier Examples for SD-JWT VC

Trial Server (trial.authlete.net)

```
NONCE=`curl https://trial.authlete.net/api/nonce -X POST | jq -r .c_nonce`
KEY_PROOF=`./generate-key-proof --client-id=trial_client --issuer=https://trial.authlete.net --key=holder.jwk --nonce=${NONCE}`
```

```
curl --oauth2-bearer ${ACCESS_TOKEN} \
     https://trial.authlete.net/api/credential \
     --json '{
  "credential_identifier": "DigitalCredential/Type1",
  "proofs": {
    "jwt": ["'${KEY_PROOF}'"]
  }
}'
```

Local Server (localhost)

```
NONCE=`curl http://localhost:8080/api/nonce -X POST | jq -r .c_nonce`
KEY_PROOF=`./generate-key-proof --client-id=trial_client --issuer=https://trial.authlete.net --key=holder.jwk --nonce=${NONCE}`
```

```
curl --oauth2-bearer ${ACCESS_TOKEN} \
     http://localhost:8080/api/credential \
     --json '{
  "credential_identifier": "DigitalCredential/Type1",
  "proofs": {
    "jwt": ["'${KEY_PROOF}'"]
  }
}'
```
