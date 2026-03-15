DELEGATECALL NOTES
What is it:

A special type of message call in EVM
Executes the target contract's CODE but in the CALLER's context
msg.sender, msg.value, and storage all remain of the calling contract
How it works:

Contract A delegatecalls Contract B
B's code runs, but reads/writes A's storage
B sees msg.sender as the original caller, not A
Key difference from regular call:

call: runs code in target's context, uses target's storage
delegatecall: runs code in caller's context, uses caller's storage
staticcall: read-only, no state changes allowed
Why it exists:

Powers proxy patterns (UUPS, Transparent, Beacon)
Proxy holds storage + receives calls
Implementation holds logic, executed via delegatecall
Allows upgradeability — swap implementation, keep storage
Storage context rule:

slot 0 in implementation maps to slot 0 in proxy
slot 1 maps to slot 1, and so on
Variable NAMES don't matter, only SLOT POSITIONS matter
This is why storage layout must match between proxy and implementation
Example of the danger:
Proxy storage:      Implementation storage:
slot 0 → admin      slot 0 → owner
slot 1 → impl       slot 1 → balance
If implementation writes to "owner" (slot 0), it actually overwrites "admin" in proxy. This is a storage collision.Return behavior:

delegatecall returns success/failure bool and return data
If the called function reverts, delegatecall returns false
Low-level: (bool success, bytes memory data) = target.delegatecall(calldata)
Must check success manually — doesn't auto-revert
Security notes:

Never delegatecall to untrusted/user-supplied addresses
Implementation contracts should be initialized or use disable-initializer to prevent attackers calling initialize directly
selfdestruct in implementation via delegatecall destroys the PROXY (pre-Dencun)
Storage layout mismatch = silent corruption, very hard to debug
If implementation has a function that does delegatecall to user input → attacker gets full control of proxy storage