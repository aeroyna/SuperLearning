# Lua Scripting

## Learning Objectives
- Write and execute Lua scripts in Redis
- Implement atomic operations with scripts
- Optimize performance with server-side logic
- Apply best practices for Lua scripting

---

## 1. Lua Scripting Fundamentals

### Why Lua Scripts?

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Benefits of Lua Scripts                           │
│                                                                      │
│  Without Scripts (Multiple Round-trips):                            │
│  ┌────────┐            ┌────────┐                                   │
│  │ Client │───GET───▶  │ Redis  │                                   │
│  │        │◀──value──  │        │                                   │
│  │        │──INCR────▶ │        │  3 round-trips                   │
│  │        │◀──value──  │        │  No atomicity                    │
│  │        │──SET─────▶ │        │                                   │
│  │        │◀───OK────  │        │                                   │
│  └────────┘            └────────┘                                   │
│                                                                      │
│  With Lua Script (Single Round-trip):                               │
│  ┌────────┐            ┌────────┐                                   │
│  │ Client │───EVAL───▶ │ Redis  │                                   │
│  │        │            │ Script │  1 round-trip                    │
│  │        │            │ (atomic)│  Full atomicity                  │
│  │        │◀──result── │        │                                   │
│  └────────┘            └────────┘                                   │
│                                                                      │
│  Key Benefits:                                                       │
│  • Atomicity - entire script runs without interruption              │
│  • Reduced latency - single network round-trip                      │
│  • Complex logic - conditionals, loops, calculations                │
│  • Reusability - cache scripts with SCRIPT LOAD                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Basic Syntax

```redis
# EVAL command
EVAL "return 'Hello, World!'" 0

# With keys and arguments
EVAL "return {KEYS[1], KEYS[2], ARGV[1], ARGV[2]}" 2 key1 key2 arg1 arg2
# Returns: ["key1", "key2", "arg1", "arg2"]

# Access Redis commands via redis.call()
EVAL "return redis.call('GET', KEYS[1])" 1 mykey

# redis.call vs redis.pcall
# redis.call()  - raises error on failure
# redis.pcall() - returns error as table
```

### KEYS and ARGV

```lua
-- KEYS[n] - Keys the script will access
-- ARGV[n] - Additional arguments
-- Both are 1-indexed (Lua convention)

-- Example: Increment with limit
local key = KEYS[1]
local limit = tonumber(ARGV[1])

local current = redis.call('GET', key)
current = tonumber(current) or 0

if current < limit then
    return redis.call('INCR', key)
else
    return current
end
```

```redis
# Call the script
EVAL "local key = KEYS[1]; local limit = tonumber(ARGV[1]); local current = redis.call('GET', key); current = tonumber(current) or 0; if current < limit then return redis.call('INCR', key) else return current end" 1 counter 100
```

---

## 2. Data Types and Conversions

### Redis to Lua Type Conversion

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Type Conversions                                  │
│                                                                      │
│  Redis Type         →    Lua Type                                   │
│  ─────────────────────────────────────────────────────────────────  │
│  Integer            →    Number                                     │
│  Bulk String        →    String                                     │
│  Array              →    Table (array)                              │
│  Status Reply       →    Table {ok = "..."}                         │
│  Error Reply        →    Table {err = "..."}                        │
│  Nil                →    false (Lua boolean)                        │
│                                                                      │
│  Lua Type           →    Redis Type                                 │
│  ─────────────────────────────────────────────────────────────────  │
│  Number             →    Integer                                    │
│  String             →    Bulk String                                │
│  Table (array)      →    Array                                      │
│  Table {ok=...}     →    Status Reply                               │
│  Table {err=...}    →    Error Reply                                │
│  Boolean false/nil  →    Nil                                        │
│  Boolean true       →    Integer 1                                  │
└─────────────────────────────────────────────────────────────────────┘
```

### Working with Types

```lua
-- Handling nil values
local value = redis.call('GET', KEYS[1])
if value == false then  -- nil from Redis becomes false
    return "Key does not exist"
end
return value

-- Converting numbers
local count = redis.call('GET', 'counter')
count = tonumber(count) or 0  -- Handle nil case
return count + 1

-- Working with arrays
local members = redis.call('SMEMBERS', 'myset')
for i, member in ipairs(members) do
    -- Process each member
end

-- Return multiple values as array
return {value1, value2, value3}

-- Return status
return redis.status_reply("OK")

