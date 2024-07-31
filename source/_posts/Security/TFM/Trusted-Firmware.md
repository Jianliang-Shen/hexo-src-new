---
title: ARM Trusted Firmware
date: 2021-05-28 20:59:25
index_img: /img/index_img/tfm_logo.png
tags:
    - TF-M
    - Firmware
    - TEE
    - PSA
    - Arm
    - Security
categories:
    - Security
---

<!-- more -->

## The steps of Threat Modeling:

1. **Model System**: Model the system you're building, deploying, or changing
2. **Find Threats**: Find threats using that model and the approaches
3. **Address Threats**: Address threats using the approaches
4. **Validate**: Validate your work for completeness and effectiveness

People often use an approach centered on models of their **assets**, models of **attackers**, or models of their **software**.

### Finding Threats

Using **STRIDE** model and **Attack Trees, Attack Libraries, Privacy Tools** to find threats in system. The STRIDE model is:

* **Spoofing**: Pretending to be something or someone other than yourself
* **Tampering**: Modifying something on disk, on a network, or in memory
* **Repudiation**: Claiming that you didn’t do something, or were not responsible. Repudiation can be honest or false, and the key question for system designers is, what evidence do you have?
* **Information Disclosure**: Providing information to someone not authorized to see it
* **Denial of Service**: Absorbing resources needed to provide service
* **Elevation of Privilege**: Allowing someone to do something they’re not authorized to do

### Defensive Technologies

* **Authentication**: Mitigating Spoofing
  * mainly technologies: Digital signatures and Hashes
  * Methods for authenticating people: password, access card or biometric information
* **Integrity**: Mitigating Tampering
  * using digital signatures or hashes to avoid tampering
* **Non-Repudiatio**n: Mitigating Repudiation
  * using logs or digital signatures to avoid reputation
* **Confidentialit**y: Mitigating Information Disclosure
  * using cryptography and keys
* **Availability**: Mitigating Denial of Service
* **Authorization**: Mitigating Elevation of Privilege

### Threat modeling of TF-M(TMSA)

TMSA of Network Camera.

The **assets** of system:

* Camera ID: A unique ID to identify the device on a network, such as a Media Access Control (MAC) address.
* Firmware: The camera’s firmware.
* Firmware Certificate:  The cryptographic certificate used to authenticate firmware and firmware updates.
* Logs: The event logs, that can be used to detect suspicious activities.
* Video Stream: The video stream produced by the camera sent over the network.
* Configuration: The camera’s dynamic configuration, including network configuration such as the name of a Wireless Local Area Network (WLAN), or IP and Domain Name System (DNS) addresses and camera settings such as pan, tilt, and zoom, the events to be detected and notified.
* Credentials: The authentication credentials, used for local and remote authentication
  * Network credentials, to authenticate if needed on the network, for instance a Wi-Fi pre-shared key or a 802.1x certificate, to be protected in integrity and confidentiality.
  * Device authentication credentials to authenticate on remote servers, to be protected in integrity and confidentiality.
  * Server authentication data, such as public key certificates, to be protected in integrity.
  * Session keys, used after establishment of a trusted communication channel with servers, to be protected in integrity and confidentiality.
  * Administration and user credentials, to authenticate to the services provided by the network camera, either for administration or for regular use, to be protected in integrity and confidentiality.
  * User biometric patterns to be used in face recognition or similar algorithms, to be protected in integrity.

The **threats** of system:

* **IMPERSONATION**
  * An attacker impersonates a legitimate user on the camera, either a regular user that can access the video stream or an admin user.
  * The user credentials may be obtained through default admin passwords, interception, for instance in insecure communication links, or exposed through data disclosure.
  * The attacker may then access video stream, modify configuration or try to modify firmware.
  * Assets threatened directly: Credentials Assets threatened indirectly: Video Stream, Configuration.
