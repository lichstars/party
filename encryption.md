
# Encryption
In cryptography, encryption is the process of encoding a message or information in such a way that only authorized parties can access it and those who are not authorized cannot.

### Properties
- **Authencity** - Are you communicating with the right party ?
- **Secrecy/Confidentiality** - Can anyone between site A and B be able to read the information sent between the two sites?
- **Integrity** - Is the information sent *accurate* and has not been changed while sending between A and B?

## Asymmetric key encryption
The key used to encrypt a message is different to the key used to decrypt the same message.

#### Elliptic Curve Cryptography (ECC)
Read a [relatively easy to understand primer here](https://blog.cloudflare.com/a-relatively-easy-to-understand-primer-on-elliptic-curve-cryptography/)

#### Rivest–Shamir–Adleman (RSA)
RSA (Rivest–Shamir–Adleman) is one of the first public-key cryptosystems and is widely used for secure data transmission. In such a cryptosystem, the encryption key is public and it is different from the decryption key which is kept secret (private).

 In RSA, this asymmetry is based on the practical difficulty of the factorization of the product of two large prime numbers.

 A user of RSA creates and then publishes a public key based on two large prime numbers. The prime numbers must be kept secret. Anyone can use the public key to encrypt a message and only someone with knowledge of the prime numbers (used to generate a private key) can decode the message feasibly.

 RSA is a relatively slow algorithm, and because of this, it is less commonly used to directly encrypt user data. More often, RSA passes encrypted shared keys for symmetric key cryptography which in turn can perform bulk encryption-decryption operations at much higher speed.

- Disadvantages:
  - Limited data can be encrypted
  - Slow, Hard to implement
  - If two large prime numbers are used and one of them accidentally is not a prime number, you'll get more than one (depending on number of factors) private key, which weakens the key.
- Advantages:
  - Solves key sharing problem in symmetric key encryption
  - can be used in conjunction with symmetric key encryption to speed encryption up


Further reading:
- **Trap door functions**: A trapdoor function is a function that is easy to compute in one direction, yet difficult to compute in the opposite direction (finding its inverse) without special information, called the "trapdoor". Trapdoor functions are widely used in cryptography.
- **Prime numbers** Prime numbers are such that they divide only by themselves and 1. For example: 2,3,5,7,11,13,17,19,23...
- **Co-prime** : In number theory, two integers a and b are said to be relatively prime, mutually prime, or co-prime if the only positive integer (factor) that divides both of them is 1. For example 4 and 5 or 39 and 8.
- **Congruence**: Exactly equal in size and shape. Congruent sides or segments have the exact same length. Congruent angles have the exact same measure. For any set of congruent geometric figures, corresponding sides, angles, faces, etc. are congruent. Two integers are **congruent mod n** if they both have the same remainder when divided by n.

  For example
  ```
  a mod n == b mod n
    (r)   ==  (r)      => they both have the same remainder, therefore are congruent
  ```

Practical example

1. PersonA picks two very large prime numbers and multiples them
2. Using this value they do a series of math calculations (see image below using smaller prime numbers)

  ![image](https://github.com/lichstars/party/blob/master/resources/encryption/rsa-maths.png?raw=true")

3. Using the values from the maths calculations, we can now derive the private and public keys. PersonA at this point would send their public key to PersonB to encrypt a message. The encrypted message is returned to PersonA and PersonA would use the private key to decrypt the message.

  ![image](https://github.com/lichstars/party/blob/master/resources/encryption/rsa-encrypt-decrypt.png?raw=true")
  
4. Only the private key can decrypt the message. This solves the key sharing problem that is present in symmetric key encryption.


## Symmetric key encryption
Symmetric-key algorithms are algorithms for cryptography that use the same cryptographic keys for both encryption of plaintext and decryption of ciphertext. This requirement that both parties have access to the secret key is one of the main drawbacks of symmetric key encryption, in comparison to public-key encryption (also known as asymmetric key encryption)

#### Considerations
  - **Key sharing problem**
This problem exists because we need to send both the ciphertext and the key that was used to encrpyt it to the recipient so that they
are able to decrypt the message.

#### Key derivation function
An encryption algorithm can contain a Key derivation function (KDF). KDF's derive one or more secret keys from a secret value. KDFs can be used to stretch keys into longer keys or to obtain keys of a required format. Key derivation function's are generally made deliberately slower so as to frustrate brute-force attack or dictionary attack on the password or passphrase input value. [Wiki on KDF](https://en.wikipedia.org/wiki/Key_derivation_function)

#### Diffie-Hellman key exchange
Diffie–Hellman key exchange (DH) is a method of securely exchanging cryptographic keys over a public channel and was one of the first public-key protocols as originally conceptualized by Ralph Merkle and named after Whitfield Diffie and Martin Hellman. It is primarily used as a method of exchanging cryptography keys for use in symmetric encryption algorithms like AES. The shared secret key can then be used to plug into the KDF used in symmetric key encryption.

![image](https://github.com/lichstars/party/blob/master/resources/encryption/diffie-hellman.png?raw=true")


### Modes
#### Stream
A typical stream cipher encrypts plaintext one byte at a time. A stream cipher is a symmetric key cipher where plaintext
digits are combined with a pseudorandom cipher digit stream.
In practice, a digit is typically a bit and the combining operation an exclusive-or (XOR). This encryption method can be computationally slow compared to block encryption.

#### Block
Block ciphers take a number of bits and encrypt them as a single unit, padding the plaintext so that it is a multiple of the block size. Blocks of 32, 64 and 128 bits are commonly used to match with hardware.

  **Types**
  - Electronic Code Book (ECB)

  The simplest of the encryption modes is the Electronic Codebook (ECB) mode. The message is divided into blocks, and each block is encrypted separately. The disadvantage of this method is a lack of diffusion. Because ECB encrypts identical plaintext blocks into identical ciphertext blocks, it does not hide data patterns well. In some senses, it doesn't provide serious message confidentiality, and it is not recommended for use in cryptographic protocols at all.

  More on the [wiki](https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation#CBC)
  ![image](https://github.com/lichstars/party/blob/master/resources/encryption/ecb-encrypt-example.png?raw=true")

  - CTR (Counter)

  Counter mode turns a block cipher into a stream cipher. It generates the next keystream block by encrypting successive values of a "counter". The counter can be any function which produces a sequence which is guaranteed not to repeat for a long time, although an actual increment-by-one counter is the simplest and most popular.

  More on the [wiki](https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation#CBC)

  - CBC (Cipher block-chaining)

  In CBC mode, each block of plaintext is XORed with the previous ciphertext block before being encrypted. This way, each ciphertext block depends on all plaintext blocks processed up to that point. To make each message unique, an initialization vector must be used in the first block.

  More on the [wiki](https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation#CBC)

  Encrypting example

  ![image](https://github.com/lichstars/party/blob/master/resources/encryption/cbc-encrypt-example.png?raw=true)

  Decrypting example

  ![image](https://github.com/lichstars/party/blob/master/resources/encryption/cbc-decrypt-example.png?raw=true)

  CBC has been the most commonly used mode of operation. Its main drawbacks are that encryption is sequential (i.e., it cannot be parallelized), and that the message must be padded to a multiple of the cipher block size.

  - GCM