-- Return error
return redis.error_reply("Something went wrong")
```

---

## 3. Common Patterns

### Atomic Get-Set Operations

```lua
-- Compare and Swap (CAS)
local key = KEYS[1]
local expected = ARGV[1]
local new_value = ARGV[2]

local current = redis.call('GET', key)
if current == expected then
    redis.call('SET', key, new_value)
    return 1
else
    return 0
end
```

```redis
EVAL "local key = KEYS[1] local expected = ARGV[1] local new_value = ARGV[2] local current = redis.call('GET', key) if current == expected then redis.call('SET', key, new_value) return 1 else return 0 end" 1 mykey "old_value" "new_value"
```

### Rate Limiting

```lua
-- Token bucket rate limiter
local key = KEYS[1]
local capacity = tonumber(ARGV[1])
local refill_rate = tonumber(ARGV[2])
local now = tonumber(ARGV[3])
local requested = tonumber(ARGV[4]) or 1

-- Get current bucket state
local bucket = redis.call('HMGET', key, 'tokens', 'last_update')
local tokens = tonumber(bucket[1]) or capacity
local last_update = tonumber(bucket[2]) or now

-- Calculate token refill
local elapsed = now - last_update
local refill = elapsed * refill_rate
tokens = math.min(capacity, tokens + refill)

-- Try to consume tokens
if tokens >= requested then
    tokens = tokens - requested
    redis.call('HMSET', key, 'tokens', tokens, 'last_update', now)
    redis.call('EXPIRE', key, math.ceil(capacity / refill_rate) * 2)
    return {1, tokens}  -- Allowed, remaining tokens
else
    redis.call('HMSET', key, 'tokens', tokens, 'last_update', now)
    redis.call('EXPIRE', key, math.ceil(capacity / refill_rate) * 2)
    return {0, tokens}  -- Denied, remaining tokens
end
```

### Distributed Lock

```lua
-- Acquire lock with timeout
local key = KEYS[1]
local token = ARGV[1]
local ttl = tonumber(ARGV[2])

-- Try to acquire
local acquired = redis.call('SET', key, token, 'NX', 'PX', ttl)
if acquired then
    return 1
else
    return 0
end
```

```lua
-- Release lock (only if owner)
local key = KEYS[1]
local token = ARGV[1]

local current_token = redis.call('GET', key)
if current_token == token then
    redis.call('DEL', key)
    return 1
else
    return 0
end
```

### Leaderboard Operations

```lua
-- Update score and get rank atomically
local leaderboard = KEYS[1]
local player = ARGV[1]
local score_delta = tonumber(ARGV[2])

-- Update score
local new_score = redis.call('ZINCRBY', leaderboard, score_delta, player)

-- Get rank (0-indexed)
local rank = redis.call('ZREVRANK', leaderboard, player)

-- Get surrounding players
local start = math.max(0, rank - 2)
local neighbors = redis.call('ZREVRANGE', leaderboard, start, start + 4, 'WITHSCORES')

return {new_score, rank, neighbors}
```

### Inventory Management

```lua
-- Reserve inventory atomically
local product_key = KEYS[1]
local reservation_key = KEYS[2]
local quantity = tonumber(ARGV[1])
local reservation_id = ARGV[2]
local ttl = tonumber(ARGV[3])

-- Check available inventory
local available = tonumber(redis.call('GET', product_key)) or 0

if available >= quantity then
    -- Decrement inventory
    redis.call('DECRBY', product_key, quantity)

    -- Create reservation record
    redis.call('HSET', reservation_key, reservation_id, quantity)
    redis.call('EXPIRE', reservation_key, ttl)

    return {1, available - quantity}  -- Success, remaining
else
    return {0, available}  -- Failed, available
end
```

---

## 4. Script Management

### SCRIPT LOAD and EVALSHA

```redis
# Load script into Redis (returns SHA1 hash)
SCRIPT LOAD "return redis.call('GET', KEYS[1])"
# Returns: "a42059b356c875f0717db19a51f6aaa9161e77a2"

# Execute by SHA (faster - no script transmission)
EVALSHA a42059b356c875f0717db19a51f6aaa9161e77a2 1 mykey

# Check if script exists
SCRIPT EXISTS a42059b356c875f0717db19a51f6aaa9161e77a2
# Returns: [1] if exists, [0] if not

# Flush all scripts
SCRIPT FLUSH

# Kill running script
SCRIPT KILL
```

### Client-Side Script Management

```python
import redis
import hashlib

