<div class="cwikmeta">{
"title":"FILTERLOAD",
"related":["/protocol","/protocol/network/messages/filterclear.md", "/protocol/network/messages/inv.md", "/protocol/network/messages/merkleblock.md"]
}</div>

# Request: Filter Load (“filterload”)

Inserts a transaction and merkle block filter into the peer, overwriting any existing filter.

### Effect on Transactions

This message installs a bloom filter into the peer.  Subsequent [inv](/protocol/network/messages/inv) and [merkleblock](/protocol/network/messages/merkleblock) messages only provide transactions that in match this bloom filter in some manner.  The following items in a transaction are checked against the bloom filter:

 - The transaction hash
 - Each data field in every [output script](/glossary/output__script.md) in the transaction
	 - Most importantly, this allows public keys and public key hashes (essentially bitcoin addresses) to be added to the bloom filter, allowing a wallet to detect an incoming transfer.
 - Each [previous output](/glossary/previous__output.md) in the transaction
	 - This allows a wallet to detect that a different wallet has spent funds that are co-controlled
 - Each data field in every [input script](/glossary/input__script.md) in the transaction.

See [CBloomFilter::MatchAndInsertOutputs](https://github.com/BitcoinUnlimited/BitcoinUnlimited/blob/bucash1.7.0.0/src/bloom.cpp#L186), and [CBloomFilter::MatchInputs](https://github.com/BitcoinUnlimited/BitcoinUnlimited/blob/eb264e627e231f7219e60eef41b4e37cc52d6d9d/src/bloom.cpp#L234)

### Effect on Merkle Blocks

If a [filtered block](https://github.com/BitcoinUnlimited/BitcoinUnlimited/blob/bucash1.7.0.0/src/protocol.h#L483) is requested via an inventory message, the installed bloom filter is used to choose which transactions are included in the response using the same matching algorithm as described above for transaction inventories.

## Message Format

| Field | Length | Format | Description |  
|--|--|--|--|
| bloom filter size | variable | [variable length integer](/protocol/formats/variable-length-integer) | The size of the bloom filter to be used. |
| bloom filter data | variable | `bloom_filter_size` bytes | Raw bloom filter data (up to 36,000 bytes). |
| number of hash functions | 4 bytes | unsigned integer<sup>[(LE)](/protocol/misc/endian/little)</sup> | The number of hash functions used when generating this filter (max 50). |
| tweak | 4 bytes | unsigned integer<sup>[(LE)](/protocol/misc/endian/little)</sup> | Arbitrary number that helps randomize this bloom filter. |
| flags | 1 byte | bit field | See [bloom filter flags](#bloom-filter-flags). |

### Bloom Filter Flags

 The lowest two bits of flags control how the installed bloom filter is  automatically updated.  The remaining bits are reserved and **should** be set to 0.

| value | description |
|--|--|
| 0 | Do not update
| 1 | update all
| 2 | update pay-to-pubkey, multi-sig, or check-lock-time-verify scripts only

