# Open Zigbee coordinator backup format
An ongoing specification for a JSON Zigbee coordinator backup format.


## Rationale
The primary use case for this specification is to provide a simple standard for exporting
Zigbee network information to disk. Standardizing this format will allowing users to
backup, restore, and migrate their networks between coordinator hardware and network
management software without having to needlessly rejoin all of their devices to a new
network.


## Provisional specification
JSON is both human-readable supported well by common programming languages, making it a
good fit for files that will be interpreted by both.


### Encoding sequences of bytes
JSON does not natively support sequences of bytes (`char[]`, `Buffer`, bytestrings, etc.)
so an intermediate representation is necessary.

Sequences of bytes will be encoded as a case-insensitive string of each
byte's hex representation, with leading zeroes, concatenated together. For example,
`\x0A\xBB\xC0` will be represented as `"0abbc0"`.


### `metadata`
The top-level `metadata` object contains three keys:
 - An integer `version` specifier that will be incremented upon major specification
   changes that will require migration or introduce new keys. The current value is `0`.
 - A string `source` that describes the application or package generating the file.
   This value will be the name of the package, followed by a literal `@`, then the
   version identifier of the generating package. For example, `zigpy-znp@0.3.1` or
   `zigbee-herdsman@d12727f3`. The purpose of this key is to uniquely identify the
   application generating the backup format and potentially work around bugs introduced
   in specific versions.
 - An opaque `internal` object that can be used by the generating application to store
   any other useful non-network information. For the purposes of interoperability, it is
   not recommended to store any critical network information, for example stack-specific
   settings, in this object. Use the top-level `stack_specific` key with the appropriate
   sub-object for the stack.

### `stack_specific`
The top-level `stack_specific` object contains information that is necessary to restore
a backup but using information that is specific to the stack. For example, the seed for
hashed link keys in Z-Stack. To ensure backups created by different applications working
with the same Zigbee stack remain compatible and to simplify parsing, this object will
contain a top-level key identifying the stack.

Currently the only sub-object is `zstack` with a key containing the Z-Stack Trust Center
Link Key seed. For example:

```JSON
{
	"stack_specific": {
		"zstack": {
			"tclk_seed": "6ad02ebcdd3159547a1f2228985e6978"
		}
	}
}
```

### Basic network information
Basic information about the network is stored in the following top-level keys:

#### `pan_id`
The network's 16-bit PAN identifier will be stored big-endian and encoded as hex, with
leading zeroes. This is to match its display format in most other software. For example,
`0xABCD` will be stored as `"abcd"`.

#### `extended_pan_id`
The 64-bit extended PAN identifier will be stored big-endian and encoded as hex. For
example, `77:66:55:44:33:22:11:00` will be stored as `"7766554433221100"`.

#### `coordinator_ieee`
The 64-bit IEEE address of the coordinator will be stored big-endian and encoded as hex.
For example, `00:11:22:33:44:55:66:77` will be stored as `"0011223344556677"`.

#### `channel`
The current logical channel of the network will be stored as an integer between 11 and
26.

#### `channel_mask`
The logical channels on which the network can be formed will be stored as an array of
integers, each between 11 and 26. If the network can only be formed on a single channel,
the array will contain a single integer.

#### `nwk_update_id`
The network update identifier will be stored as an integer between 0 and 255.

#### `security_level`
The network security level will be stored as an integer between 0 and 7.

#### `network_key`
...


### Device-specific information
...



## Sample JSON
```JSON
{
	...
}
```
