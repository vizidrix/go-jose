
## JSON Object Signing and Encryption (JOSE)

[ ![Codeship Status for vizidrix/jose](https://codeship.com/projects/3bb2a380-de23-0132-e133-2204af7cc628/status?branch=master)](https://codeship.com/projects/80367)
[![GoDoc](https://godoc.org/github.com/vizidrix/jose?status.png)](https://godoc.org/github.com/vizidrix/jose)

The goal of this implementation is to:
- Provide a clean api for interacting with JOSE tokens
- Follow the existing standard as closely as possible
	- [OpenID Connect](openid.bitbucket.org/openid-connect-core-1_0.html)
	- [JWS Spec](https://tools.ietf.org/html/draft-ietf-jose-json-web-signature-41)
	- [JWA Spec](tools.ietf.org/html/draft-ietf-jose-json-web-algorithms)
	- [JWE Spec](https://tools.ietf.org/html/draft-ietf-jose-json-web-encryption-17)
	- [JWK Spec](https://tools.ietf.org/html/draft-ietf-jose-json-web-key-41)
		- [WebCryptoAPI](http://www.w3.org/TR/2014/CR-WebCryptoAPI-20141211/)
	- [JWT Spec](http://self-issued.info/docs/draft-ietf-oauth-json-web-token.html)
- Avoid [Critical Security Vulnerabilities in JWT](https://auth0.com/blog/2015/03/31/critical-vulnerabilities-in-json-web-token-libraries/)
	- Disable "none" by default (see below for an example of re-enabling it)
	- Don't rely on "alg" param for identifying verification requirements
	- Implement and check JWK entries with verifiable [kid]s
* [Follow up to Vulnerabilities](https://threatpost.com/critical-vulnerabilities-affect-json-web-token-libraries/111943)

> Although many of the fields are strictly optional we recommend including unique and/or variable data in each token (i.e. at least one timestamp or a unique, random, nonce)

> * A nonce is generated by default unless disabled

## Examples

> Make an empty JWS with "None" and decode it

> { notice that explicitly enabling the algo is required in both encoding and decoding}

````golang
rem_none := jose.RemoveConstraints(j.None_Algo)
jwt := jose.New(rem_none)
token, _ := jwt.GetToken()
_, err := jose.Decode(token, rem_none)
````

> Make an HMAC 256 signed token

````golang
jwt := jose.New(
	jose.HS256("key_id", []byte("secret")).Signer(),
	jose.Id("10"),
)
token, err := jwt.GetToken()
if err != nil {
	// Error building token - validation error(s)
	return
}
log.Printf("My HMAC+SHA256 Signed Token: [ %s ]", token)
...

jwt_parsed := jwt.Decode(token, jose.HS256("key_id", []byte("secret")).Signer())
if errs := jwt_parsed.GetErrors(); len(errs) != 0 {
	// Parse error(s) - Covers both structural and/or validation errors
}
...
````

> You can also define keys at server load to standardize and simplify calls

````
key := jose.HS512("key_id", []byte("secret")).Signer()
...

token, err := jose.New(key, jose.Id("10")).GetToken()
if err != nil {
	// Error building token: [ %s ]", token)
	return
}
log.Printf("My HMAC+SHA512 Signed Token: [ %s ]", token)
...

jwt_parsed := jwt.Decode(token, key)
if errs := jwt_parsed.GetErrors(); len(errs) != 0 {
	// Parse error(s) - Covers both structural and/or validation errors
}
...
````

## Some Light Reading

[Additional Security Guidelines](https://tools.ietf.org/html/draft-ietf-jose-json-web-signature-41#section-10)

[Message Signature or MAC Computation](https://tools.ietf.org/html/draft-ietf-jose-json-web-signature-41#section-5.1)

[Google OAuth Assertion Flow](https://sites.google.com/site/oauthgoog/Home/google-oauth2-assertion-flow)

[Brian David Campbell - Fantastic Overview](http://www.slideshare.net/briandavidcampbell/i-left-my-jwt-in-san-jose)

[JWT for microServices | Good OAuth2.0 Visualization](http://www.slideshare.net/alvarosanchezmariscal/stateless-authentication-for-microservices)

[Handbook of Applied Cryptography - Excerpt](http://cacr.uwaterloo.ca/hac/about/chap8.pdf)

[Generating Secure Random in Golang](http://elithrar.github.io/article/generating-secure-random-numbers-crypto-rand/)

[JWK Selector - Lookup Criteria](http://connect2id.com/products/nimbus-jose-jwt/examples/jwk-selectors)

## HTTP Basics for Auth

> 401 "Unauthorized" == Unauthenticated i.e. Don't know who you are or don't recognize your token

> 403 "Forbidden" == Unauthorized i.e. Identity known but request rejected



# JWS Algorithms

| "alg" Param(s) | Description |
| :-- | :--: |
| none | Unsigned Plaintext |
| HS256, HS384 and HS512 | HMAC w/ SHA2 |
| RS256, RS384 and RS512 | RSASSA-PKCS1-V1_5 Digital Signatures with SHA2 |
| ES256, ES384 and ES512 | Elliptic Curve Digital Signatures (ECDSA) with SHA2 |
| PS256, PS384 and PS512 | RSASSA-PSS Digital Signatures with SHA2 |

# JWE Info

- "alg": Algorithm
- "enc": Encryption Method
- "zip": Compression Algorithm ("DEF" DEFLATE Compressed Data Format only supported?)

````
{header}.{encrypted_key}.{initialization_vector}.{cipher_tet}.{authentication_tag}
````

| "enc" Param(s) | Description |
| :-- | :--: |
| A128GCM, A192GCM and A256GCM | Authenticated encryption with Advanced Encryption Standard (AES) in Galois/Counter Mode (GCM) |
| A128CBC-HS256, A192CBC-HS384 and A256CBC-HS512 | Authenticated encryption with AES-CBC and HMAC-SHA2 composite |

| "alg" Param(s) | Description |
| :-- | :--: |
| dir | Direct encryption with a shared symmetric key |
| RSA1_5 | RSAES-PKCS1-V1_5 key encryption |
| RSA-OAEP and RSA-OAEP-256 | RSAES using OAEP key encryption |
| A128KW, A192KW and A256KW | AES key wrap |
| A128GCMKW, A192GCMKW and A256GCMKW | AES GCM key encryption |
| ECDH-ES | Elliptic Curve Diffie-Hellman Ephemeral Static Key Agreement using Concat KDF |
| ECDH-ES+A128KW, ECDH-ES+A192KW and ECDH-ES+A256KW | Elliptic Curve Diffie-Hellman Ephemeral Static Key Agreement using Concat KDF with AES key wrap |
| PBES2-HS256+A128KW, PBES2-HS384+A192KW and PBES2-? | PBES2 with HMAC SHA-2 and AES key wrap |


# JWK Info

JSON data structure representing cryptographic key(s)
- Public/Private Keys: RSA & Eliptic Curve
- Symmetric Keys: Octet Sequence

Uses:
- Included in JWS / JWE / JWT Headers ( See [security vulnerability](https://auth0.com/blog/2015/03/31/critical-vulnerabilities-in-json-web-token-libraries/) for why JWK is important to include )
- Published, over TLS, and referenced inside the data set
- Can replace Self-Signed Certificates
- Saved to Disk, Sent Over-The-Wire (email, etc)

> JWK Example [ RSA-SHA256 | Key ID: '9er' | Public Key Uri ]

````json
{"alg":"RS256","kid":"9er","jku":"https://www.example.com/jwks"}
````

## Key Definition Guidelines

Inclusion of both "use" and "key_ops" should be avoided
- Prefer either/or if possible
- If both are included they must match purpose (Sign vs Encrypt)

Duplicate Key_Ops not permitted on a single key
- Field is optional unless needed for processing

Multiple unrelated key ops should be avoided
- If included ops should be complementary
	- "sign" with "verify"
	- "encrypt" with "decrypt"
	- "wrapKey" with "unwrapKey"

Key Id should be unique within a set
- Except in cases where Key Type is different, indicating multiple valid options

Key Types not understood by the server should be rejected outright

#TODO:
- WebKey structs for other providers
	- EC: curve, x, y
	- RSA: modulus, exponent
Add additional features:
- Option to set skew - variance allowed by clock time checks
- Option to provide time - to keep the library pure
- QueryStringHash string `json:"qsh"`
	- Hash producded from the Method, Relative Path plus the Sorted Set of Query String Params, used to prevent URL tampering
