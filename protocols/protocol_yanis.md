# Broadcast Protocol

## Objective

Players belonging to the same team can only communicate through the Zappy server using the `Broadcast` command.

To coordinate level-ups and resource management efficiently, all messages follow a compact and encrypted protocol.

---

# Message Format

Before encryption, all messages follow the format:

```text
TeamName,Type,Param1,Param2,...
```

## Fields

| Field    | Description              |
| -------- | ------------------------ |
| TeamName | Team identifier          |
| Type     | Message type             |
| ParamX   | Type-specific parameters |

---

# Encryption

Before being sent, a message is serialized:

```text
TeamName,Type,Param1,...
```

and encrypted using the team's secret key.

The resulting payload is sent through:

```text
Broadcast <EncryptedPayload>
```

Upon reception:

1. Decrypt the payload.
2. Verify the team name.
3. Parse the message.
4. Execute the associated action.

Messages originating from other teams are ignored.

---
### Encryption Protocol Specification (Key Generation)

To guarantee the authenticity of `broadcast` messages and prevent identity theft by opposing teams, all communications are encrypted using a Pre-Shared Key (PSK). 


#### Prerequisites
* The `crypto.txt` file located at the root of the project (encoded in **UTF-8** with **UNIX LF `\n`** line endings).
* The static masking key: `ZAPPY2026`

#### Step-by-Step Reconstruction Procedure

* **Step 1: Root Wallet Identification**
  Analyze the `crypto.txt` file, which models a directed acyclic graph (DAG) of financial transactions. Identify the unique wallet address that emits funds but never receives any (the orphan source node of the graph).

* **Step 2: Endianness Inversion**
  Strictly reverse the character order of the string obtained in Step 1.

* **Step 3: Physical File Footprint**
  Calculate the raw **MD5** hash of the entire content of the `crypto.txt` file.
  *Warning:* Any modification of a single space or line ending (such as automatic Windows CRLF conversion during a copy-paste) will corrupt this checksum.

* **Step 4: Multi-Factor XOR Masking**
  Take the reversed string (Step 2) and apply a bitwise character-by-character `XOR` operation with both the static key `ZAPPY2026` **AND** the file's MD5 hash (Step 3). For keys longer than the reversed string, use a modulo operator (`i % key_length`).
  *Expected Format:* The result must be encoded as a raw hexadecimal string (lowercase, 2 characters per byte).

* **Step 5: Proof of Work**
  Take the raw hexadecimal string resulting from the XOR operation (Step 4) and compute its **SHA-256** hash. Repeat the hashing operation on the resulting output exactly **5,000,000 times**.

#### Final Key
The final 64-character hexadecimal string obtained at the end of the 5,000,000th iteration of Step 5 constitutes the **unique encryption key** hardcoded into our AI to validate and decode network packets.

---

# Message Types

## GATHER

Used to announce the preparation of a level-up.

The message contains all information required for cooperation:

* Target level
* Missing players
* Missing resources

### Format

```text
TeamName,GATHER,Level,Players,Linemate,Deraumere,Sibur,Mendiane,Phiras,Thystame
```

### Parameters

| Parameter | Description                           |
| --------- | ------------------------------------- |
| Level     | Target level                          |
| Players   | Number of additional players required |
| Linemate  | Missing linemates                     |
| Deraumere | Missing deraumeres                    |
| Sibur     | Missing siburs                        |
| Mendiane  | Missing mendianes                     |
| Phiras    | Missing phiras                        |
| Thystame  | Missing thystames                     |

### Example

```text
ZAP,GATHER,4,2,0,1,0,0,1,0
```

Meaning:

```text
Level 4 elevation is being prepared.

Still required:
- 2 players
- 1 deraumere
- 1 phiras
```

### Usage

When receiving a GATHER message:

* Available players may move toward the broadcaster.
* Collectors may prioritize the missing resources.
* Lower-priority gathering requests may be ignored.

---

## FOUND

Used to announce the discovery of a rare or strategically important resource.

This message should only be emitted for resources whose value justifies the communication cost.

Typical examples:

* Thystame
* A large concentration of a required resource
* A resource currently blocking an elevation

### Format

```text
TeamName,FOUND,Resource,Amount
```

### Parameters

| Parameter | Description         |
| --------- | ------------------- |
| Resource  | Resource type       |
| Amount    | Quantity discovered |

### Example

```text
ZAP,FOUND,THYSTAME,2
```

Meaning:

```text
A player has discovered 2 thystames.
```

### Usage

When receiving a FOUND message:

* Collectors may prioritize the announced resource.
* Players involved in a future elevation may adjust their objectives.
* The message may be ignored if the resource is not currently required.

### Notes

To limit communication costs, this message should not be used for:

* Common food discoveries
* Single low-value resources
* Resources that are not currently useful to the team

FOUND should remain reserved for rare or high-impact discoveries.

---

## SUCCESS

Used to announce a successful level-up.

### Format

```text
TeamName,SUCCESS,Level
```

### Example

```text
ZAP,SUCCESS,4
```

Meaning:

```text
A level 4 elevation has been successfully completed.
```

### Usage

Allows all players to update their strategic objectives.

---

## FORK

Used to announce the creation of an egg.

### Format

```text
TeamName,FORK
```

### Example

```text
ZAP,FORK
```

### Usage

Allows the team to know that a future player slot is available.

---

# Summary Table

| Type    | Parameters                                                             | Purpose                               |
| ------- | ---------------------------------------------------------------------- | ------------------------------------- |
| GATHER  | Level, Players, Linemate, Deraumere, Sibur, Mendiane, Phiras, Thystame | Coordinate a level-up                 |
| FOUND   | Resource, Amount                                                       | Announce a rare or strategic resource |
| SUCCESS | Level                                                                  | Notify successful level-up            |
| FORK    | None                                                                   | Notify egg creation                   |

---

# Protocol Philosophy

This protocol is based on the following principles:

1. Minimize the number of broadcasts.
2. Avoid redundant messages.
3. Exchange only strategic information.
4. Keep all level-up related information inside a single GATHER message.
5. Let players remain autonomous for exploration and routine resource collection.

Broadcasts are therefore primarily used for:

* Coordinating level-ups
* Announcing rare resource discoveries
* Announcing strategic progress
* Informing the team about egg creation

Ordinary resource discoveries are intentionally not broadcast to avoid unnecessary communication costs.

---