class ScriptManager:
    def __init__(self, redis_client):
        self.redis = redis_client
        self.scripts = {}

    def register(self, name, script):
        """Register a Lua script"""
        sha = self.redis.script_load(script)
        self.scripts[name] = {
            'sha': sha,
            'script': script
        }
        return sha

    def run(self, name, keys=None, args=None):
        """Run a registered script"""
        if name not in self.scripts:
            raise ValueError(f"Script '{name}' not registered")

        script_info = self.scripts[name]
        keys = keys or []
        args = args or []

        try:
            return self.redis.evalsha(
                script_info['sha'],
                len(keys),
                *keys,
                *args
            )
        except redis.exceptions.NoScriptError:
            # Script not in cache, reload
            sha = self.redis.script_load(script_info['script'])
            script_info['sha'] = sha
            return self.redis.evalsha(sha, len(keys), *keys, *args)


# Usage
sm = ScriptManager(redis.Redis())

# Register scripts
sm.register('incr_with_limit', '''
    local current = tonumber(redis.call('GET', KEYS[1])) or 0
    local limit = tonumber(ARGV[1])
    if current < limit then
        return redis.call('INCR', KEYS[1])
    end
    return current
''')

# Run script
result = sm.run('incr_with_limit', keys=['counter'], args=[100])
```

---

## 5. Redis 7+ Functions

### Defining Functions

```lua
-- Define function library
#!lua name=mylib

-- Helper function (local, not callable from Redis)
local function helper(x)
    return x * 2
end

-- Redis function (callable from FCALL)
redis.register_function('double_value', function(keys, args)
    local key = keys[1]
    local value = tonumber(redis.call('GET', key)) or 0
    local result = helper(value)
    redis.call('SET', key, result)
    return result
end)

-- Another function with flags
redis.register_function{
    function_name = 'readonly_func',
    callback = function(keys, args)
        return redis.call('GET', keys[1])
    end,
    flags = {'no-writes'}
}
```

```redis
# Load function library
FUNCTION LOAD "#!lua name=mylib\n\nredis.register_function('myfunc', function(keys, args) return 'hello' end)"

# Call function
FCALL myfunc 0

# List functions
FUNCTION LIST

# Delete library
FUNCTION DELETE mylib

# Dump all functions (for backup)
FUNCTION DUMP

# Restore functions
FUNCTION RESTORE <serialized-value>
```

### Functions vs Scripts

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Functions vs Scripts                              │
│                                                                      │
│  Feature              │ EVAL/EVALSHA       │ Functions (Redis 7+)   │
│  ─────────────────────┼────────────────────┼────────────────────────│
│  Persistence          │ No                 │ Yes (replicated)       │
│  Replication          │ Script in command  │ Auto-replicated        │
│  Libraries            │ No                 │ Yes                    │
│  Naming               │ SHA hash           │ Named functions        │
│  Upgrade              │ Re-register        │ FUNCTION LOAD REPLACE  │
│  Cluster              │ Per-node scripts   │ Cluster-wide           │
│                                                                      │
│  Use Functions for:                                                  │
│  • Production workloads                                             │
│  • Persistent logic                                                 │
│  • Complex libraries                                                │
│                                                                      │
│  Use EVAL for:                                                       │
│  • Quick prototyping                                                │
│  • One-off operations                                               │
│  • Pre-Redis 7 compatibility                                        │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 6. Error Handling

### Script Errors

```lua
-- Using redis.pcall for error handling
local result = redis.pcall('INCR', KEYS[1])

if type(result) == 'table' and result.err then
    -- Handle error
    return redis.error_reply("Failed to increment: " .. result.err)
end

return result
```

```lua
-- Input validation
local key = KEYS[1]
local value = ARGV[1]

if not key or key == '' then
    return redis.error_reply("ERR Key is required")
end

if not value then
    return redis.error_reply("ERR Value is required")
end

-- Safe conversion
local num = tonumber(value)
if not num then
    return redis.error_reply("ERR Value must be a number")
end

return redis.call('SET', key, num)
```

### Debugging

```redis
# Enable verbose logging
CONFIG SET lua-replicate-commands yes

# Debug script
redis-cli --ldb --eval /path/to/script.lua key1 key2 , arg1 arg2
# Opens Lua debugger

# Debug commands in debugger:
# s - step
# n - next
# c - continue
# p <var> - print variable
# b <line> - set breakpoint
# q - quit
```

```lua
-- Logging for debugging (goes to Redis log)
redis.log(redis.LOG_WARNING, "Debug message: " .. tostring(value))

