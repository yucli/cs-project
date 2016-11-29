# Bitmessage protocol
## My uses
|original use           |translation  |
|:---------------------:|:-----------:|
|Proof of Work          |POW          |
|indicating             |->           |
|identifying            |(=)          |
|results in             |=>           |
|not                    |!            |
|confused with          |~-~          |
|Variable length integer|vli          |
|followed by            |<-~          |
|prefixed with          |@-           |
|notifying              |->3          |
|Inventory vector       |iv           |
## Common standards
### Hashes
#### SHA-512 & RIPEMD-160
> double round of SHA-512 for **POW**
>
> one round of SHA-512 and RIPEMD-160 for creating an **address**

## Common structures
> All integers are encoded in **big endian** (different from Bitcoin).

### Message Structures
|description(size, data type)|comments|
|---------------------------|--------|
|magic(4, uint32_t)         |magic value -> **message origin network**, and used to seek next message when **stream state unknown**|
|command(12, char[12])      |ASCII string (=) the **package content, NULL padded**(non-NULL padding => packet rejected)|
|length(4, uint32_t)        |**length of payload in number of bytes**; other restrictions => some clients use **sanity-check** to avoid processing message > 1600003 bytes|
|check sum(4, uint32_t)     |first 4 bytes of **sha-512(payload)**|
|message_payload(?, uchar[])|the actual data, a **message** or an **object**(!~-~ **object payload**)|

#### known magic values:
|magic value|sent over wire as|
|-----------|-----------------|
|0xE9BEB4D9|E9 BE B4 D9|

### <font color = red>Variable length integer</font>(?)
> Varints **MUST** use the **minimum** possible number of bytes to encode a value(for **saving space**)

|value        |storage length|format |
|-------------|--------------|-------|
|<0xfd        |1             |uint8_t|
|<=0xffff     |3             |0xfd followed by the integer as uint16_t|
|<= 0xffffffff|5             |0xfe followed by the integer as uint32_t|
|-            |9             |0xff followed by the integer as uint64_t|

### Variable length string
> stored using a vli <-~ the string itself

|description(size, data type)|comments|
|---------------------------|--------|
|length(1+, **var_int**)    |length of the string|
|string(?, char[])          |the string itself(**can be empty**)|

### Variable length list of integers
> n integers can be stored using n+1 variable length integers where the first var_int equals n.

|description(size, data type)|comments|
|---------------------------|--------|
|count(1+, var_int)         |number of var_ints below|
|(1+, var_int)              |the first value stored|
|(1+, var_int)              |etc...|

### Network address
> used when needed. **!@- a timestamp or stream in the version message**

|description(size, data type)|comments    |
|---------------------------|------------|
|time(8, uint64_t)          |the **Time**|
|stream(4, uint32_t)        |**Stream number for this node**|
|services(8, uint64_t)      |same service(s) listed in **version**|
|IPv6/4(16, char[16])       |IPv6 address. IPv4(12 bytes 00 00 00 00 00 00 00 00 00 00 FF FF, <-~ the 4 bytes of the IPv4 address).|
|port(2, uint16_t)          |**port** number|

### <font color = red>Inventory Vectors</font>
> used for ->3 other nodes about **objects** they have or **data** which is being requested. Two rounds of SHA-512 are used, => a 64 byte hash. Only the **first 32** bytes are used; the later 32 bytes are ignored

#### ivs consist of the following data format:
|description(size, data type)|comments    |
|---------------------------|------------|
|hash(32, char[32])         |hash of the object|

### Encrypted payload(表格?)
> Bitmessage uses **ECIES** to encrypt its messages.

|description(size, data type)|comments    |
|---------------------------|------------|
|IV(16, uchar[])            |**Initialization Vector** used for AES-256-C|
|uint16_t(2, Curve type)    |Elliptic Curve type 0x02CA (714)|
|uint16_t(2, X length)      |Length of X component of **public key R**|
|uchar[] (X length, X)      |X component of public key R|
|uint16_t(2, Y length)      |Length of Y component of public key R|
|uchar[] (Y length, Y)      |Y component of public key R|
|encrypted(?, uchar[])      |**Cipher text**|
|MAC(32, uchar[]   )        |HMACSHA256 **Message Authentication Code**|