* **MITM**
  * An attacker performs a Man-In-The-Middle attack or impersonates a server the camera connects to, for instance to upload the video stream or the event logs.
  * The attacker may rely on insecure communication links or prior modification of the server credentials on the camera through insecure configuration.
  * The attacker may then access and modify Video Stream, Logs, Credentials, Configuration data.
  * Assets threatened directly: Credentials (Server), Logs, Video Stream, Configuration
* **FIRMWARE ABUSE**
  * An attacker installs a flawed version of the firmware and obtains partial or total control of the camera. The firmware may have been modified prior to the attack to include a malware or consist of an outdated version of the original firmware.
  * The attacker may for instance modify on the device the value of the firmware certificate used to authenticate the installed firmware or firmware updates.
  * Such an attack can allow for elevation of privileges, where a regular user gains access to admin privileges.
  * This attack can also be used to take control over the TOE( Target of Evaluation) resources, for instance to carry a denial-of-service attack on other network devices, to store illegal files or to mine cryptocurrencies.
  * Assets threatened directly: Firmware, Firmware Certificate, Computing Power, Network Bandwidth, Storage Space.
  * Assets threatened indirectly: All.
* **TAMPER**
  * An attacker tampers with the camera and tries to access or modify the media on which assets are stored.
  * This includes basic Printed Circuit Board (PCB) attacks, after opening the camera case, such as eavesdropping buses, disordering memory chips, use of debug interfaces.
  * Assets threatened directly: All.

Security Objectives:

* **Access control**
  * The TOE shall authenticate User before granting access to the Video Stream.
  * The TOE shall authenticate Admin before granting access the camera configuration and logs and before performing firmware update.
* **Secure storage**
  * The TOE shall protect integrity and confidentiality of Credentials when stored, and protect integrity of Firmware Certificate, Configuration and Logs when stored.
* **Firmware authenticity**
  * The TOE shall authenticate and verify integrity of firmware image during boot and of new firmware versions prior upgrade.
  * The TOE shall also reject attempts of firmware downgrade.
* **Communication**
  * The TOE shall be able to authenticate remote servers where Video Stream and Logs are uploaded and provide integrity and confidentiality protection for export outside of the TOE.
* **Audit(log)**
  * The TOE shall maintain log of all significant events and allow access and analysis of these logs to authorized users only.
* **Secure state**
  * The TOE shall maintain a secure state even in case of failures, for instance failure of verification of firmware integrity.

## Fundamentals of Cryptography

There are three classed of cryptography:

* symmetric cryptography
* public-key cryptography
* hybrid cryptography

### Symmetric cryptography

The data(assets) is encoded into a binary sequence, we use XOR operation, one-time pad or DES/AES to encrypt the plaintext into ciphertext.

### Public-key cryptography

RSA and ECC are the commonly used encryption algorithms. The theory of RSA:

Ciphertext = Plaintext ^ E mod N, the (E, N) is a pair of public keys
Plaintext = Ciphertext ^ D mode N, the (D, N) is a pair of private keys

How to calculate the keys:

1. prepare two prime number P and Q, $N = P * Q$
2. $L = lcm(P-1,Q-1)$, lcm means calculate  the least common multiple of two numbers
3. $1 < E < L, gcd( E, L) = 1$, gcd means calculate the greatest common divisor of two numbers
4. $1 < D < L, E * D mod L =1$

The problem of Public-key cryptography is Man-in-the-middle attack, it needs the public key authentication.

### Hybrid cryptography

It combines the advantages of symmetric and public-key cryptographies, and solute the problem of key distribution.

Other security technologies

* Hashes: The hashes are used to guarantee the integrity of data or software, it can defense the tamper attack.
* MAC(Message Authentication Code): the inputs of MAC are message and shared key, the output of MAC is a fixed length number calculated by hashes. The MAC and defense the tamper and spoofing attack. But it cannot solve the Repudiation problem.
* Digital signature: the digital signature can defense the tamper, spoofing and repudiation attacks. It is realized by public-key cryptography. Alice use the private key to encrypt and Bob use the public key to decrypt so Alice cannot repudiate the message.
* PKC, public-key certificate: It is used to solve the public-key authentication problem.

