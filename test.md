MOCKITO CORE ANNOTATIONS & UTILITIES
------------------------------------

| Annotation / Class | Description |
| --- | --- |
| `@Mock` | Creates a mock of a dependency. |
| `@InjectMocks` | Injects mocks into the class being tested. |
| `@Spy` | Wraps a real object, allows stubbing specific methods. |
| `@Captor` | Captures method arguments passed to mocks. |
| `Mockito.when()` | Defines behavior of a mock method. |
| `Mockito.verify()` | Checks whether a mock method was called. |
| `Mockito.any()` / `anyString()` / `eq()` | Matchers used in stubbing or verification. |
| `Mockito.doReturn()` | Used with `@Spy` or void methods. |
| `assertThrows()` | Used to verify exceptions. |
