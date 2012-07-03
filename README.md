taidaishar
==========

Taidaishar is an encryption algorithm and my first programming project in C++.
It's more of a personal project at the moment, but I welcome any comments or
suggestions, and will try to answer any questions you have if you ask.

It utilizes sha.h and randpool.h from the crypto++ library, and has a
deterministic random number generator (seeded by the key) and a nondeterministic
RNG (seeded by the state of the local machine. If i'm referring to the non-
deterministic generator I'll be clear to specify that. Otherwise I'm referring
to the deterministic one.


|||||   |||||   |||||   |||||   |||||   |||||   |||||   |||||   |||||   |||||
_________
|       |
[ SETUP ] Initializing the algorithm
|       |

01) The algorithm requires a 16 to 64-character password (128 to 512 bits)

02) This password is repeated to fill a 64-character array

03) Starting at a random slot in the array, each character is XORed with its
    own random number.

04) A 256-char, randomly sorted array is created to function as a substitution
    box.

___________
|         |
[ LOADING ] Loading the algorithm with data and setting up
|         |

01) User adds entropy to the nondeterministic RNG (not yet implemented).

02) User specifies whether to encrypt or decrypt the data (this determines the
    process sequence. This description will only detail the encryption sequence).

03) Data is fed into the algorithm in 32-byte chunks

04) The 32-byte data segment is cut into two 16-byte arrays (herein referred to
    as blocks).

05) Three new empty blocks are created and the first is filled with 16 non-
    deterministically random characters. The XOR of that block with the first of
    the two data blocks is stored into the middle of the 3 blocks, and the 2nd
    of the data blocks is XORed with the middle of the three, and the result is
    stored into the third data block. 

                   +---+---+                 +---+---+
                   | 1 | 2 |                 | 5 | 6 |
         xor-------+---+---+       xor-------+---+---+
          |        | 3 | 4 |        |        | 7 | 8 |
          |        +---+---+        |        +---+---+
          |            |            |            |
      +---+---+        |        +---+---+        |        +---+---+
      | 4 | 1 |        |        | 5 | 3 |        |        | 0 | 5 |
      +---+---+        +------->+---+---+        +------->+---+---+
      | 8 | 3 |                 | B | 0 |                 | C | 8 |
      +---+---+                 +---+---+                 +---+---+

06) The two old blocks (with the clear data) are discarded and the three new
    ones are XOR padded with a random byte stream.

___________
|         |
[ PROCESS ] Encryption of the 3 blocks (this process is repeated 8 times)
|         |

01) Each byte in the boxes is replaced using the substitution box.

02) The three boxes undergo a type of fiestel operation utilizing a generic
    hashing algorithm ( Taidaishar::fiesty_mod() ). The operation looks like
    this:

      Box 1         Box 2         Box 3
      --+--         --+--         --+--
        |             |             |
        +----hash->---------xor---->|
        |             |             |
        |<-xor<-hash--+             |
        |             |             |
        +--hash->xor->|             |
        |             |             |
        |             |<-xor<-hash--+
        |             |             |
        |             +--hash->xor->|
        |             |             |
        |<-----xor-------<---hash---+
        |             |             |
      --+--         --+--         --+--
      Box A         Box B         Box C

      Or:
          B3 ^= hash(B1); B1 ^= hash(B2); B2 ^= hash(B1);
          B2 ^= hash(B3); B3 ^= hash(B2); B1 ^= hash(B3);

03) The byte from a random slot in the first block is placed into a random
    slot in the second block. The byte that was is that slot is moved to a
    random slot in the third block. The byte that was in that slot is placed
    into the empty slot in the first block. This operation has three
    iterations.

04) One block is randomly selected to be XORed by the entire key ( a block is
    16 bytes and the key is 64 bytes: 4 pads).

|||||   |||||   |||||   |||||   |||||   |||||   |||||   |||||   |||||   |||||


THOUGHTS

  This process does unfortunately produce ciphertext that is 150% the size of
the plaintext, however with a dependable non-deterministic RNG I don't think
the extra size yields any extra information (each byte of plaintext is
transformed into a sequence-dependent 2-byte representation of the same data).

  I think the main issue is that the 48-byte chunks of ciphertext are
independent of one another. If chunk 32 is destroyed, the data in chunks 20
and 400 are still salvageable. 


end