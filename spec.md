# EncryptedCamera Design Specification

The purpose of this document is to describe the goals, non-goals, functional requirements, and threat model supported by this project. All official applications released by this project must abide by this document.

## Project Overview

Many people find themselves in situations where they need to take photographs, they have a mobile device with a camera available (Android, iOS, etc.) - but they worry that because of the circumstances that there device may be later searched, and photographs censored, or used against the photographer or others.

This project aims to provide applications for common platforms, to allow the public to take photographs that are encrypted using the public key of their choosing. The corresponding private key may be on their desktop, stored offline at a secure location, or in the control of another person. Only with the corresponding private key, can the photographs be decrypted and viewed.

## How this is achieved

The system is made up of two applications - a desktop application (Windows, OSX, *nix), and a mobile application (Andriod, iOS). The applications will have the following responsibilities:

**Desktop:**
* Generate Public/Private Key Pair
* Decrypt Images

**Mobile:**
* Capture Public Key
* Capture Images
* Encrypt Images

The mobile application will have no ability to decrypt images, or handle the private key. As such, the images can only be viewed from the desktop application, with the appropriate private key.

---

The user will provide the mobile application with the proper public key, which will be used to encrypt all images. When the user takes a picture with the application, it is immediately encrypted with the provide public key prior to being saved to the device's storage.

## Users & Use Cases

*Journalist:* A journalist, working a story that may displease local authorities needs to take photos, but needs to ensure that the local authorities are unable to review or censor selected photos. In this case, if the local authorities seize the journalists phone, they will be able to determine that photos were taken, but not where they were taken, or what the photos contain.

*Protester:* A protester at a demonstration wants to document the behavior of the police and other protesters, but fears that the phone may be seized and used against them, or other protesters. In this case, if the device is seized, it will be impossible from the device to decrypt the images to determine what they contain or who they feature.

*International Traveler:* A person travels to a country that is known to aggressively screen devices for any material that may reflect poorly on the government. In this case, the inspectors at the border will be unable to determine what the photos contain, and may either remove them all, or allow all to pass. Selective censorship would not be possible.


## Encryption

All crypto operations will be performed by the most recent stable version of `libsodium` - the library is trusted to be secure and well designed with regard to side-channel resistance.

Files will be encrypted with curve25519/XSalsa20/Poly1305 via the libsodium `crypto_box` / `crypto_box_easy` methods. The application will use the provided public key, and a newly generated ephemeral key pair to perform each file encryption. The ephemeral public key will be prepended to the file when written, the ephemeral private key MUST NOT be persisted to any form of storage and must be zeroed from memory as soon as the encryption completed.

All key pairs will be randomly generated; deterministic generation should not be used.

## File Format

Image files will be saved in the following format:

* `ver` - The file format version, one byte, the current version is `0x1`
* `pub` - The ephemeral public key that corresponds to the ephemeral private key used during encryption. This is required to perform the decryption. 32 bytes.
* `ciphertext` - Encrypted JPEG data. File length minus 33 bytes.

File will be written in the following format (no encoding will be applied):

`ver || pub || ciphertext`

## Non-Goals

The following features / functionality / threats are considered to be out of scope at this time. These items may be removed at a later time, but are considered out of scope at the present.

* Video - Supporting video will add quite a few complications and needs additional research to determine feasibility.
* Insecure Desktops - No attempt will be made by the desktop applications to protect the secrecy of the images or keys should the desktop be insecure. The presence of malware or other backdoors will render moot the security provided by this system; at this time it is beyond the scope of this project to address this situation.
* Insecure Mobile Devices (prior to image capture) - If the mobile device has been tampered with, such as the installation of malware, it is impossible to know if the application or image capture API has been modified. As such, this can only provide protection after the image has been captured and encrypted.

## Threat Model

This is intended to be an informal threat model, intended to provide insight to users and guidance to developers.

### Assumptions

The following assumptions apply to all security goals.

* Mobile device is secure and has not been tampered with at the time the image is captured and encrypted. If the device has been tampered with prior to image capture, the application could be modified or the API modified to save unencrypted copies or perform other actions that bypass the protections provided by this system.
* User’s desktop device is secure, and does not contain any malware or other malicious modifications. If, at any time, the desktop device is compromised, the private key and photos stored on the device, or later decrypted on the device, should be considered compromised as well.

### Security Goals

* Protect images from being viewed by an unauthorized third-party.
* Prevent images from being altered after capture.
* Generate secure public / private keys.

### Security Non-Goals

These items are specifically excluded from the security profile of the system.

* Preventing captured images from being deleted from the device by an unauthorized party.
* Maintaining security of the private key or photos on a compromised desktop device.
* Meeting security goals on a mobile device that was compromised prior to image capture.
* Preventing a compromised mobile device from compromising the user’s desktop device.
* Preventing an unauthorized party with access to the mobile device from determining any of the following:
 * Number of photos taken
 * Time & date photos were taken

### Threats

The following are threats that it is believed that the system is subject to. This list may not be exhaustive, and should be updated as new threats are identified.

#### Backdoored Copy of Mobile Application

As the official applications are all released as open source, it’s a simple task that an attacker could release a modified version that bypass the security properties of the system. This can be done in a variety of ways:

* Replace the ephemeral key pair with a fix key pair, allowing an attacker to decrypt images without the user’s private key.
* Save a copy of the image to a different location, before encryption is performed.
* Add a remote access shell, or other backdoor to allow the attacker to perform attacks against the mobile device, or the desktop device when connected.

The exact impact varies by method used, but in any case would cause a bypass of the intended security goals.

**Mitigation / Solution:** TBD.

#### Malicious Mobile Application Update

An attacker that was able to acquire credentials needed to publish an application update, could modify the application to add malicious code. The attacker would be able to perform actions similar to the above.

**Mitigation / Solution:** TBD.

#### Cryptographic Breakthrough

Should a flaw be found in the selected cryptographic primitives be broken, it would allow an attacker to access encrypted data and/or modify existing files.

**Mitigation / Solution:** The primitives used have been well studied and are believed to be generally secure. The likelihood of such an event is low.

Should such an event occur, it would be necessary to update the applications to use a different set of primitives.

#### Image Replacement

An attacker that gains access to a mobile device could remove images, and replace them with other images.

From the file information, the attacker would know when an image was taken, they could use this information to craft a replacement file that includes the appropriate EXIF data to indicate that it was taken at the desired time. The crafted image could replace existing images.

This could be used to replace images that are suspected to be sensitive, without the user being aware.

**Mitigation / Solution:** TBD.

#### Malicious Image Replacement

An attacker that gains access to a mobile device could place specially crafted images that leverage vulnerabilities in image viewers, the GUI applications, or others.

Such an attack could compromise the user’s desktop device, bypassing the security of the system.

**Mitigation / Solution:** TBD.

## Key Escrow & Recovery, Backdoors, Third-Party Access

To ensure the integrity of this project, and the safety of users, the following MUST NOT be supported in any application distributed by the project:

* Key escrow mechanisms
* Key recovery / backup
* Backdoors for law enforcement and/or government use
* Any form of third-party access to keys or decrypted data

Should the project or the developers be compelled to add such functionality by a court, coercion, or force, the project will halt all development and all signing keys will be destroyed.