## Security Requirements

Example: Security requirements of Network Camera

* **ACCESS CONTROL**
  * The TOE shall maintain the roles Admin and User.
  * The TOE shall allow authentication of users according to these roles through user-initiated interactive sessions.
  * Note 1: The ST writer shall explicit how credentials for user authentication are managed on the TOE. This may include techniques to prevent weak passwords and also to protect them against disclosure (for instance use of salted hashes).
  * The TOE shall manage a threshold for unsuccessful authentication attempts. The ST writer shall precise the actions taken is this threshold is reached.
  * The TOE shall require each user to be successfully authenticated before allowing any other actions on behalf of that user.
  * The TOE shall allow termination of user’s own interactive session and automatically terminate a remote interactive session after session inactivity.
  * The TOE shall enforce an access control policy on TOE assets and operations based on the identity of the user requesting access. The ST writer shall define rules of this policy.
  * Note 2: This policy will typically include rules such as:
    * Access to Configuration, Logs, Credentials assets is only allowed to authenticated users with role Admin.
    * Access to Firmware upgrade operations is only allowed to authenticated users with role Admin.
    * Access to video stream assets is only allowed to authenticated users with role User.
  * The TOE shall prevent unauthorized uses of all assets. In particular, the TOE shall prevent reading of all Credentials and shall not provide an interface to do so.
* **SECURE_STORAGE**
  * The TOE shall monitor for integrity errors assets with a security need for integrity (Camera ID, Firmware, Firmware Certificate, Logs, Configuration, Credentials).
  * Note 3: The TOE will typically ensure integrity either with hardware based write-once mechanisms, such as One-Time-Programmable (OTP), or through cryptographic hash functions. In the latter case, the ST writer shall explicit the cryptographic algorithms used for secure storage and related key characteristics and random generation methods.
  * Upon detection of a data integrity error, the TOE shall maintain a secure state. The ST writer shall specify reaction of the TOE in this case.
  * Note 4: For assets with a security need for confidentiality (Credentials), protection of relies on access control measures (OT.ACCESS_CONTROL). However the TOE may offer additional protection by encryption of persistent memory. The ST writer shall specify the mechanism used and related encryption techniques.
* **FIRMWARE_AUTHENTICITY**
  * The TOE shall rely on a secure boot mechanism to authenticate and verify integrity of firmware prior transferring control to the firmware.
  * Note 5: A secure boot will typically rely on a multi-stage boot process where the authenticity of the first stage is assumed from read-only memory and other stages with verification of cryptographic signatures with asymmetric keys. The ST writer shall explicit which signature schemes are used at the various stages, including the hash algorithm, and the size of the various parameters (e.g., modulus of 2048 bits and exponent of 32 bits for RSASSA-PSS with SHA-512). He shall also specify the list of standards that are met by the chosen schemes or none.
  * If the firmware is loaded from a removable media, the TOE shall use a persistent storage to store the version of the last installed firmware and compare this version to the version from the loaded firmware to prevent loading of an out-dated firmware.
  * Upon detection of a firmware authenticity error, the TOE shall maintain a secure state. The ST writer shall specify the action to be taken if the verification fails (cf. OT.SECURE_STATE).
  * Note 6: The TOE may enter a maintenance mode where the ability to return a secure state is provided.
  * On firmware upgrade requests, the TOE shall first authenticate the upgrade binary based on digital signature and verify its integrity. The TOE shall also check that version of the firmware for upgrade is more recent than the firmware currently installed.
  * Note 7: A Security Target derived from this TMSA must state which signature scheme is used.
  * Upon detection of an error during upgrade, the TOE shall revert to the version of the firmware prior the upgrade request.
  * The TOE should provide the ability to check availability of firmware upgrade and notify Admin.