-- Log levels:
-- redis.LOG_DEBUG
-- redis.LOG_VERBOSE
-- redis.LOG_NOTICE
-- redis.LOG_WARNING
```

---

## 7. Performance Considerations

### Script Execution Time

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Performance Guidelines                            │
│                                                                      │
│  ⚠ Scripts block Redis - keep them short!                           │
│                                                                      │
│  Default timeout: 5 seconds                                         │
│  lua-time-limit 5000  # in redis.conf                               │
│                                                                      │
│  If script exceeds limit:                                            │
│  • Redis starts accepting read commands only                        │
│  • SCRIPT KILL can terminate (if no writes)                         │
│  • SHUTDOWN NOSAVE if script made writes                            │
│                                                                      │
│  Optimization tips:                                                  │
│  • Use EVALSHA instead of EVAL                                      │
│  • Minimize Redis calls within script                               │
│  • Avoid loops over large datasets                                  │
│  • Use SCAN for large collections                                   │
│  • Pre-compute values when possible                                 │
└─────────────────────────────────────────────────────────────────────┘
```

### Efficient Patterns

```lua
-- Bad: Multiple calls
for i = 1, 100 do
    redis.call('LPUSH', KEYS[1], ARGV[i])
end

-- Good: Single call with multiple values
local args = {}
for i = 1, #ARGV do
    table.insert(args, ARGV[i])
end
redis.call('LPUSH', KEYS[1], unpack(args))
```

```lua
-- Bad: Loading large hash
local all_data = redis.call('HGETALL', KEYS[1])  -- Millions of fields!

-- Good: Use HSCAN for large hashes
local cursor = "0"
local results = {}
repeat
    local scan_result = redis.call('HSCAN', KEYS[1], cursor, 'COUNT', 1000)
    cursor = scan_result[1]
    local data = scan_result[2]
    for i = 1, #data, 2 do
        -- Process in chunks
    end
until cursor == "0"
```

---

## 8. Redis Cluster Considerations

### Script Key Rules

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Cluster Script Rules                              │
│                                                                      │
│  All keys must hash to the same slot!                               │
│                                                                      │
│  ✓ Valid:                                                            │
│  EVAL "..." 2 {user}:profile {user}:settings                        │
│  (Hash tag ensures same slot)                                       │
│                                                                      │
│  ✗ Invalid:                                                          │
│  EVAL "..." 2 user:1:profile user:2:profile                         │
│  (Different slots - will fail with CROSSSLOT error)                 │
│                                                                      │
│  Best Practice:                                                      │
│  • Design keys with hash tags                                       │
│  • Validate keys before script execution                            │
│  • Handle CROSSSLOT errors gracefully                               │
└─────────────────────────────────────────────────────────────────────┘
```

```lua
-- Script that works in cluster
-- All keys use same hash tag pattern

local user_id = ARGV[1]
local profile_key = "{user:" .. user_id .. "}:profile"
local settings_key = "{user:" .. user_id .. "}:settings"
local orders_key = "{user:" .. user_id .. "}:orders"

-- These all hash to same slot
local profile = redis.call('HGETALL', profile_key)
local settings = redis.call('HGETALL', settings_key)
local orders = redis.call('LRANGE', orders_key, 0, 9)

return {profile, settings, orders}
```

---

## Summary

| Feature | EVAL | EVALSHA | Functions |
|---------|------|---------|-----------|
| Script Transfer | Every call | Never | Load once |
| Persistence | No | No | Yes |
| Naming | None | SHA hash | Function name |
| Redis Version | 2.6+ | 2.6+ | 7.0+ |

---

## Best Practices

```
Script Design:
✓ Keep scripts short and focused
✓ Use EVALSHA for production
✓ Validate all inputs
✓ Handle errors with redis.pcall
✓ Use hash tags in cluster mode

Performance:
✓ Minimize Redis calls within scripts
✓ Avoid loops over unbounded data
✓ Use SCAN for large collections
✓ Pre-register scripts at startup

Debugging:
✓ Use redis.log() for debugging
✓ Test scripts in development with --ldb
✓ Monitor lua-time-limit warnings
✓ Have SCRIPT KILL ready for emergencies

Maintenance:
✓ Version control your scripts
✓ Document script purpose and parameters
✓ Use Functions for Redis 7+
✓ Test scripts after Redis upgrades
```
