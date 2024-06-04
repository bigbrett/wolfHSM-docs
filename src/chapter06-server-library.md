# wolfHSM Server Library

The wolfHSM server library is a server-side implementation of the wolfCrypt cryptography library.
It provides an interface for applications to offload cryptographic operations to a
dedicated server, which runs the wolfHSM server software. This allows the application to perform
cryptographic operations without having to manage the cryptographic keys or perform the
operations locally.

## Getting Started
TODO

## Architecture
TODO

## API Reference
TODO

## Key Management
TODO

## Cryptographic

wolfHSM uses wplfCrypt for all cryptographic operations, these incude Chinese mandated ones such as
SM2, SM3 and SM3. It also supports post-quantum cryptography such as Kyber, LMS, XMSS.

