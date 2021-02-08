# Open ZigBee Coordinator Backup Format
An ongoing specification for a JSON ZigBee coordinator backup format.

## Rationale
The primary use case for this specification is to provide a simple standard for exporting
ZigBee network information to disk. Standardizing this format will allowing users to
backup, restore, and migrate their networks between coordinator hardware and network
management software without having to needlessly rejoin all of their devices to a new network.

## Provisional Specification
JSON is both human-readable supported well by common programming languages, making it a
good fit for files that will be interpreted by both.


### Encoding Sequences of Bytes
JSON does not natively support sequences of bytes (`char[]`, `Buffer`, bytestrings, etc.), therefore an intermediate representation is necessary.

Sequences of bytes will be encoded as a case-insensitive string of each
byte's hex representation, with leading zeroes, concatenated together. For example,
`\x0A\xBB\xC0` will be represented as `"0abbc0"`.

Different platforms may internally use different byte ordering (big endian, little endian). All binary sequences in this format need to be stored **MSB-LSB (big endian)**. For example, IEEE addresses are represented within Z-Stack LSB-MSB (little endian), therefore address `00:12:4b:00:06:10:4e:22` would be internally stored as `22:4e:10:06:00:4b:12:00`. For purposes of this specification the address needs to be reversed and converted to specified byte array representation form - `00124b0006104e22`.

## Backup Structure
The generated backup is a JSON document with the following top-level keys:
* `metadata`: *object*,
* `stack_specific`: *object*,
* `coordinator_ieee`: *string*,
* `pan_id`: *string*,
* `extended_pan_id`: *string*,
* `channel`: *number*,
* `channel_mask`: *number[]*,
* `security_level`: *number*,
* `nwk_update_id`: *number*,
* `network_key`: *object*,
* `devices`: *object[]*.

### `metadata`
The top-level `metadata` object contains basic information about the backup itself:
* `format`: *string* - needs to be set to `zigpy/open-coordinator-backup`,
* `version`: *number* - specifier that will be incremented upon major specification
 changes that will require migration or introduce new keys. The current value is `1`,
* `source`: *string* - describes the application or package generating the file. This value will be the name of the package, followed by a literal `@`, then the version identifier of the generating package. For example, `zigpy-znp@0.3.1` or `zigbee-herdsman@d12727f3`. The purpose of this key is to uniquely identify the application generating the backup format and potentially work around bugs introduced in specific versions,
* `internal`: *object* - arbitrary-structure object that can be used by the generating application to store any other useful non-network information. For the purposes of interoperability, it is not recommended to store any critical network information, for example stack-specific settings, in this object. Use the top-level `stack_specific` key with the appropriate sub-object for the stack.

### `stack_specific`
The top-level `stack_specific` object contains information that is necessary to restore a backup but using information that is specific to the stack the backup was created from.

Currently the only sub-object is `zstack` with a key containing the Z-Stack Trust Center Link Key seed. Z-Stack uses **Trust Center Link Key Seed** to generate unique link keys for APS encryption. Only seed shift is stored and actual keys are derived from this seed.

Therefore the structure currently supports the following keys:
* `zstack`: *object*,
   * `tclk_seed`: *string* - 128-bit binary-encoded TCLK seed.

*Other stack-specific fields may be added as they are used.*

### `coordinator_ieee`
Unique 64-bit coordinator adapter IEEE address represented as described in [Encoding Sequences of Bytes](#Encoding-Sequences-of-Bytes). Example value: `00124b0009d69f77`.

### `pan_id`
The network's 16-bit Personal Area Network Identifier (PAN ID) will be stored big-endian and encoded as hex, with leading zeroes as per [Encoding Sequences of Bytes](#Encoding-Sequences-of-Bytes). Example value: `d4a1`.

### `extended_pan_id`
The 64-bit Extended Personal Area Network Identifier (EPID) stored as described in [Encoding Sequences of Bytes](#Encoding-Sequences-of-Bytes). Example value: `00124b0009418a6b`.

### `channel`
The current logical channel of the network will be stored as an integer between 11 and 26.

### `channel_mask`
The logical channels on which the network can be formed will be stored as an array of numbers, each between 11 and 26. If the network can only be formed on a single channel, the array will contain a single integer.

### `security_level`
The network security level will be stored as an integer between 0 and 7. Supported values are listed in the table below ([source](https://research.kudelskisecurity.com/2017/11/08/zigbee-security-basics-part-2/)).

| Value | Security Level Identifier |	Security Attributes |	Data Encryption |	Frame Integrity (length of MIC) |
| - | - | - | - | - |
| **0** | 0x00 | None | OFF | NO (M = 0) |
| **1** | 0x01 | MIC-32 | OFF | YES (M=4) |
| **2** | 0x02 | MIC-64 | OFF | YES (M=8) |
| **3** | 0x03 | MIC-128 | OFF | YES (M=16) |
| **4** | 0x04 | ENC | ON | NO (M = 0) |
| **5** | 0x05 | ENC-MIC-32 | ON | YES (M=4) |
| **6** | 0x06 | ENC-MIC-64 | ON | YES (M=8) |
| **7** | 0x07 | ENC-MIC-128 | ON | YES (M=16) |

### `nwk_update_id`
The network update identifier will be stored as an integer between 0 and 255.

### `network_key`
This object contains vital information related to network key required to restore the network. The following keys are present:
* `key`: *string* - 128-bit key encoded as described in [Encoding Sequences of Bytes](#Encoding-Sequences-of-Bytes),
* `sequence_number`: *number* - value of 0 to 255 indicating the re-keying sequence,
* `frame_counter`: *number* - numeric value of the 32-bit network frame counter.

### `devices`
This is an array of devices recognized (paired with) by the coordinator. Every item maps network address to extended IEEE address of a device - which may be required by some adapters.

The primary reason for this array is storage of APS encryption unique link keys, which may be required by device using APS security.

Every object within this array contains the following fields:
* `nwk_address`: *string* - 16-bit device network address encoded as per [Encoding Sequences of Bytes](#Encoding-Sequences-of-Bytes),
* `ieee_address`: *string* - 64-bit IEEE address of the device encoded as per [Encoding Sequences of Bytes](#Encoding-Sequences-of-Bytes),
* `link_key`: *object* (optional),
   * `key`: *string* - 128-bit key encoded as described in [Encoding Sequences of Bytes](#Encoding-Sequences-of-Bytes),
   * `rx_counter`: *number* - value of 32-bit receive frame counter for the link key,
   * `tx_counter`: *number* - value of 32-bit transmit frame counter for the link key.

## Samples
| Name | Source | Description |
| - | - | - |
| [z2m-sample-1.json](samples/z2m-sample-1.json) | Zigbee2MQTT | Backup taken from CC2538 adapter on a formed network with working APS encryption. |

## TBD
* JSON Schema,
* Structures for various languages (TS, Python, ...),
* Information on migrations.
