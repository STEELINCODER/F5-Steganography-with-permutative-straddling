<img width="1501" height="711" alt="image" src="https://github.com/user-attachments/assets/045e1625-23ea-472d-a526-6c6f7c2ab706" /># F5-Steganography with permutative straddling

# What are we trying to do here?

In a nutshell we are trying to hide text messages in an image 

<img width="1501" height="711" alt="image" src="https://github.com/user-attachments/assets/bb01ac05-56a2-4b04-809d-288c2ede7692" />


but we are not directly manipulating pixel values like that of Least Significant Bit, In a standard 24-bit color image, each pixel has three color channels they are:

Red, Green, and Blue. Each channel uses 8 bits or 1 byte with values from 0 to 255. LSB steganography replaces the last/rightmost bit of these color values with bits from a secret message. Because changing the last bit alters the color intensity by only 1 unit, the human eye cannot see the difference.

You can easily find stuff out using advanced steganalysis tools and it is kinda outdated, Chi square attacks and stuff measures the entropy, where the number of 0s and 1s of the LSB be statistically same.

# So how does F5 steganography help us?

F5 steganography algorithm hides secret data in JPEG images by modifying transform coefficients rather than raw spatial pixels. It uses matrix encoding to reduce the number of required changes and permutation straddling to scatter modifications randomly, making the hidden data hard to detect.

This is the tech part but listen an image has two parts, luminance and chrominance, lumina - brightness, chrominance - colors of an image. Human eye is more receptive to brightness rather than color due to the fact that we have more rod cells than cone cells. F5 follows the pipeline of JPEG compression algorithm which hides data in the frequency co-efficients. 

Watch this wonderful video by Branch Education explaining JPEG compression.

https://youtu.be/Kv1Hiv3ox8I?si=N3Wg2mfHnqH6WNwt

<img width="640" height="355" alt="image" src="https://github.com/user-attachments/assets/6d1be88b-defc-4312-ad70-14255b020f7a" />


## Philosophy and Full Disclosure
*"Anyone can design a cipher that they themselves cannot break." — Schneier's Law*

I have created this repository to provide a PoC for a steganographic medium of transport. Too often, steganography tools rely on "security by obscurity" or weak "kid-sister" cryptography. They use predictable random number generators, fail to authenticate their payloads, or use deprecated ciphers. 

This code here provides 3 different algorithms and benchmarks them. I have created this code after multiple iterations of trial and error and after going through open source code and building upon previous work.

This project was built on the principle that true security requires full disclosure. The source code is public so that the cryptographic and steganalysis communities can audit, attack, and improve it. 

Also I am welcome to any new ideas and am willing to learn new things so anything is appreciated.

## How It Works (The pipeline)??

This system separates the security of the payload from the stealth of the carrier. It operates in two phases:

### 1. Cryptographic Payload Encapsulation
Before embedding, the payload is secured using modern cryptographic primitives. 

*   **Core Transport:** AES-256 in Galois/Counter Mode (GCM) for Authenticated Encryption. This ensures that the payload cannot be read or tampered with.
*   **Key Wrapping:** The symmetric AES key is encapsulated using one of three routing options which you can choose from:
    *   **RSA-4096** (OAEP padding)
    *   **ECC SECP384R1** (ECDH Key Exchange)
    *   **Symmetric PBKDF2** (400,000 iterations for key derivation)

### 2. CSPRNG-Driven Embedding (F5 Algorithm)

The encrypted, high-entropy payload is embedded into the Discrete Cosine Transform (DCT) coefficients of a carrier JPEG.
*   **Cryptographic Permutation:** We do not use statistical PRNGs (like the Mersenne Twister). The embedding path is determined by a Fisher-Yates shuffle driven by an AES-CTR keystream. 
*   **F5 Shrinkage Mitigation:** The algorithm dynamically adjusts coefficients to avoid flattening the natural statistical distribution of the image matrix, a common flaw in naive F5 implementations.

## The Threat Model 

Do not conflate visual stealth with statistical stealth. 

*   **What this protects against??:** Casual observation, forensic diff-mapping, password-guessing, and tampering. The encryption will resist state-level adversaries.
*   **What this is vulnerable to:** Advanced statistical steganalysis. Injecting perfectly random data (encrypted ciphertext) into a natural image inherently alters its frequency domain. While the spatial footprint is randomized, a dedicated intelligence agency analyzing the DCT coefficient histograms may flag the image as an anomaly. 

<img width="2190" height="964" alt="__results___3_1" src="https://github.com/user-attachments/assets/1d6408a2-05b0-473c-a621-b26df1e7dbf6" />

## Usage Requirements

*   Python 3.8+
*   `cryptography` (for all cryptographic primitives)
*   `opencv-python` (cv2)
*   `numpy`
*   `Pillow` (PIL)
*   `reedsolo` (Reed-Solomon error correction)

## Disclaimer
This software is provided for educational and research purposes. Digital Rights Management (DRM) and closed-source security models fail because they prioritize vendor control over user security. By using this tool, you own the code and the keys. However, do not use this for life-and-death operational security without subjecting it to rigorous, independent peer review.
