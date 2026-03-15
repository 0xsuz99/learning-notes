ABI ENCODING NOTES
What is ABI encoding:

How function calls and data are converted to raw bytes for the EVM
Used in calldata (external calls), events, and abi.encode functions
EVM only understands bytes — ABI is the translation layer

Function selector:

First 4 bytes of keccak256 of the function signature
Example: transfer(address,uint256) → keccak256 → first 4 bytes = 0xa9059cbb
No spaces in signature, use canonical types (uint256 not uint)

Static types encoding:

Always 32 bytes, left-padded with zeros
uint256(42) → 0x000...002a
bool(true) → 0x000...0001
address → 0x000...{20 bytes}
bytesN → right-padded (opposite of other types)

Dynamic types encoding (string, bytes, arrays):

Uses offset/pointer system — head + tail
Head section: contains offsets pointing to where actual data lives
Tail section: contains length + actual data, padded to 32-byte chunks

Example — foo(uint256, string) called with (42, "hello"):
0x00: 000...002a          ← 42 (static, placed directly)
0x20: 000...0040          ← offset (64 bytes from start → points to string data)
0x40: 000...0005          ← string length (5)
0x60: 68656c6c6f000...00  ← "hello" padded to 32 bytes
abi.encode vs abi.encodePacked:

abi.encode: standard, 32-byte padded, unambiguous decoding
abi.encodePacked: tightly packed, no padding, shorter output
encodePacked is NOT safe for hashing multiple dynamic types

encodePacked collision example:
abi.encodePacked("ab", "c") == abi.encodePacked("a", "bc")
Both produce: 0x616263
No length prefix or padding → can't tell where first arg ends
Encoding for different contexts:

abi.encode(args) → standard encoding, used for general purposes
abi.encodePacked(args) → tight packing, used sometimes for hashing
abi.encodeWithSelector(selector, args) → selector + encoded args
abi.encodeWithSignature("func(type)", args) → computes selector for you
abi.encodeCall(Contract.func, (args)) → type-safe, compile-time checked (safest)

Decoding:

abi.decode(data, (types)) → decodes encoded bytes back to values
If data doesn't match expected types → reverts

Security notes:

Never use abi.encodePacked with multiple dynamic types for hashing — use abi.encode instead
Calldata can be crafted manually — don't assume it came from a proper function call
Signature collisions: 4-byte selectors can collide (2^32 space) — rare but real
Malformed calldata with extra bytes at the end is silently ignored by Solidity — can be used to trick off-chain systems
Low-level calls with hand-crafted calldata bypass compiler type checks — always validate
