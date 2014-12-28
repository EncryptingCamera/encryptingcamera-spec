# EncryptingCamera Specification

## What is EncryptingCamera?

This is an effort to create camera applications for popular mobile devices, that perform seamless encryption - ensuring that if a device is later stolen or seized, the photos on it can't be accessed. All pictures taken with the application will be encrypted using curve25519/XSalsa20/Poly1305 - this way the device need only know the user's public key. The user should store their private key, that allows decryption offline.

## Intended Users

*Journalist:* A journalist, working a story that may displease local authorities needs to take photos, but needs to ensure that the local authorities are unable to review or censor selected photos. In this case, if the local authorities seize the journalists phone, they will be able to determine that photos were taken, but not where they were taken, or what the photos contain.

*Protester:* A protester at a demonstration wants to document the behavior of the police and other protesters, but fears that the phone may be seized and used against them, or other protesters. In this case, if the device is seized, it will be impossible from the device to decrypt the images to determine what they contain or who they feature.

*International Traveler:* A person travels to a country that is known to aggressively screen devices for any material that may reflect poorly on the government. In this case, the inspectors at the border will be unable to determine what the photos contain, and may either remove them all, or allow all to pass. Selective censorship would not be possible.

## Users & Cases Not Supported

In the case that a user doesn’t know, or suspects that the device may have been tampered with, it is possible that malware may capture pictures taken at the operating system level, or modify the application to make a copy of images before encrypting them.

This application is only intended to be secure on devices that were secure at the time that a photo was taken. 

## What platforms are supported?

The goal is to support as many of the common device and desktop platforms as possible; for the initial version, Linux and Android will be supported. Other platforms will be added as quickly as possible.

## What's the status?

This project is still in the planning phase. We will begin development once the specification is stable.
