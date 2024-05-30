# Introduction

This manual is written as a technical guide to the wolfHSM embedded hardware security module library. It
will explain how to build and get started with wolfHSM, provide an overview of build
options, features, portability enhancements, support, and much more.

You can find the PDF version of this document [here](https://www.wolfssl.com/documentation/manuals/wolfHSM/wolfHSM-Manual.pdf).

## Why Choose wolfHSM?

Automotive HSMs (Hardware Security Modules) dramatically improve the security of cryptographic keys and cryptographic processing by isolating signature verification and cryptographic execution, which are the core of security, into physically independent processors. Automotive HSMs are mandatory or strongly recommended for ECU’s that require robust security. With this in mind, wolfSSL has ported our popular, well tested, and industry leading cryptographic library to run in popular Automotive HSMs like Aurix Tricore TC3XX.

wolfHSM provides a portable and open-source abstraction to hardware cryptography, non-volatile memory, and isolated secure processing that maximizes security and performance for ECUs. By integrating the wolfCrypt software crypto engine on hardware HSM’s like Infineon Aurix Tricore TC3XX, Chinese mandated government algorithms like SM2, SM3, SM4 are available. Additionally, Post Quantum Cryptography algos like Kyber, LMS, XMSS and others are easily made available to automotive users to meet customer requirements. At the same time, when hardware cryptographic processing is available on the HSM, we consume it to enhance performance.

wolfBoot is a mature and portable secure bootloader solution designed for bare-metal bootloaders and equipped with failsafe NVM controls. It offers comprehensive firmware authentication and update mechanisms, leveraging a minimalistic design and a tiny HAL API, which makes it fully independent from any operating system or bare-metal application. wolfBoot manages the flash interface and pre-boot environment, accurately measures and authenticates applications, and utilizes low-level hardware cryptography as needed. wolfBoot can use the wolfHSM client to support HSM-assisted application core secure boot, Additionally, wolfBoot can run on the HSM core to ensure the HSM server is intact, offering a secondary layer protection. This setup ensures a secure boot sequence, aligning well with the booting processes of HSM cores that rely on NVM support.

## Features

WolfHSM also comes with an extensive list of additional features:
- Extensibility of cryptographic algorithms
- consistency with security functions 
- integration with Autosar
- integration with SHE+
- Direct usage of HSM from wolCrypt's externalized API's
- PKCS11 interface available
- TPM 2.0 interface available
- Secure OnBoard Communication (SecOC)
- Module integration available
- Certificate handling available
- Symmetric and Asymmetric keys and cryptography
- Customization available
- FIPS 140-3 available




