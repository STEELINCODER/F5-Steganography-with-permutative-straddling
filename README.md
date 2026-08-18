# F5-Steganography-with-permutative-straddling
# Cryptographically Secure Permutative Straddling (F5 Steganography)

## Philosophy and Full Disclosure
*"Anyone can design a cipher that they themselves cannot break." — Schneier's Law*

This repository contains a proof-of-concept for a hardened steganographic transport mechanism. Too often, steganography tools rely on "security by obscurity" or weak "kid-sister" cryptography. They use predictable random number generators, fail to authenticate their payloads, or use deprecated ciphers. 

This project was built on the principle that true security requires full disclosure. The source code is public so that the cryptographic and steganalysis communities can audit, attack, and improve it. 

Also I am welcome to any new ideas and am willing to learn new things so anything is appreciated.

## How It Works ??

This system separates the security of the payload from the stealth of the carrier. It operates in two phases:

### 1. Cryptographic Payload Encapsulation
Before embedding, the payload is secured using modern cryptographic primitives. 

*   **Core Transport:** AES-256 in Galois/Counter Mode (GCM) for Authenticated Encryption with Associated Data (AEAD). This ensures the payload cannot be read or tampered with.
*   **Key Wrapping:** The symmetric AES key is encapsulated using one of three routing options which you can choose from:
    *   **RSA-4096** (OAEP padding)
    *   **ECC SECP384R1** (ECDH Key Exchange)
    *   **Symmetric PBKDF2** (400,000 iterations for key derivation)

### 2. CSPRNG-Driven Embedding (F5 Algorithm)

The encrypted, high-entropy payload is embedded into the Discrete Cosine Transform (DCT) coefficients of a carrier JPEG.
*   **Cryptographic Permutation:** We do not use statistical PRNGs (like the Mersenne Twister). The embedding path is determined by a Fisher-Yates shuffle driven by an AES-CTR keystream. 
*   **F5 Shrinkage Mitigation:** The algorithm dynamically adjusts coefficients to avoid flattening the natural statistical distribution of the image matrix, a common flaw in naive F5 implementations.

## The Threat Model (A Reality Check)

Do not conflate visual stealth with statistical stealth. 

*   **What this protects against:** Casual observation, forensic diff-mapping, password-guessing, and tampering. The encryption will resist state-level adversaries.
*   **What this is vulnerable to:** Advanced statistical steganalysis. Injecting perfectly random data (encrypted ciphertext) into a natural image inherently alters its frequency domain. While the spatial footprint is randomized, a dedicated intelligence agency analyzing the DCT coefficient histograms may flag the image as an anomaly. 

## Usage Requirements

*   Python 3.8+
*   `cryptography` (for all cryptographic primitives)
*   `opencv-python` (cv2)
*   `numpy`
*   `Pillow` (PIL)
*   `reedsolo` (Reed-Solomon error correction)

## Disclaimer
This software is provided for educational and research purposes. Digital Rights Management (DRM) and closed-source security models fail because they prioritize vendor control over user security. By using this tool, you own the code and the keys. However, do not use this for life-and-death operational security without subjecting it to rigorous, independent peer review.