### <font color = red>Unencrypted Message Data</font>
|description(size, data type)   |comments    |
|-------------------------------|------------|
|msg_version(1+, var_int)       |Message format version. This field is <font color = red>not included after the protocol v3 upgrade period</font>.|
|address_version(1+, var_int)   |**Sender's address version number** needed in order to **calculate the sender's address to show in the UI**, and also to allow for **forwards compatible changes** to the **public-key data** included below.|
|stream(1+, var_int)            |**Sender's stream number**|
|behavior bitfield(4, uint32_t) |<font color = red>A **bitfield** of **optional behaviors and features** that can be expected from the node with this **pubkey** included in this **msg message**(the sender's pubkey).</font>|
|public signing key(64, uchar[])|The **ECC** public key used for **signing** (**uncompressed** format; normally **prepended with \x04** )|
|public encryption key(64, uchar[])|The **ECC** public key used for **encryption** (uncompressed format; normally prepended with \x04 )|
|nonce_trials_per_byte(1+, var_int)|Used to **calculate the difficulty target of messages accepted by this node**. higher => more difficult the POW before this individual will accept the message. **the average number of nonce trials a node** will have to perform to **meet** the POW requirement. **1000** is the network minimum so any lower values will be automatically raised to 1000. This field is new and is only included when the **address_version >= 3**.|
|extra_bytes(1+, var_int)       |Used to **calculate the difficulty target of messages accepted by this node**. higher => more difficult the POW before this individual will accept the message. This number is **added to the data length** to make sending **small messages** more difficult. **1000** is the network minimum so any lower values will be automatically raised to 1000. This field is new and is only included when the **address_version >= 3**.|
|destination ripe(20, uchar[])  |The **ripe hash of the public key** of the **receiver** of the message|
|encoding(1+, var_int)          |**Message Encoding** type|
|message_length(1+, var_int)    |Message Length|
|message(message_length, uchar[])|The message|
|ack_length(1+, var_int)        |Length of the **acknowledgement data**|
|ack_data(ack_length, uchar[])  |The acknowledgement data to be transmitted. This takes the **form** of a **Bitmessage protocol message**, like another **msg message**. The POW therein must already be **completed**.|
|sig_length(1+, var_int)        |Length of the **signature**|
|signature(sig_length, uchar[]) |The **ECDSA(Elliptic Curve Digital Signature Algorithm)** signature which covers the **object header starting with the time**, appended with the data described in this table down to the ack_data.|

### Message Encodings
|value(name)|description|
|-----------|-----------|
|0(IGNORE)  |Any data with this number may be **ignored**. The **sending node** might simply be **sharing its public key with you**.|
|1(TRIVIAL) |UTF-8. No 'Subject' or 'Body' sections. Useful for **simple strings** of data, like **URIs or magnet links**.|
|2(SIMPLE)  |UTF-8. Uses 'Subject' and 'Body' sections. No MIME is used. **messageToTransmit = 'Subject:' + subject + '\n' + 'Body:' + message**|
|3(EXTENDED)|A data structure in **msgpack**, then **compressed with zlib**. Null data type is encoded as an empty string, and booleans as an integer 0 (false) or 1 (true). Text fields are encoded using UTF-8. **v5** and newer address versions MUST support this. Proposal, exact structure pending standardisation.|

> <font color = red>Further values for the message encodings can be decided upon by the community. Any MIME or MIME-like encoding format, should they be used, should make use of **Bitmessage's 8-bit bytes**.</font>?

### Pubkey bitfield features
|bit(name)              |description|
|-----------------------|-----------|
|0(undefined)           |The most significant bit at the beginning of the structure. Undefined|
|1(undefined)           |The next most significant bit. Undefined|
|...(...)               |...|
|29(extended_encoding)  |**Receiving node** supports **extended encoding**.|
|30(include_destination)|<font color = red>**Receiving node** expects that the **RIPE hash encoded in their address** preceedes the encrypted message data of **msg messages** bound for them.</font>|
|31(does_ack)           |If **true**, the **receiving node** does **send acknowledgements** (rather than dropping them).|

## Message types
> Undefined messages received on the wire must be ignored.

### version
> When a node creates an **outgoing connection**, it will immediately advertise its version. The remote node will respond with its version. No futher communication is possible until both peers have **exchanged their version**

#### payload
|description(size, data type)|comments    |
|----------------------------|------------|
|version(4, int32_t)         |(=) **protocol version** being used by the node. Should equal **3**. Nodes should disconnect if the remote node's version is lower but continue with the connection if it is higher.|
|services(8, uint64_t)       |**bitfield of features** to be enabled for this connection|
|timestamp(8, int64_t)       |standard UNIX timestamp in **seconds**|
|addr_recv(26, net_addr)     |The network address of the node receiving this message (**not including the time or stream number**)|
|addr_from(26, net_addr)     |The network address of the node emitting this message (**not including the time or stream number** and the **ip itself** is **ignored** by the receiver)|
|nonce(8, uint64_t)|<font color = red>**Random** nonce used to **detect connections to self**.</font>|
|user_agent(1+, var_str)     |<font color = red>**User Agent**</font> (0x00 if string is 0 bytes long). Sending nodes must ! include a user_agent > 5000 bytes.|
|stream_numbers(1+, var_int_list)|The stream numbers that the emitting node is **interested in**. Sending nodes must ! include > 160000 stream numbers.|

> **A "verack" packet shall be sent if the version packet was accepted.** Once you have sent and received a verack messages with the remote node, send an **addr message advertising up to 1000 peers of which you are aware**, and **one or more inv messages advertising all of the valid objects of which you are aware**.

#### The following services are currently assigned:
|value(name)    |description|
|---------------|-----------|
|1(NODE_NETWORK)|This is a normal network node.|
|2(NODE_SSL)    |This node supports **SSL/TLS** in the current connect (python < 2.7.9 only supports a SSL client, so in that case it would only have this on when the connection is a client).|

### verack
> The verack message is sent in reply to version. This message consists of only **a message header with the command string "verack"**. **The TCP timeout** starts out at 20 seconds; after verack messages are exchanged, the timeout is raised to 10 minutes.
>
> If both sides announce that they **support SSL**, they MUST perform **a SSL handshake immediately after they both send and receive verack**. During this SSL handshake, the TCP client acts as a SSL client, and the TCP server acts as a SSL server. The current implementation (**v0.5.4 or later**) requires the **AECDH-AES256-SHA cipher over TLSv1 protocol**, and prefers the **secp256k1 curve** (but other curves may be accepted, depending on the version of python and OpenSSL used).

### addr
> Provide information on **known nodes** of the network. Non-advertised nodes should be forgotten after typically **3 hours**

#### Payload:
|description(size, data type)|comments    |
|----------------------------|------------|
|count(1+, var_int)          |Number of address entries (**max: 1000**)|
|addr_list(38, net_addr)     |Address of other nodes on the network.|

### inv
#### Allows a node to advertise its knowledge of one or more objects. Payload (maximum payload length: 50000 items):

|description(size, data type)|comments    |
|----------------------------|------------|
|count(?, var_int)           |Number of inventory entries|
|inventory(32x?, inv_vect[]) |**Inventory vectors**|

### getdata
> getdata is used in **response to an inv message** to **retrieve the content of a specific object** after filtering known elements.

#### Payload (maximum payload length: 50000 entries):
|description(size, data type)|comments    |
|----------------------------|------------|
|count(?, var_int)           |Number of inventory entries|
|inventory(32x?, inv_vect[]) |**Inventory vectors**|

### object
> An object is **a message which is shared throughout a stream**. It is the only message which propagates; **all others are only between two nodes**. Objects **have a type, like 'msg', or 'broadcast'**. <font color = red>**To be a valid object, the POW must be done.**</font> The maximum allowable length of an object (**!~-~ the objectPayload**) is **218 bytes**.

|description(size, data type)|comments    |
|----------------------------|------------|
|nonce(8, uint64_t)          |**Random nonce** used for the POW|
|expiresTime(8, uint64_t)    |The "end of life" time of this object (be aware, in version 2 of the protocol this was the generation time). Objects shall be shared with peers until its end-of-life time has been reached. **The node should store the inventory vector of that object for some extra period of time** to avoid reloading it from another node with a small time delay. The time may be **no further than 28 days + 3 hours in the future**.|
|objectType(4, uint32_t)     |Four values are currently defined: **0-"getpubkey", 1-"pubkey", 2-"msg", 3-"broadcast"**. All other values are reserved. Nodes **should relay** objects even if they use an **undefined object type**.|
|version(1+, var_int)        |**The object's version**. Note that msg objects won't contain a version until Sun, 16 Nov 2014 22:00:00 GMT.|
|stream number(1+, var_int)  |**The stream number in which this object may propagate**|
|objectPayload(?, uchar[])   |This field **varies depending on the object type**; see below.|

## Object types
> Here are the **payloads** for various object types.

### getpubkey
> When a node **has the hash of a public key (from an address) but not the public key itself**, it must send out a request for the public key.

|description(size, data type)|comments    |
|----------------------------|------------|
|ripe(20, uchar[])           |The **ripemd hash of the public key**. This field is only included when the **address version is <= 3**.|
|tag(32, uchar[])            |The tag **derived from the address version, stream number, and ripe**. This field is only included when the **address version is >= 4**.|

### pubkey
#### A version 2 pubkey. This is still in use and supported by current clients but new v2 addresses are not generated by clients.

|description(size, data type)      |comments    |
|----------------------------------|------------|
|behavior bitfield(4, uint32_t)    |A bitfield of **optional behaviors and features** that can be expected from the node receiving the message.|
|public signing key(64, uchar[])   |The **ECC** public key used for **signing** (**uncompressed** format; normally **prepended with \x04**)|
|public encryption key(64, uchar[])|The **ECC** public key used for **encryption** (uncompressed format; normally prepended with \x04 )|

#### A version 3 pubkey
|description(size, data type)      |comments    |
|----------------------------------|------------|
|behavior bitfield(4, uint32_t)    |A bitfield of **optional behaviors and features** that can be expected from the node receiving the message.|
|public signing key(64, uchar[])   |The **ECC** public key used for **signing** (**uncompressed** format; normally **prepended with \x04**)|
|public encryption key(64, uchar[])|The **ECC** public key used for **encryption** (uncompressed format; normally prepended with \x04 )|
|nonce_trials_per_byte(1+, var_int)|Used to **calculate the difficulty target of messages accepted by this node**. higher => more difficult the POW before this individual will accept the message. This number is the **average number of nonce trials a node** will have to perform to **meet** the POW requirement. **1000** is the network minimum so any lower values will be automatically raised to 1000.|
|extra_bytes(1+, var_int)          |Used to **calculate the difficulty target of messages accepted by this node**. higher => more difficult the POW before this individual will accept the message. This number is **added to the data length to make sending small messages more difficult**. **1000** is the network minimum so any lower values will be automatically raised to 1000.  |
|sig_length(1+, var_int)           |Length of the signature|
|signature(sig_length, uchar[])    |The **ECDSA** signature which, <font color = red>as of protocol v3, covers the object header starting with the time, appended with the data described in this table down to the extra_bytes.</font>|

#### A version 4 pubkey
|description(size, data type)      |comments    |
|----------------------------------|------------|
|tag(32, uchar[])                  |The tag made up of bytes **32-64** of the **double hash** of the **address data** (see example python code below).|
|encrypted(?, uchar[])             |**Encrypted pubkey data.**|

> When version 4 pubkeys are created, most of the data in the pubkey is encrypted. This is done in such a way that <font color = red>**only someone who has the Bitmessage address which corresponds to a pubkey can decrypt and use that pubkey.**</font> This prevents people from gathering pubkeys sent around the network and using the data from them to create messages to be used in spam or in flooding attacks.
>
> In order to encrypt the pubkey data, **a double SHA-512 hash** is calculated from **the address version number, stream number, and ripe hash of the Bitmessage address that the pubkey corresponds to**. The **first 32 bytes** of this hash are used to create a public and private key pair with which to encrypt and decrypt the pubkey data, using the same algorithm as message encryption (see Encryption). The **remaining 32 bytes** of this hash are added to the **unencrypted** part of the pubkey and used as a **tag**, as above. This allows nodes to determine which pubkey to decrypt when they wish to send a message.
>
> In PyBitmessage, the double hash of the address data is calculated using the python code below:
> **doubleHashOfAddressData = hashlib.sha512(hashlib.sha512(encodeVarint(addressVersionNumber) + encodeVarint(streamNumber) + hash).digest()).digest()**

#### Encrypted data in version 4 pubkeys:(?)
> same as **A version 3 pubkey** except the below entity

|description(size, data type)      |comments    |
|----------------------------------|------------|
|signature(sig_length, uchar[])    |The **ECDSA** signature which covers everything from the object header starting with the time, then appended with the **decrypted data** down to the extra_bytes. <font color = red>This was changed in protocol v3.</font>|

### msg
> Used for **person-to-person messages**. Note that msg objects won't contain a version in the object header until Sun, 16 Nov 2014 22:00:00 GMT.

|description(size, data type)      |comments    |
|----------------------------------|------------|
|encrypted(?, uchar[])             |**Encrypted data**. See Encrypted payload. See also Unencrypted Message Data Format|

### broadcast(?)
> Users who are subscribed to the sending address will see the message appear in their inbox. Broadcasts are <font color = red>version 4 or 5</font>.
>
> Pubkey objects and **v5** broadcast objects are encrypted the same way: The data encoded in the **sender's Bitmessage address is hashed twice**. The **first 32 bytes** of the resulting hash constitutes the **"private" encryption key** and the **last 32 bytes** constitute a **tag** so that anyone **listening** can easily decide if this particular message is interesting. The sender calculates the public key from the private key and then <font color = red>**encrypts the object with this public key**</font>. Thus anyone who knows the Bitmessage address of the sender of a broadcast or pubkey object can decrypt it.
>
> The version of broadcast objects was previously 2 or 3 but was <font color = red>changed to 4 or 5 for protocol v3</font>. Having a broadcast version of **5** indicates that a tag is used which, in turn, is used when the **sender's address version is >=4**.

|description(size, data type)|comments    |
|----------------------------|------------|
|tag(32, uchar[])            |the tag. This field is new and only included when the **broadcast version is >= 5**. Changed in protocol v3|
|encrypted(?, uchar[])       |Encrypted broadcast data. The **keys** are derived as described in the **paragraph above**. See Encrypted payload for details about the encryption algorithm itself.|

#### Unencrypted data format:
|description(size, data type)       |comments    |
|-------------------------------    |------------|
|broadcast version(1+, var_int)     |The version number of this broadcast protocol message which is equal to 2 or 3. This is included here so that it can be signed. <font color = red>This is no longer included in protocol v3</font>|
|address version(1+, var_int)       |The sender's address version|
|stream number(1+, var_int)         |The sender's stream number|
|behavior bitfield(4, uint32_t)	    |A bitfield of optional behaviors and features that can be expected from the **owner of this pubkey**.|
|public signing key(64, uchar[])    |The ECC public key used for signing (uncompressed format; normally prepended with \x04 )|
|public encryption key(64, uchar[]) |The ECC public key used for encryption (uncompressed format; normally prepended with \x04 )|
|nonce_trials_per_byte(1+, var_int) |same as the one in **Unencrypted Message Data**|
|extra_bytes(1+, var_int)           |same as the one in **Unencrypted Message Data**|
|encoding(1+, var_int)             |The **encoding type** of the message|
|messageLength(1+, var_int)        |The message length in bytes|
|message(messageLength, uchar[])   |**The message**|
|sig_length(1+, var_int)           |Length of the signature|
|signature(sig_length, 	uchar[])   |The signature which did cover the unencrypted data from the broadcast version down through the message. <font color = red>In protocol v3, it covers the unencrypted object header starting with the time, all appended with the decrypted data</font>.|
