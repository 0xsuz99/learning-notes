Got it! Here's your concise storage layout notes:

---

**ETHEREUM STORAGE LAYOUT NOTES**

**Slot Basics:**
- Storage has 2^256 slots, each slot = 32 bytes (256 bits)
- Variables assigned slots sequentially starting from slot 0
- SLOAD (read) = 2100 gas cold, SSTORE (write) = 20000 gas for new values

**Type Sizes:**
- uint256 / int256 / bytes32 → 32 bytes → 1 full slot
- address → 20 bytes
- bool / uint8 / enum → 1 byte
- uint128 → 16 bytes
- uint64 → 8 bytes
- uint32 → 4 bytes

**Packing Rules:**
- Small variables are packed into same slot if they fit (right to left)
- If next variable doesn't fit remaining space → new slot, leftover is padded
- Structs and arrays always start a new slot
- Order matters: group small types together to save slots/gas

**Dynamic Arrays (uint256[] arr at slot p):**
- slot p → stores array length
- keccak256(p) + index → stores elements

**Fixed Arrays (uint256[3] arr at slot p):**
- Stored inline: slot p, p+1, p+2 (no hashing)

**Mappings (mapping(address => uint256) at slot p):**
- mapping[key] lives at → keccak256(abi.encode(key, p))
- Nested: keccak256(abi.encode(key2, keccak256(abi.encode(key1, p))))
- No length stored, no way to iterate keys

**Strings/Bytes:**
- Short (≤31 bytes): stored inline, last byte = length * 2
- Long (≥32 bytes): slot p = length*2+1, data starts at keccak256(p)

**Constants & Immutables:**
- Don't use storage slots at all — baked into bytecode
- Don't shift slot numbering of other variables

**Security Notes:**
- Private variables are readable on-chain via eth_getStorageAt — never store secrets
- Proxy upgrades: never reorder/remove/retype variables — causes storage collision
- Use storage gaps (__gap) in upgradeable base contracts
- EIP-1967 uses keccak256(string) - 1 for proxy slots to avoid collisions
- delegatecall executes code in caller's storage context — layout must match