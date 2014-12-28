# EncryptingCamera Specification

## What is EncryptingCamera?

This is an effort to create camera applications for popular mobile devices, that perform seemless encryption - ensuring that if a device is later stolen or seized, the photos on it can't be accessed. All pictures taken with the application will be encrypted using curve25519/XSalsa20/Poly1305 - this way the device need only know the user's public key. The user should store their private key, that allows decryption offline.

## What platforms are supported?

The goal is to support as many of the common device and desktop platforms as possible; for the initial version, Linux and Android will be supported. Other platforms will be added as quickly as possible.

## What's the status?

This project is still in the planning phase. We will begin development once the specification is stable.
