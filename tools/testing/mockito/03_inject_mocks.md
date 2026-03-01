# Dependency Injection with @InjectMocks

Mockito can automatically inject mocks into the class under test.

## Example

```java
@ExtendWith(MockitoExtension.class) // Enable Annotations
class UserServiceTest {

    @Mock
    UserRepository userRepo;      // 1. Create Mock

    @Mock
    EmailService emailService;    // 2. Create Mock

    @InjectMocks
    UserService userService;      // 3. Create Real Instance and inject 1 & 2

    @Test
    void testRegister() {
        userService.register("user");
        verify(userRepo).save(any());
        verify(emailService).sendWelcomeEmail(any());
    }
}
```
*   **`@Mock`**: Creates a mock.
*   **`@InjectMocks`**: Creates an instance of the class and tries to inject the mocks into it (via constructor, setter, or field injection).
