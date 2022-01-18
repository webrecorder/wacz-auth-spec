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

Other proposals, such as Signed Exchanges, provide new ways for web servers to serve content that could be non-repudiation. However, this introduces a new format
and must be supported by the web server.

The goal of this specification is to provide a way to sign and authenticated archived web content through the client creating the web archive.

This proposal will provide a mechanism to prove:
- who created a web archive
- when the web archive was created.

This requires trusting the client or a trusted third party to create the web archive. This proposal does not make any guarantees from the perspective of the web server serving the content,
as this is not currently possible.
