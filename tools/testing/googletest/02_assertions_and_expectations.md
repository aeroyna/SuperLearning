# Assertions and Expectations

Assertions are macros that resemble function calls. You test a class or function by making assertions about its behavior.

## Fatal vs Non-Fatal

GTest provides two versions of every assertion:

1.  **`ASSERT_*` (Fatal)**: Generates a fatal failure. When it fails, it **aborts the current function**. Use this when it's pointless to continue (e.g., if checking array size fails, checking elements will cause a crash).
2.  **`EXPECT_*` (Non-Fatal)**: Generates a non-fatal failure. When it fails, the test continues. This is usually preferred because it allows you to see multiple failures in a single run.

## Basic Assertions

| Condition | Fatal | Non-Fatal | Meaning |
|:---|:---|:---|:---|
| Boolean | `ASSERT_TRUE(condition)` | `EXPECT_TRUE(condition)` | True |
| Boolean | `ASSERT_FALSE(condition)` | `EXPECT_FALSE(condition)` | False |

## Binary Comparison

| Type | Fatal | Non-Fatal |
|:---|:---|:---|
| Equal | `ASSERT_EQ(val1, val2)` | `EXPECT_EQ(val1, val2)` |
| Not Equal | `ASSERT_NE(val1, val2)` | `EXPECT_NE(val1, val2)` |
| Less Than | `ASSERT_LT(val1, val2)` | `EXPECT_LT(val1, val2)` |
| Less/Equal | `ASSERT_LE(val1, val2)` | `EXPECT_LE(val1, val2)` |
| Greater | `ASSERT_GT(val1, val2)` | `EXPECT_GT(val1, val2)` |
| Greater/Equal| `ASSERT_GE(val1, val2)` | `EXPECT_GE(val1, val2)` |

## String Comparison

To compare C-style strings (`char*`), use these specialized macros to verify string content rather than pointer addresses:

*   `ASSERT_STREQ(str1, str2)` / `EXPECT_STREQ`
*   `ASSERT_STRNE(str1, str2)` / `EXPECT_STRNE`
*   `ASSERT_STRCASEEQ(str1, str2)` (Case insensitive)

## Exception Assertions

Verify that code throws (or doesn't throw) exceptions:

```cpp
// Verifies that function() throws std::invalid_argument
EXPECT_THROW(function(), std::invalid_argument);

// Verifies that function() throws ANY exception
EXPECT_ANY_THROW(function());

// Verifies that function() throws NOTHING
EXPECT_NO_THROW(function());
```
