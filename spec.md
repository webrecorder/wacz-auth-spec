# Introduction

This document is a working draft for a proposal to create signed WACZ packages which allow the packages author to be cryptographically proven.

The specification is an addendum to he main [WACZ Specification](https://webrecorder.github.io/wacz-spec/)

## Motivation

The purpose of this specification is to provide an optional mechanism to make web archives bundled in WACZ more trusted.

The WACZ format provides a way to create portable web archives and distribute them through any online network in a decentralized manner.

To increase trust in web archives, it becomes necessary to guarantee certain properties about who the web archive was created and when.

## Provenance of Authenticity

Proving web archive authenticity can be difficult. Ideally, proof of authenticity could guarantee that any web server served a particular URL at a particular point in time.
Unfortunately, this is not currently possible with existing web standards, even with TLS, as TLS does not provide "non-repudiation".

There are proposals, such as [signed exchanges](https://wicg.github.io/webpackage/draft-yasskin-http-origin-signed-responses.html), which provide new ways for web servers to serve content that could later be verified. This requires additional support for nwe formats on the HTTP/S server.

The goal of this specification is to provide a way for a client that is creating a web archive to add authentication information. As such, this must work with archives of any HTTP and HTTPS web content.

This proposal will provide a mechanism to prove:
- who created a web archive
- when the web archive was created.

This requires trusting the client or a trusted third party to create the web archive. This proposal does not make any guarantees from the perspective of the web server serving the content, as this is not currently possible.


## WACZ Signature File and Format

The WACZ format builds on top of the Frictionless Data Package. The data package includes a manifest `datapackage.json` file which contains the hashes of all the files in the data package.

A signed WACZ also contains a `datapackage-digest.json`, which contains a hash and signature of the `datapackage.json`

(Note: While this specification is primarily designed for WACZ files, the approach described here may be adapted to any Frictionless Data Package-based specification)

### `datapackage-digest.json` File

The `datapackage-digest.json` file, included in the root of the WACZ file, contains the following structure:

```
{
  "path": "datapackage.json",
  "hash": "<sha256 hash of datapackage.json>"
  "signedData": <SignatureData struct>
}
```

### Signature Data Format

The SignatureData structure is the output of the signing operation contains be one of the following formats,
a signature with a public key, which requires external key management, and a certificate signed signature,
which can be validated by publicly available certificates.


#### Anonymous Signature

```json
{
    // input hash to sign
    "hash": "<sha256 hash of datapackage.json>,
  
    // metadata info
    "created": "<ISO 8861 Date>",
    "software": "<string>",
    "version": "<string>",
  
    // signature of hash
    "signature": "<base64 encoded signature>",
  
    // public key of keypair used to created the signature
    "publicKey": "<base64 encoded public key (ECDSA) >",
}
```

With this approach, the WACZ contains just enough to validate that they signature with the publicKey.
  
To validate authorship of the WACZ, external key management is required, and this signature is otherwise anonymous.
  
Currently, this approach is used in decentralized tooling, such as the ArchiveWeb.page extension.
  

#### Observer Attestation: Domain-Ownership Identity + Signed Timestamp

```
{
    // input hash to sign
    "hash": "<sha256 hash of datapackage.json>,
  
    // metadata info
    "created": "<ISO 8861 Date>",
    "software": "<string>",
    "version": "<string>",
  
    // signature of 'hash' by domainCert
    "signature": "<base64 encoded signature>",
    "domain": "<valid hostname>",
    "domainCert": "<PEM certificate chain>",
  
    // signature of 'signature' by timestampCert
    "timeSignature": "<base64 encoded signature>",
    "timestampCert": "<PEM ceriticate chain>",
    
    // optional: cross-signing cert for "signature"
    "crossSignedCert": "<PEM certificate chain>"
}
```

This approach allows for the WACZ signature to be created by the same private key as is used to create a TLS certificate for a particular domain.
  
The creator of the WACZ file is the same as the owner of a particular TLS certificate, which can be explored via Certificate Transparency logs.

This approach also includes an RFC 3161 timestamp server `timeSignature` of the first `signature`.

The `timeSignature` includes the timestampped and is designed to further guarantee that the signature was created close to the specified creation time.

For additional verification, an optional `crossSignedCert` can be provided which can be used as an alternative to the `domainCert`, in case the domain
certificate has been found to be compromised for any reason. The cross-signed certificate simply provides a way to provide an alternative trust path 
not tied to the domainCert, if it becomes needed for any reason.

## Signing

To generate the `signatureData`, the creator of the WACZ can perform the following steps.

### Anonymous Signing

At minimum, the creator of the WACZ can create the anonymous signature `signatureData`, containing only its public key and signature.

With this approach, the client must distribute its public key out-of-band to make WACZ verification possible.

### Domain-Name Identity + Timestamp Signing

To create the signatureData in the second example, the client must perform the following:

1) Generate a private ECDSA keypair
2) Create a Certificate Request signed by the private key
3) Receive a TLS certificate for its CSR from a trusted CA, such as LetsEncrypt.
4) Optionally: use a second CA (such as a private self-signed CA) to generate a second cross-signed certificate as backup.
5) Sign the hash using its private key to generate the first signature (signature)
6) Use an RFC 3161 timestamp server to sign the previous signature (timeSignature)

This approach is based on a 'trusted-third party' which securely creates and signs the WACZ files.

The intended workflow for this approach is that the creator of the WACZ has a secure, private connection
to the attestation server which is running on the designated domain and can observe and sign the requested hash.
Access to the attestation server must be restricted to trusted creator of the WACZ.
Establishing this secure connection is beyond the scope of this specification.







