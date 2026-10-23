# Huffman Compression + AES-256-GCM Encryption

A project that **compresses** text with canonical Huffman coding and then **encrypts** it with a password-derived AES-256-GCM key. The receiver uses the same password to decrypt and recover the original text.

- **Sender:** Text → Encrypted data
- **Receiver:** Encrypted data → Text

## Why this project?

| Goal | Technique | Benefit |
|------|-----------|---------|
| Make data smaller | Canonical Huffman coding | Less data to store or send |
| Keep data secret | AES-256-GCM | Only the password holder can read it |
| Detect tampering | GCM authentication tag | Wrong password or changed data is rejected |

## Key terms

- **Huffman coding:** gives short bit-codes to frequent characters and long codes to rare ones, so the total size shrinks.
- **Canonical Huffman:** a Huffman variant where codes are rebuilt from code lengths alone, so the tree never needs to be sent.
- **UTF-8:** a standard that stores every character as 1 to 4 bytes.
- **Salt:** random data added to a password before hashing.
- **Nonce:** a number used once, unique for every message.
- **AES:** a symmetric cipher, meaning the same key encrypts and decrypts.
- **Authentication tag:** a checksum over the data and the secret key, used to detect changes.

## How it works

### Encode

1. **Enter text and password.** These are the two inputs.
2. **Convert text to UTF-8 bytes.** This lets the program handle emojis and any language, not only plain English.
3. **Apply canonical Huffman coding.** Normal Huffman needs the whole tree to decode, and sending the tree is large and risky. Canonical Huffman sends only each byte's code length, and the receiver rebuilds the exact same codes.
4. **Build the payload.** The data to be encrypted has three parts:
   - Original text length (4 bytes)
   - Code-length table (256 bytes, one entry per possible byte value)
   - Packed Huffman bits
5. **Derive the key from the password.** A random salt (16 bytes) is added to the password and passed through a hashing function. The result is a 32-byte key, so there are 2²⁵⁶ possible keys and guessing is practically impossible. The salt stops identical passwords from producing the same key and blocks lookup-table attacks.
6. **Encrypt with AES-256-GCM.**

   | | Item | Size |
   |---|------|------|
   | Input | Key | 32 bytes |
   | Input | Nonce | 12 bytes |
   | Input | Payload | variable |
   | Output | Ciphertext | same length as payload |
   | Output | Authentication tag | 16 bytes |

   AES scrambles data using byte substitution (SubBytes), row shifting, column mixing and XOR with the key. Every step is mathematically reversible.

The app also displays the Huffman tree and the generated codes.

### Decode

1. Take the encrypted text and the password.
2. Split out the salt, nonce and ciphertext, then re-derive the key and decrypt with AES-256-GCM.
3. If the password is wrong or the data was changed, the authentication tag will not match and nothing is returned.
4. If the tag matches, rebuild the Huffman codes from the code-length table and show the original text.

## Canonical Huffman example

Suppose the code lengths are: A = 2 bits, B = 1 bit, C = 3 bits, D = 3 bits.

1. **Sort** by length, then alphabetically: `B, A, C, D`
2. **Assign codes.** Start at `0`. For each next symbol, add 1, then shift left when the length grows.

| Symbol | Length | Code |
|--------|--------|------|
| B | 1 | `0` |
| A | 2 | `10` |
| C | 3 | `110` |
| D | 3 | `111` |

Both sides follow the same rule, so only the lengths need to be shared.

## Security notes

- The key is never stored or sent. It is derived from the password and the salt.
- A new salt and a unique nonce are used for every message.
- GCM gives both **secrecy** and **tamper detection**.

## Summary

> Compress with canonical Huffman so no tree is sent, then lock the result with a password-derived AES-256-GCM key.

## Getting started

<!-- Add your language, dependencies, install steps and run commands here. -->

## License

<!-- Add your license here. -->