* **COMMUNICATION**
  * The TOE shall establish a trusted communication channel with remote servers prior any exchange of Target of Evaluation Security Functionality (TSF) data or User data and verify if the peer certificate is valid.
  * Note 8: Trusted communication channels include any of Internet Protocol Security (IPsec), Transport Layer Security (TLS) or (Hyper Text Transfer Protocol Secure (HTTPS) performed by the TOE. Validity of the peer certificate is determined by the certificate path, the expiration date, and the revocation status.
  * The TOE shall prevent the disclosure and modification of user data when exporting user data outside of the TOE.
  * Note 9: Protection of user data relies on the encryption techniques provided with the trusted communication channel. The ST writer shall explicit which encryption algorithms are used and related key sizes.
  * The TOE should support network authentication (e.g. Wi-Fi or IEEE 802.1X).
* **AUDIT**
  * The TOE shall maintain an audit trail of security events. Each record shall mention the nature of the event, date and time of the event and the user, if any, responsible for the event.
  * Note 10: The ST writer shall explicit which events are logged. This will include at least failed and successful authentication attempts, firmware upgrade requests and progress, integrity errors.
  * The TOE shall prevent users from deleting entries from the audit trail.
  * Note 11: The only audit trail operations and interfaces that should be available on the TOE are appending a line to the audit trail and export outside of the TOE.
  * The TOE should rely on a Network Time Protocol (NTP) server to provide reliable source for time stamps for the audit trail.
  * Note 12: If the TOE supports the use of a NTP server, the operational guidance for the TOE shall provide instructions for the Admin to configure the NTP client on the TOE.
* **SECURE_STATE**
  * The TOE shall maintain a secure state in case of failures, such as firmware integrity error, firmware upgrade error, Random Number Generation (RNG) error, failure to establish a trusted communication channel.
  * Note 13: If the TOE should encounter a failure in the middle of a critical operation, the TOE should not just quit operating, leaving key material and user data unprotected. The ST writer shall specify
  * The TOE shall ensure residual information protection for credentials and session keys after they are being used.

Summary:

* Security Lifecycle: During its creation and use, the system progresses through a series of states. These states indicate the assets present in the system and the functionality that is available or has been disabled. Progression through the states is usually controlled using a write-once mechanism. The figure contains the minimum number of states and it is expected that a full product will contain more that are specific to a product family, OEM manufacturing, or market requirements.
* Boot ROM and reset
  * Boot keys: Root of Trust Public Key. To minimize the amount of required on-chip memory, an SoC should only store a cryptographic hash of the public key, while the public key is held in external storage. On each boot, the Boot ROM can then calculate the hash of the loaded public key and compare it with the hash in on-chip memory to ensure it has not been tampered.
  * Boot type: cold and warm boot.
  * Boot parameters: Some Boot ROM implementations can be influenced by additional configuration information stored in on-chip one-time programmable memory (OTP).
  * Boot robustness: The execution of the Boot ROM must not be perturbed by other agents (for example, through DMA) or via interfaces (for example, PCI, JTAG).
  * Secondary processing elements: If the SoC implements multiple processing elements (PE), the designated boot PE is called the primary. After the de-assertion of a reset, the primary boot PE executes the Boot ROM code, and the remaining PEs are held in reset or a safe platform-specific state until the primary boot PE initializes and boots them.
* Clock and power
* Memory system: the system sees the memory map as into two spaces, Secure and Non-secure storage, in which Trusted world assets are held in Secure storage and Non-Trusted world assets are held in Non-secure storage. Each transaction originates from either the Trusted world or Non-Trusted world. Before any shared storage can be reallocated from one security domain to an untrusted security domain, the asset must be securely removed. This process is called scrubbing. Scrubbing is the process of overwriting an asset using any of the following methods:
  * Overwritten with a pre-defined constant value (for example, zero).
  * Overwritten with a random value.
  * Indirectly changed to a random value, for example by changing a key which is used to decrypt the content.
* Processing elements
* Interrupts: Each world may receive interrupts of some form. An interrupt that is only meant to be received by the Trusted world is referred to as a Trusted interrupt. In most cases, a Trusted interrupt must not be visible to a Non-Trusted operation, in order to prevent information leaks that might be useful to an attacker. Consequently, the on-chip interrupt network must be able to route any interrupt to any world. However, the routing of Trusted interrupts must only be configured from the Trusted world. When a memory access violation occurs, such as when the Non-Trusted world tries to access a Trusted asset, a security exception or security interrupt is raised.
* Debug: DPM, debug protection mechanism
* Peripherals and subsystems
* Platform identity: Arm strongly recommends using public key cryptography, whereby the attestation key is a keypair consisting of a (secret) private key and a public key.
* Random number generation: true random number generator (TRNG.
* Timers: Trusted world depends on must be resistant against tampering and must only be configured by the Trusted world. Trusted timers are needed to provide time-based triggers to Trusted world services. The SoC must support one or more Trusted timers.
* Cryptography: The cryptographic algorithms that are used must be strong against networked adversaries and local attackers. The specific choice of algorithms depends on the target market and local regulations.
* Secure storage:
  * A hardware unique key (HUK)
  * An on-chip Trusted non-volatile counter, which is required for version control of firmware and trusted data held in external storage. An important property of these counters is that it must not be possible to roll them back, to prevent replay attacks. There must be at least one counter for Trusted firmware use and at least one counter for Non-trusted firmware use.
* Main memory: Example Secure RAM use cases are:
  * Secure boot code and data.
  * Secure Monitor code.
  * A Trusted OS or a Secure Partition Manager.
  * Cryptographic services.
  * Trusted services.


## Security Model

The goals of security model:

* Devices are uniquely identifiable.
* Devices support a security lifecycle.
* Devices are securely attestable.
* Devices ensure that only authorized software can be executed.
* Devices support secure update.
* Devices prevent unauthorized rollback of software.
* Devices support isolation.
* Devices support interaction over isolation boundaries.
* Devices support unique binding of sensitive data to a device.
* Devices support a minimal set of trusted services and cryptographic operations that are necessary for the device to support the other goals.

## Address Threats

Threat priority: Threat priority is indicated by the score calculated via Common Vulnerability Scoring System (CVSS) Version 3.1 [CVSS](https://www.first.org/cvss/calculator/3.1). The higher the threat scores, the greater severity the threat is with and the higher the priority is. CVSS scores can be mapped to qualitative severity ratings defined in CVSS 3.1 specification [CVSS_SPEC](https://www.first.org/cvss/calculator/3.1). This threat model follows the same mapping between CVSS scores and threat priority rating.

![fig1](/img/TF-M/tfm_1.png)

Data flow and threats:

* DF1: TF-M initializes NS entry and activates NSPE.
  * Tampering: The NS image can be tampered by an attacker.
  * Tampering: An attacker may replace the current NS image with an older version.
  * Tampering/Information disclosure: If SPE doesn’t complete isolation configuration before NSPE starts, NSPE can access secure regions which it is disallowed to.
  * Tampering/Information disclosure: If SPE doesn’t complete isolation configuration before NSPE starts, NSPE can control devices or peripherals which it is disallowed to.
  * Information disclosure: If SPE leaves some SPE information in non-secure memory or shared registers when NSPE starts, NSPE may access those SPE information.
  * Denial of service: An attacker may block NS to boot up
* DF2: NSPE requests TF-M RoT services.
  * Spoofing: A malicious NS application may pretend as a secure client to access secure data which NSPE must not directly access.
  * Tampering: An attacker in NSPE may tamper the service request input or output vectors between check and use (Time-Of-Check-to-Time-Of-Use (TOCTOU)).
  * Tampering: A malicious NS application may request to tamper data belonging to SPE.
  * Repudiation: A NS application may repudiate that it has requested services from a RoT service.
  * Information disclosure: A malicious NS application may request to read data belonging to SPE.
  * Tampering/Information disclose: A malicious NS application may request to control secure device and peripherals, on which it doesn’t have the permission.
  * Denial of service: A Malicious NS applications may frequently call secure services to block secure service requests from other NS applications.
  * Denial of service: A malicious NS application may provide invalid NS memory addresses as the addresses of input and output data in RoT service requests.
* DF3: Secure Partitions fetch input data from NS and write back output data to NS.
  * Tampering: An attacker may tamper NS input data while the RoT service is processing those data.
  * Tampering: A malicious NS application may embed secure memory addresses into a structure in RoT service request input vectors, to tamper secure memory which the NS application must not access.
  * Information disclosure: a malicious NS application can embed secure memory addresses in a parameter structure in RoT service request input vectors, to read secure data which the NS application must not access.
* DF4: TF-M returns RoT service results to NSPE after NS request to RoT service is completed.
  * Information disclosure: SPE may leave secure data in the registers not banked after the SPE completes PSA Client calls and executes BXNS to switch Armv8-M back to Non-secure state.
* DF5: Non-secure interrupts preempt SPE execution in single Armv8-M core scenarios.
  * Information disclosure: Shared registers may contain secure data when NS interrupts occur.
  * Denial of service: An attacker may trigger spurious NS interrupts frequently to block SPE execution.
* DF6: Secure interrupts preempt NSPE execution in single Armv8-M core scenarios.
  * Information disclosure: Shared registers may contain secure data when Armv8-M core switches back to Non-secure state on Secure interrupt return.
* Other miscellaneous threats
  * Elevation of privilege: Armv8-M processor Secure software Stack Sealing vulnerability.
  * Denial of service/Tampering: Invoking Secure functions from handler mode may cause TF-M IPC model to behave unexpectedly.

How does TF-M defense or mitigate threats: please ignore the table because it is the personal opinion.

|  | Spoofing  | Tampering |  Repudiation | Information Disclosure |Denial of Service  |  Elevation of Privilege |
|--|--|--|--|--|--|--|
|  **Isolation**| √  | √ |   |√  | √ |√  |
|**Anti-rollback/FWU**  | √  |  √|   | √ |  |  |
| **Secure boot** |  √ |  |   |  |  |  √|
| **Internal trusted storage** |   | √ |   | √ |  |√  |
|**Device binding**  | √  |√  |   |  |  |  |
|**Cryptographic service**  | √  |√  |   | √ |  |√  |
|**Initial attestation**  | √  | √ |  √ |  |  | √ |
|**Audit logging**  |   | √ |  √ |  |  |  |
|**Protected storage**  |   | √ |  √  | √  |  | √  |
|**PSA RoT**  | √   |  |   |  |  | √  |

## Some related knowledge of ARMv8-M

## FF-M and PSA interfaces

According to ARMv8-M document, the CPU has thread mode and handler mode. In handler mode, the processor has root privileges and accesses more system resources compared with thread mode. In thread mode, the processor can work with root privileges or none root privileges which is managed by nPRIV in CONTROL register. The handler mode can switch to the thread mode but the reverse process cannot complete by itself. If we want to turn the thread mode to handler mode, we need to put a processor exception.When the processor is reset, the CONTROL is cleared so it is under privileged thread mode. Most time the applications work under the 'user mode' (unprivileged thread mode) which is similar to Linux operating system.

![fig2](/img/TF-M/tfm_2.png)
The psa interface code shows:

```c
__attribute__((naked))
psa_handle_t psa_connect(uint32_t sid, uint32_t version)
{
    __ASM volatile("SVC %0           \n"
                   "BX LR            \n"
                   : : "I" (TFM_SVC_PSA_CONNECT));
}
```

Here is a flow chart to describe the psa interfaces in IPC mode:
![fig3](/img/TF-M/tfm_3.png)

The interfaces in library mode:
![fig4](/img/TF-M/tfm_4.png)
