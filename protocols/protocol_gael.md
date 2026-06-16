# Documentation

> [!CAUTION]
> The 'positive infinite' will be capped at `14603` (`-1.460354508809587...`)
>
> The `FLOAT_PRECISION` constant used will be defined with `1e-(7 - 1)`, same as the float32 precision - 1

### Table of Contents
 - [2ETP-2C Protocol (v1.0.0)](#2etp-2c-protocol-v100)
 - [WTMH Encoding (v2.0.0)](#wtmh-encoding-v200)
 - [c2dmp-hsm](#c2dmp-hsm-edited-version)

## 2ETP-2C Protocol (v1.0.0)
> **E**ncrypted & **E**ncoded, **T**yped **T**ransfer **P**rotocol - **C**ivilization **C**ommunication

2ETP-2C is an civilization communication protocol designed for 'structured' data exchange over a 'reliable' transport.

It provides:
 - Obfuscation (using [WTMH Encoding](#wtmh-encoding-v200))
 - Encoding (Base64 framing)
 - Origin minimal check (using [c2dmp-hsm](#c2dmp-hsm-edited-version) & [Zeta Function](#zeta-function), see [here](#origin-check))
 - Typed messages (data, events, commands, ...)
 - Framed transmission using control characters (ENQ, ETB, EOT, ...)

### Payload

Each transmission MUST follow this structure:
```
MAGIC ENQ <payload> [<res> <branch_miss_rate> <loop_count> <s1> <s2>] EOT
```
 - MAGIC (`0x22`) = Magic byte (help to identify the transmition)
 - ENQ (`0x05`) = Start of transmission
 - EOT (`0x04`) = End of transmission
 - payload = message content (obfuscated with WTMH v2.0.0 and after encoded in Base64)
 - res (4 bytes/float 32) = result of the origin check
 - branch_miss_rate (4 bytes/float 32) = average of the branch miss rate of the c2dmp calls during the origin check
 - loop_count (2 bytes) = count of loop executed for the result
 - s1 = first string used for the origin check
 - s2 = second string used for the origin check

> [!NOTE]
> The part `[<res> <branch_miss_rate> <loop_count> <s1> <s2>]` is comprise in the payload and allways defined and obfuscated since it's in the payload

The first byte of the payload define it's type such as:
| Byte (hexa)  | Type                                |
| ------------ | ----------------------------------- |
| ESC (`0x1B`) | [Data](#data)                       |
| BEL (`0x07`) | [Request](#request)                 |

### Data
> Tell the world that Armel was stolen (at least 1 material was picked up)
```
ESC "I just stole Armel" <material> <quantity>
```
 - ESC (`0x1B`) = payload type byte (Data)
 - ETB (`0x17`) = Separator
 - material (1 byte) = id of the material (from 0 'food' to 6 'thystame')
 - quantity (2 bytes) = quantity of the material

> Tell the world Gaël lost his cacolac (at least 1 material was droped)
```
ESC "Gael just lost (the Game) his cacolac" <material> <quantity>
```
 - ESC (`0x1B`) = payload type byte (Data)
 - ETB (`0x17`) = Separator
 - material (1 byte) = id of the material (from 0 'food' to 6 'thystame')
 - quantity (2 bytes) = quantity of the material

### Request
> The avengers (Armel) are making a national rally for the second round (I want to start an incantation of level x)
```
BEL "Avengers Assemble!!!" <uuid> ETB <level>
```
 - BEL (`0x07`) = payload type byte (Event)
 - ETB (`0x17`) = Separator
 - uuid = uuid of the sender (using timestamp with nano seconds)
 - level (1 byte) = level of the incantation

## Origin Check
> using: \<res> \<branch_miss_rate> \<loop_count> \<s1> \<s2>

> [!NOTE]
> The internal loop_count (used as a sender) is computed one time at the start from `(execution time of 1f) / ((execution time of a lambda origin check with loop_count as Z) / Z) * 2.5`
>
> Where `Z` is the result of the aproximation of the [zeta analytic continuation](zeta-function) with s equal the order/real part of the non trivial zeros solution, the result multiplied by -250

> [!WARNING]
> Any metric that dosen't respect the error range will result in an ignored payload

| Metric | Error Range |
| ------ | ----------- |
| `res`  | ±0.001% |
| `branch_miss_rate` | ±0.025% |
| `loop_count` | ±15% |

> [!CAUTION]
> If your algorithym is poorly optimized you might just kill your client due to computation time

> [!NOTE]
> ps: That exactly what i want :)

The s1 and s2 string are randomly generated at each transmition, the same pair of s1 and s2 can't be seen 2 time or the payload will be ignored.
The randomly generated string are always 404 bytes long.

> You will need to repeat the steps `loop_count` times

| Step n° | Resume |
| ------- | ------ |
| 0       | `res = c2dmp(s1, s2)` |
| 1       | `s1 += str(round(res, 2))` |
> round(x, 2) -> keep 2 digit after the ','

### Zeta Function

> [!CAUTION]
> Don't use the wrong analytic continuation of the Riemann's zeta function it's like an infinite continuous jigsaw puzzle
>
> We will be using the one defined [here](https://en.wikipedia.org/wiki/Riemann_zeta_function) at the `Riemann's functional equation` article

- **Analytic Continuation**
```math
\zeta(s)=2^{s}\pi^{s-1}\sin\left(\frac{\pi s}{2}\right)\Gamma(1-s)\zeta(1-s)
```

- **Gamma**
```math
\Gamma(z)=\int_{0}^{\infty} t^{z-1} e^{-t}
```

## WTMH Encoding (v2.0.0)
> **W**elcome **T**o **M**y **H**ead

> [!WARNING]
> This encoding add bytes at the end & start of the bytes given at input [here](#extra-bytes)

> [!NOTE]
> The size of the real payload isn't affected by the encoding

### Table of Contents (WTMH)
 - [Extra Bytes](#extra-bytes)
 - [Encrypting](#encrypting)
 - [Matrix Computing](#matrix-computing)

### Steps (preview)

> [!CAUTION]
> During the use of the c2dmp-hsm the `prefixDepthSearch` will be defined at `5` and the `UINTN` at `uint32` during this whole process 
> 
> When bytes are splited in half, ignore the last byte in the final count if the number of bytes is odd
>
> Only bytes from the payload are encoded and dosen't affect the extra bytes as we speak of bytes for the key or any other things

| Step n° | Resume |
| ------- | ------ |
| 0       | encoding base64 |
| 1       | bits invertion |
| 2       | bits circular right shift (for each bits) |
| 3       | encrypting first half, second half used has the key |
| 4       | encrypting second half, first half used has the key |
| 5       | bits circular left shift (for each bits) |
| 6       | encrypting first half, second half used has the key |
| 7       | encrypting second half, first half used has the key |
| 8       | bits circular right shift (for each bits) |
| 9       | bits invertion |
| 10      | apply matrix computing |
| 11      | bits invertion |
| 12      | encoding base64 |
| 13      | add magic byte + borne (ENQ & EOT) |

### Extra Bytes

The bytes are placed in this order: `<size> <payload> <integrity>`

- **Size (2 bytes)**
```
Number of bytes of the initial input (exactly the same as the number of byte after transformation)
```

- **Integrity (floor(Size / 8) + 1 bytes)**
```
The integrity bits are boolean 0 (false) & 1 (true) used to know if during the matrix computing a number was modified (1) or not (0).
There are in order of the matrix, row by row from left to right.
The last byte could have more bits than needed to round the whole byte (can't cut it)
```

### Encrypting

> [!CAUTION]
> The number of bytes in the key should absolutly be even

- **Key**
```
const var FLOAT_PRECISION 1e-(7 - 1) // float32 precision - 1
Using the cd2mp-hsm algorythm we generate a float value from it:
 - s1 = first half bytes
 - s2 = second half bytes
You could apply this on s1 & s2 before using the c2dmp-hsm (in the given order): s1 ^= s2; s2 ^= s1; s1 ^= s2; s1 ^= s2; s2 ^= s1; s1 ^= s2;
The result is multiplied by: (floor(log(result)) x FLOAT_PRECISION)e-1 (to use the influence of the floating digits without getting into the unknow)
We take the integer part of it using a round function.
```

- **Encryption**
> Rust, well yes... not my problem :)
```cpp
#include <cstddef> // std::size_t
#include <cstdint> // std::uint8_t, std::uint32_t
#include <vector>  // std::vector

static inline nodiscard std::uint32_t lcg(std::uint32_t& state)
{return (state = 1664525 * state + 1013904223);}

nodiscard std::vector<std::uint8_t> crypt(const std::vector<std::uint8_t>& data, std::uint32_t key) {
    std::vector<std::uint8_t> encrypted = data;
    std::uint32_t state = key;
    for (std::size_t i = 0; i < out.size(); i++) {
        std::uint8_t keystream = lcg(state) & 0xFF;
        encrypted[i] ^= keystream;
    }
    return encrypted;
}
```
```py
#!/bin/python
from typing import ByteString

def lcg(state: int) -> int:
    return (1664525 * state + 1013904223) & 0xFFFFFFFF

def crypt(data: ByteString, key: int) -> bytes:
    encrypted = bytearray(data)
    state = key & 0xFFFFFFFF
    for i in range(len(out)):
        state = lcg(state)
        keystream = state & 0xFF
        encrypted[i] ^= keystream
    return bytes(encrypted)
```

### Matrix Computing

- **Building A WTMH Matrix**
```
Build the biggest squared matrix that you can with the bytes.
Add the leftover bytes to a new rows under the squared matrix (don't create new col).

exemple (bytes will be named from 1 to 18):
bytes = [1, ..., 18]

squared matrix = 
| 1| 2| 3| 4|
| 5| 6| 7| 8|
| 9|10|11|12|
|13|14|15|16|

matrix =
| 1| 2| 3| 4|
| 5| 6| 7| 8|
| 9|10|11|12|
|13|14|15|16|
|17|18|
```

- **Computing A WTMH Matrix**
```
There is no such ungraceful things that link a normal matrix computing to a perfect WTMH matrix computing!

First (prime circular array):
 - We will need an circular array with prime.
 - The array will contain prime from the n°6700 (included) to the n°7200 (included) prime (primes are around 6e4-7e4).

Second (multiplication, value are unsigned): -> col by col from left to right and from top to bottom
 - For each value, multiply the value by a prime and pass to the next prime for the next value
 - If a number can be multiplied without overflow then it's corresponding integrity bit is set to 1 or 0 in other case (should be at 0 by default)
```

## c2dmp-hsm (edited version)
> **C**ompute **D**istance using **D**ifferences, **M**isplaced characters and **P**refix depth – **H**euristic **S**tring **M**atching

> [!WARNING]
> As stated before the use of the c2dmp-hsm the `prefixDepthSearch` will be defined at `5` and the `UINTN` at `uint32`

### Parameters
- **s1** (source) = the first string from which characters are extracted
- **s2** (target) = the second string in which these characters are searched and compared
- **prefixDepthSearch** (default: `3`) = depth of the prefix search
- **UINTN** (default: `uint32_t`) = type used for the computation of values, which limits the maximum value

### Core Principles

- **Lookup Table (used for characters normalization)**
```cpp
std::array<unsigned char, 256> table{};

// Default char
for (std::size_t i = 0; i < 256; ++i)
    table[i] = static_cast<unsigned char>(i);

// To lower case
for (unsigned char x = 'A'; x <= 'Z'; ++x)
    table[x] = x + 32;

// Special char
table[0xE2]='a'; table[0xE4]='a'; table[0xE3]='a'; table[0xE5]='a';
table[0xC0]='a'; table[0xC1]='a'; table[0xC2]='a'; table[0xC4]='a'; table[0xC3]='a'; table[0xC5]='a';
table[0xE7]='c'; table[0xC7]='c';
table[0xE9]='e'; table[0xE8]='e'; table[0xEA]='e'; table[0xEB]='e';
table[0xC9]='e'; table[0xC8]='e'; table[0xCA]='e'; table[0xCB]='e';
table[0xEE]='i'; table[0xEF]='i'; table[0xED]='i'; table[0xEC]='i';
table[0xCE]='i'; table[0xCF]='i'; table[0xCD]='i'; table[0xCC]='i';
table[0xF1]='n'; table[0xD1]='n';
table[0xF4]='o'; table[0xF6]='o'; table[0xF2]='o'; table[0xF3]='o'; table[0xF5]='o';
table[0xD4]='o'; table[0xD6]='o'; table[0xD2]='o'; table[0xD3]='o'; table[0xD5]='o';
table[0xF9]='u'; table[0xFA]='u'; table[0xFB]='u'; table[0xFC]='u';
table[0xD9]='u'; table[0xDA]='u'; table[0xDB]='u'; table[0xDC]='u';
table[0xFD]='y'; table[0xFF]='y'; table[0xDD]='y';
```

- **Distance**
```
The distance is the final floating value that determines the similarity between two given strings.
This value is computed by adding costs (which can be positive or negative). The value starts at '0'.
(The lower the value the better. If it goes below 0 it should represent a good match.)
```

- **Differences**
```
When comparing 2 characters they are normalized (such as A → a, Â → a, é → e, ...).
By default when both characters are equal the cost is '0'.
Differences are evaluated at '+1' for the cost and correspond to the difference between two characters at the same index of the strings.
During a comparison there is a possibility for the cost to be '-0.5', if two characters are equal before normalization
and also different from their normalized version.
```

- **Misplaced characters**
```
During the comparison of the characters in parallel we count the number of misplaced characters.
Misplaced characters are those that aren't equal to their corresponding character at the same position in the second array
but have another matching character in the second array that is also not equal to its corresponding position.
The cost of a misplaced character is by default '1' but it is multiplied by a coefficient.
The coefficient uses the number of characters separating both strings abs(len(s1) - len(s2)) marked as 'diff'.
The floor limit is when 'diff' is 0 and the ceiling limit is when 'diff' is 10.
Limits: 1.01 <= coef <= 1.25
coef = 1.01 + (diff / 10) * 0.25;
coef = clamp(coef, 1.01, 1.25);
```

- **Prefix depth**
```
The given prefix depth is the number of potential prefixes tested, each starting one character later than the previous in s1.
The size of the biggest prefix found is used for the final distance value computation.
The cost of a character in a prefix is by default '1' but it is multiplied by a coefficient.
The coefficient uses the depth where the biggest prefix was found marked as 'prefix_depth',
and the biggest prefix marked as 'prefix_size'.
The coefficient is proportional to the percentage of the prefix size compared to the length of s1.
Limits: 0 <= upper_limit <= 2, 0 <= coef <= upper_limit 
upper_limit = 2
for i in prefix_depth: 
    upper_limit *= (1 - (i / prefixDepthSearch))
coef = (upper_limit * (prefix_size / len(s1))); 
```

### Output Exemples

> [!NOTE]
> As stated before the use of the exemple are computated with the `prefixDepthSearch` will defined at `5` and the `UINTN` at `uint32`

| s1 | s2 | result |
| -- | -- | ------ |
| "" | "" | `-nan` |
| "test" | "test" | `-8` |
| "I'm a invalid testing" | "Nop, not the same string!" | `10.57` |
| "This is the letter é" | "This is the letter é" | `-43.5` |
| "This is the letter é" | "This is the letter e" | `-32.881` |
| "This is the letter é" | "é erttel eht si sihT" | `-0.200001` |
| "é" | "é" | `-5` |
| "e" | "e" | `-2` |
| "é" | "e" | `2` |
| "abcdef" | "abcdef" | `-12` |
| "abcdef" | "ABCDEF" | `-12` |
| "Abcdef" | "Abcdef" | `-12.5` |
| "ABCDEF" | "ABCDEF" | `-15` |
| "command" | "command" | `-14` |
| "ocmmand" | "command" | `-0.248571` |
| "cmomand" | "command" | `-0.305714` |
| "ccommand" | "command" | `-6.375` |
| "comomand" | "command" | `-1.39` |
| "comand" | "command" | `-2.105` |

## Help
> [!NOTE]
> If you are reading this, congratulation
>
> You can come ask me question or comparaison with the original code (c++20) that use the c2dmp-hsm algorithm (only global question, algorithmique one or return values will be awnser)
>
> codeword: Banana

## Licence (c2dmp-hsm)

This algorithm is distributed under the CC BY-NC 4.0 license.

Any use outside of the Epitech project `Zappy` will requires permission from the original creator, `Tsukini`.

### Note from Tsukini
I do not allow any copy of this version to be distributed, except for the single version that will be publicly visible to other groups, and not beyond that.
