# Porting Guide

## Porting Guide Goals

This porting guide aims to provide you with the necessary resources and knowledge to accurately and correctly port wolfHSM. In this section of the guide, we will look at the following: 

- WolfHSM port applications
- WolfHSM porting interface
- WolfHSM porting general configuration 
- WolfHSM porting example for Posix port
- WolfHSM porting skeleton 
- WolfHSM porting Infineon Aurix TC3xx
- WolfHSM porting ST SPC58NN

## WolfHSM Porting

wolfHSM itself is not executable and it does not contain any code to interact with any specific hardware. In order for wolfHSM to run on a specific device, the library must be configured with the necessary hardware drivers and abstraction layers so that the server application can run and communicate with the client. Specifically, getting wolfHSM to run on real hardware requires the implementation of the following:

- Server application startup and hardware initialization
- Server wolfCrypt configuration
- Server non-volatile memory configuration
- Server and client transport configuration
- Server and client connection handling

The code that provides these requirements and wraps the server API into a bootable application is collectively referred to as a wolfHSM "port".

Official ports of wolfHSM are provided for various supported architectures, with each port providing the implementation of the wolfHSM abstractions tailored to the specific device. Each port contains:

- Standalone Reference Server Application: This application is meant to run on the HSM core and handle all secure operations. It comes fully functional out-of-the-box but can also be customized by the end user to support additional use cases
- Client Library: This library can be linked against user applications to facilitate communication with the server

# Ports

## Infineon Aurix TC3XX

(Port in progress)
The distribution of this  port is restricted by the vendor.  Please contact support@wolfssl.com for access. 

Infineon Aurix TC3xx
- Up to 6x 300MHz TriCore application cores
- 1x 100MHz ARM Cortex M3 HSM core
- Crypto offload: TRNG, AES128, ECDSA, ED25519, SHA

## ST SPC58NN

(Port in progress)
The distribution of this  port is restricted by the vendor.  Please contact support@wolfssl.com for access. 

ST SPC58NN
- 3x 200MHz e200z4256 PowerPC application cores
- 1x 100MHz e200z0 PowerPC HSM core with NVM
- Crypto offload: TRNG, AES128

## Posix

The Posix port provides multiple and fully functional implementations of different wolfHSM abstractions that can be used to better understand the exact functionality expected for different hardware abstractions.

The Posix port provides:
- Memory buffer transport
- TCP transport
- Unix domain transport
- NVM device (using a filesystem)
- Flash device (using a file as a backing store)

## Skeleton 

The Skeleton port source code provides a non-functioning layout to be used as a starting point for future hardware/platform ports.  Each function provides the basic description and expected flow with error cases explained so that ports can be used interchangeably with consistent results.

The Skeleton  port provides implementations of:
- Transport 
- NVM Device
- Flash Device
- Crytp Device

# WolfHSM Porting Interface

## WolfHSM Porting Interface Explained 
When porting WolfHSM into a new platform or ours, you will need to implement some hardware-specific interfaces. We can provide helper functions for flash-like devices. You will need to provide (read, erase, program, blank check, etc.). Our library will likely work on top of it. We can then provide the object layer on top of that. You will also need to provide crypto hardware interfaces like TRNG, Keys, and symmetric/asymmetric crypto. From a platform interface perspective, you will have to understand your system's boot sequence and how the application cores get reset or marked to boot off of it at a specific start location. You may also need to control shared memory locations and memory translation units. The configuration you want to select is selected at compile time rather than link time. In your code, you select which libraries you want to use; here's the config of the libraries, and then work that at compile time, which is an AUTOSAR requirement. 

## WolfHSM Porting Interface

Ports must implement hardware-specific interfaces:
- Non-volatile memory
- Flash-like devices (read, erase, program, blankcheck, etc)

Crypto Hardware
- TRNG, Keys, symmetric/asymmetric crypto

Platform Interface
- Boot sequence, application core reset, memory limitations
- Port and configuration are selected at compile time

# General Porting Configuration 

## WolfHSM Port Configuration Example

Flash Interface using Posix flash file simulator

```c
#include “wolfhsm/wh_nvm_flash.h”								/*Server NVM Manager using flash */
#include “wolfhsm/wh_flash.h”                  /*Abstract Flash API */
#include “port/posix/posix_flash_file.h”       /*Port Implementation of Flash API */

/* Specify flash callback functions */
const whFlashCb pff_cb[1] = { POSIX_FLASH_FILE_CB };

/* Allocate implementation-specific context with empty initializer */
posixFlashFileContext pff_context[1] = {0};

/* Specify implementation-dependent configuration */
posixFlashFileConfig pff_cfg[1] = {{
	.filename       = “my_flash_file.bin”,
	.partition_size = 16384,
	.erased_byte    = 0xFF,
}};

/* Connect NVM Flash layer to this PFF instance */
const whNvmFlashConfig nvm_flash_config[1] = {{
	.cb      = pff_cb,
	.context = (void*)pff_context,
	.config  = (const void*)pff_config,
}};
```

# WolfHSM Posix Porting Example

## General Steps for Porting Configuration 

## Porting Configuration Example

# WolfHSM Porting Skeleton 

## General Steps for Porting Configuration 

## Porting Configuration Example

# WolfHSM Porting Infineon Aurix TC3xx

## General Steps for Porting Configuration 

## Porting Configuration Example

# WolfHSM Porting ST SPC58NN

## General Steps for Porting Configuration 

## Porting Configuration Example

# Summary

* Each of the implemented port-specific code and resources are kept in port directories organized first by platform vendor and possibly further by product.  Each of the ports is expected to provide the glue logic between wolfHSM abstractions (transport, NVM objects, flash, and cryptocb) and the native or vendor-provided libraries.

Due to the sensitive nature of some platform code, not all of the source and glue logic can be provided in this public repo, but the base directories of these ports are listed here with any public interfaces that can be provided and additional contact information.