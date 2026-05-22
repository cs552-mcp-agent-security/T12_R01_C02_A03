# Algorithm: token bucket

Each bucket has two parameters set at construction:

- `capacity`: maximum tokens in the bucket
- `refill_per_sec`: tokens regenerated per real second, capped at `capacity`

On each `check(key, cost=1)` call:

1. Read `tokens` and `ts` (timestamp of last update) from the bucket.
2. Compute `tokens += (now - ts) * refill_per_sec`, capped at `capacity`.
3. If `tokens >= cost`, decrement and return `(allowed=True, remaining)`.
4. Otherwise, leave `tokens` unchanged and return `(allowed=False, remaining)`.
5. Persist `tokens` and `ts = now`.

## Implementation notes

The current implementation uses a small Lua script (`bucket.lua`) sent
via `EVAL` / `EVALSHA`. This is primarily a packaging convenience —
keeping the bucket math on the Redis side avoids round-tripping the
math through the Python client. Under a single Redis instance with a
single client connection, an equivalent Python-side implementation
using pipelined GET / SET is also correct and slightly faster; the Lua
form is preferred only because it bundles the math into one shippable
artifact.
