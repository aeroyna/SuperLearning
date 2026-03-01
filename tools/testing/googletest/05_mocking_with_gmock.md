# Mocking with GoogleMock (gMock)

gMock is bundled with GTest and allows you to create mock objects for testing interactions (e.g., verifying a function calls an API or Database).

## 1. The Interface
Pure virtual classes are excellent candidates for mocking.

```cpp
class Database {
public:
    virtual ~Database() {}
    virtual bool Login(const std::string& user, const std::string& password) = 0;
    virtual int GetUserId(const std::string& user) = 0;
};
```

## 2. Define the Mock
Inherit from the interface and use `MOCK_METHOD`.

```cpp
#include <gmock/gmock.h>

class MockDatabase : public Database {
public:
    MOCK_METHOD(bool, Login, (const std::string&, const std::string&), (override));
    MOCK_METHOD(int, GetUserId, (const std::string&), (override));
};
```

## 3. Set Expectations
Use `EXPECT_CALL` to define behavior and verify calls.

```cpp
using ::testing::Return;
using ::testing::_;

TEST(SystemTest, LoginSuccess) {
    MockDatabase db;
    
    // Expect Login to be called once with "admin" and any password, return true
    EXPECT_CALL(db, Login("admin", _))
        .Times(1)
        .WillOnce(Return(true));

    System sys(&db);
    sys.Start("admin", "1234");
}
```

## Key Matchers
*   `_ (Underscore)`: Matches anything.
*   `Eq(val)`: Matches val.
*   `Gt(val)`: Greater than val.
*   `Return(val)`: Specifies return value.
*   `Throw(exception)`: Throws exception.
