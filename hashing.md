# Hashing
A hash function is any function that can be used to map data of arbitrary size to data of fixed size. The values returned by a hash function are called hash values, hash codes, digests, or simply hashes. One use is a data structure called a hash table, widely used in computer software for rapid data lookup. Hash functions accelerate table or database lookup by detecting duplicated records in a large file.

 A cryptographic hash function allows one to easily verify that some input data maps to a given hash value, but if the input data is unknown, it is deliberately difficult to reconstruct it (or equivalent alternatives) by knowing the stored hash value. This is used for assuring integrity of transmitted data, and is the building block for HMACs, which provide message authentication.


 The ideal cryptographic hash function has five main properties:

 - it is deterministic so the same message always results in the same hash
 - it is quick to compute the hash value for any given message
 - it is infeasible to generate a message from its hash value except by trying all possible messages
 - a small change to a message should change the hash value so extensively that the new hash value appears uncorrelated with the old hash value
 - it is infeasible to find two different messages with the same hash value


## Applications

 - Verifying the integrity of files or messages
 - Password verification
 - Proof of work
 - File or data identifier


### Types of general purpose hashing
 - MD5 (broken)
 - SHA1 (weak)
 - SHA256 (Ok)

### Types of password hashing
 - PBKDF
 - BCRYPT (Ok)
 - SCRYPT (Good)
