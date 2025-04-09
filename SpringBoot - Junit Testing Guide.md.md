1.Spring Boot Unit Testing with Mockito
=======================================

Unit testing is a critical part of developing robust Spring Boot applications. In this tutorial, I'll show you how to write effective unit tests for all layers of your application (Controller, Service, and Repository) using JUnit 5 and Mockito.

*Methodology: Pure Unit Testing*

-   Mocks dependencies using `@Mock` (Mockito).
-   Injects the controller manually using `@InjectMocks`.
-   Uses `MockMvcBuilders.standaloneSetup()` to test the controller in isolation.
-   Does not load Spring Context → faster test execution.

*🧰 Best for:*

-   Testing controller logic or small service classes in complete isolation.
-   Fast, lightweight tests without Spring Boot overhead.

Setup and Dependencies

First, ensure you have the necessary dependencies in your `pom.xml`:
```
 <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-test</artifactId>
    <scope>test</scope>
   </dependency>

   <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-test-autoconfigure</artifactId>
    <scope>test</scope>
   </dependency>

   <dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter</artifactId>
    <version>5.10.0</version>
    <scope>test</scope>
   </dependency>
```


SpringBoot Project
```
//Controller
@RestController
@RequestMapping(path = ApplicationConstants.API_BASE + ApplicationConstants.V1 + "employee")
@Validated
@Slf4j
@RequiredArgsConstructor
public class EmployeeController {

    @Autowired
    private final EmployeeService employeeService;

    private final ObjectMapper objectMapper;

    private final EmployeeMapper employeeMapper;

    @ApiOperation("Create a new Employee or update exist ")
    @PostMapping
    public EmployeeDto save(@RequestParam(name = "id", required = false) Long id, @RequestBody @Valid EmployeeDto employeeDto) {
        return employeeService.save(employeeDto, id);
    }

    @GetMapping("/{id}")
    public EmployeeDto getEmployeeById(@PathVariable("id") Long id) {
        return employeeService.getEmployeeById(id);
    }

    @ApiOperation("Get Employee Version History By Id")
    @GetMapping("/all")
    public List<EmployeeDto> getAllEmployees() {
        return employeeService.getAllEmployees();
    }
}

//Service
@Service
@Slf4j
@AllArgsConstructor
public class EmployeeServiceImpl implements EmployeeService {

    private final ObjectMapper objectMapper;
    @Autowired
    private EmployeeRepository employeeRepository;
    @Autowired
    private EmployeeMapper employeeMapper;
    @Autowired
    private TaskScheduler taskScheduler;
    private Map<String, ScheduledFuture<?>> scheduledTasks;
    @Autowired
    private EmailUtil emailUtil;

    @Autowired
    private ApplicationUtils applicationUtils;

    @Autowired
    private EmailFeignClient emailFeignClient;

    @Override
    public Page<EmployeeSearchResultsDto> searchAllEmployee(Pageable pageRequest, EmployeeSearchDto searchCriteria) {
        return employeeRepository.employeeSearchCriteria(pageRequest, searchCriteria);
    }

    @Transactional()
    @Override
    public EmployeeDto save(EmployeeDto employeeDto, Long employeeId) {
        log.info("save start :::: employeeId:{}", employeeId);
        try {
            Boolean createRequest = StringUtils.isEmpty(employeeId) ? Boolean.TRUE : Boolean.FALSE;
            Employee employee = getEmployee(employeeId, createRequest);
            employeeMapper.toEntity(employeeDto, employee);
            Employee entity = employeeRepository.save(employee);

            if(applicationUtils.getIsEmailEnabledForUserCreation()){
                log.info("Send Mail when ever new user created, by email service");
                EmailRequestDto emailRequestDto = EmailUtil.getEmailDtoFromEmployee(employeeDto);
                emailFeignClient.sendEmail(emailRequestDto);

            }
            log.info("save Completed :::: employeeId:{}", entity.getId());
            return employeeMapper.toDto(entity);

        } catch (Exception ex) {
            log.error("Error while saving employee", ex);
            throw new BusinessException("Save Failed");
        }
    }

    @Override
    @Cacheable(value = "employees", key = "#id")
    public EmployeeDto getEmployeeById(Long id) {
        var entity = employeeRepository.findById(id).orElseThrow(() -> new EntityNotFoundException(Employee.class, id));
        return employeeMapper.toDto(entity);
    }

    @Override
    @Cacheable(value = "employees")
    public List<EmployeeDto> getAllEmployees() {
        List<Employee> employees = employeeRepository.findAll();
        return employeeMapper.mapEntityListToDtoListForEmployee(employees);
    }
}

//Repository
public interface EmployeeRepository extends RevisionRepository<Employee, Long, Long>, JpaRepository<Employee, Long>, EmployeeRepositoryCustom,
  QuerydslPredicateExecutor<Employee>,
  QuerydslBinderCustomizer<QEmployee> {

 @Override
 default void customize(QuerydslBindings bindings, QEmployee root) {
  bindings.bind(String.class).first((StringPath path, String value) -> path.containsIgnoreCase(value));
 }
}
```


MOCKITO ANNOTATIONS
-------------------

![](https://miro.medium.com/v2/resize:fit:2000/1*QTkxl2xYXQ7SuzIIMmFy3w.png)

Core Annotations

![](https://miro.medium.com/v2/resize:fit:2000/1*7yN24Q0DNIKTGiW8xOxzwQ.png)

Controller Level Annotations

![](https://miro.medium.com/v2/resize:fit:2000/1*q-0mOANOvBzGmhG4CooTVg.png)

Service Layer Annotations

![](https://miro.medium.com/v2/resize:fit:2000/1*i0yL2YnMSWhg3m0-UH8khA.png)

Repository Layer Annotations

![](https://miro.medium.com/v2/resize:fit:2000/1*kRXgZgickgOfGhnGsq-hmQ.png)

Summary

Example Unit Testing with Mockito
```
// Controller Layer Test
@WebMvcTest(EmployeeController.class)
class EmployeeControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private EmployeeService employeeService;

    @MockBean
    private EmployeeMapper employeeMapper;

    @MockBean
    private ObjectMapper objectMapper;

    @Test
    void testSaveEmployee() throws Exception {
        EmployeeDto dto = new EmployeeDto();
        dto.setName("John");

        when(employeeService.save(any(EmployeeDto.class), isNull())).thenReturn(dto);

        mockMvc.perform(post("/api/v1/employee")
                .contentType(MediaType.APPLICATION_JSON)
                .content("{"name": "John"}"))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.name").value("John"));
    }

    @Test
    void testGetEmployeeById() throws Exception {
        EmployeeDto dto = new EmployeeDto();
        dto.setId(1L);

        when(employeeService.getEmployeeById(1L)).thenReturn(dto);

        mockMvc.perform(get("/api/v1/employee/1"))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.id").value(1));
    }

    @Test
    void testGetAllEmployees() throws Exception {
        when(employeeService.getAllEmployees()).thenReturn(List.of(new EmployeeDto()));

        mockMvc.perform(get("/api/v1/employee/all"))
                .andExpect(status().isOk());
    }
}

// Service Layer Test
@ExtendWith(MockitoExtension.class)
class EmployeeServiceImplTest {

    @InjectMocks
    private EmployeeServiceImpl employeeService;

    @Mock
    private EmployeeRepository employeeRepository;

    @Mock
    private EmployeeMapper employeeMapper;

    @Mock
    private TaskScheduler taskScheduler;

    @Mock
    private EmailUtil emailUtil;

    @Mock
    private ApplicationUtils applicationUtils;

    @Mock
    private EmailFeignClient emailFeignClient;

    @Mock
    private ObjectMapper objectMapper;

    @Test
    void testSaveEmployee_success() {
        EmployeeDto dto = new EmployeeDto();
        Employee employee = new Employee();

        when(employeeMapper.toEntity(any(), any())).thenReturn(employee);
        when(employeeRepository.save(any())).thenReturn(employee);
        when(applicationUtils.getIsEmailEnabledForUserCreation()).thenReturn(true);
        when(employeeMapper.toDto(any())).thenReturn(dto);

        EmployeeDto result = employeeService.save(dto, null);

        assertNotNull(result);
        verify(emailFeignClient).sendEmail(any());
    }

    @Test
    void testGetEmployeeById_found() {
        Employee entity = new Employee();
        entity.setId(1L);

        when(employeeRepository.findById(1L)).thenReturn(Optional.of(entity));
        when(employeeMapper.toDto(entity)).thenReturn(new EmployeeDto());

        EmployeeDto dto = employeeService.getEmployeeById(1L);

        assertNotNull(dto);
    }

    @Test
    void testGetAllEmployees() {
        when(employeeRepository.findAll()).thenReturn(List.of(new Employee()));
        when(employeeMapper.mapEntityListToDtoListForEmployee(any())).thenReturn(List.of(new EmployeeDto()));

        List<EmployeeDto> employees = employeeService.getAllEmployees();

        assertEquals(1, employees.size());
    }
}

// Repository Layer Test
@DataJpaTest
class EmployeeRepositoryTest {

    @Autowired
    private TestEntityManager entityManager;

    @Autowired
    private EmployeeRepository employeeRepository;

    @Test
    void testFindById() {
        Employee emp = new Employee();
        emp.setName("Test");
        Employee saved = entityManager.persist(emp);
        entityManager.flush();

        Optional<Employee> found = employeeRepository.findById(saved.getId());

        assertTrue(found.isPresent());
        assertEquals("Test", found.get().getName());
    }
}
```


2. Spring Context-based Test with `@MockBean` + `@ContextConfiguration`
========================================================================

-   Uses Spring's testing support (`@ExtendWith(SpringExtension.class)`).
-   Loads only specific beans using `@ContextConfiguration`.
-   Injects the controller with `@Autowired`.
-   Mocks dependencies using `@MockBean` (Spring's wrapper around Mockito).
-   Still uses `MockMvcBuilders.standaloneSetup()` for HTTP-like testing.

Best for:
---------

-   Testing controller behavior with Spring dependency injection.
-   Useful when testing how beans interact or when Spring proxies (like AOP, security) are involved.

Here are Spring Context-based tests using `@MockBean` and `@ContextConfiguration` for each layer in your setup (`Controller`, `Service`, and `Repository`). These test templates use `MockMvc` and mock the necessary dependencies using Spring's test infrastructure.

![](https://miro.medium.com/v2/resize:fit:2000/1*b4kpysDbljK18x_9enJbYA.png)

Basic. Annotations

![](https://miro.medium.com/v2/resize:fit:2000/1*ZE1qp4axOhIvP-DqrMR2yA.png)

Annotations Summary

Controller Layer Test Cases

-   `@ContextConfiguration`: Tells Spring to only load the `EmployeeController` and its dependencies.
-   `@MockBean`: Used to mock out the service and mapper dependencies (`EmployeeService`, `EmployeeMapper`, `ObjectMapper`) so that only controller logic is tested in isolation.

```
package com.employee.controller;

import static org.mockito.Mockito.when;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@ExtendWith(SpringExtension.class)
@ContextConfiguration(classes = {EmployeeController.class})
public class EmployeeControllerTest {

    @Autowired
    private EmployeeController employeeController;

    @MockBean
    private EmployeeService employeeService;

    @MockBean
    private ObjectMapper objectMapper;

    @MockBean
    private EmployeeMapper employeeMapper;

    private MockMvc mockMvc;

    @BeforeEach
    void setup() {
        mockMvc = MockMvcBuilders.standaloneSetup(employeeController).build();
    }

    @Test
    void testGetEmployeeById() throws Exception {
        Long employeeId = 1L;
        EmployeeDto mockDto = new EmployeeDto();
        when(employeeService.getEmployeeById(employeeId)).thenReturn(mockDto);

        mockMvc.perform(get("/api/v1/employee/{id}", employeeId))
                .andExpect(status().isOk());
    }

    @Test
    void testGetAllEmployees() throws Exception {
        when(employeeService.getAllEmployees()).thenReturn(Collections.emptyList());

        mockMvc.perform(get("/api/v1/employee/all"))
                .andExpect(status().isOk());
    }
}
```


Service Layer Test Cases

-   Here, we're loading only the service class, mocking all collaborators like repository, mapper, utility classes.
-   Focus is on testing business logic inside the service, not external dependencies


```
@ExtendWith(SpringExtension.class)
@ContextConfiguration(classes = {EmployeeServiceImpl.class})
public class EmployeeServiceImplTest {

    @Autowired
    private EmployeeServiceImpl employeeService;

    @MockBean
    private EmployeeRepository employeeRepository;

    @MockBean
    private EmployeeMapper employeeMapper;

    @MockBean
    private EmailUtil emailUtil;

    @MockBean
    private EmailFeignClient emailFeignClient;

    @MockBean
    private TaskScheduler taskScheduler;

    @MockBean
    private ApplicationUtils applicationUtils;

    @MockBean
    private ObjectMapper objectMapper;

    @Test
    void testGetAllEmployeesReturnsEmptyList() {
        when(employeeRepository.findAll()).thenReturn(Collections.emptyList());
        when(employeeMapper.mapEntityListToDtoListForEmployee(Collections.emptyList())).thenReturn(Collections.emptyList());

        List<EmployeeDto> result = employeeService.getAllEmployees();
        assertNotNull(result);
        assertTrue(result.isEmpty());
    }

    @Test
    void testGetEmployeeByIdFound() {
        Long id = 1L;
        Employee entity = new Employee();
        when(employeeRepository.findById(id)).thenReturn(Optional.of(entity));
        when(employeeMapper.toDto(entity)).thenReturn(new EmployeeDto());

        EmployeeDto result = employeeService.getEmployeeById(id);
        assertNotNull(result);
    }
}

```


/** @DataJpaTest: Special Spring Boot annotation to test JPA repositories. It:
Configures H2 in-memory database. Scans only @Entity and @Repository beans
Disables full Spring Boot startup for faster tests

@Import(...): Used to include additional beans if required (e.g., QueryDSL setup)
**/


```
@SpringBootTest
@TestInstance(TestInstance.Lifecycle.PER_CLASS)
@TestMethodOrder(MethodOrderer.OrderAnnotation.class)
public class EmployeeRepositoryTest {

    @MockBean
    private EmployeeSearchResultMapper employeeSearchResultMapper;

    @Autowired
    private EmployeeRepository employeeRepository;

    @Test
    @Order(1)
    void testSaveAndFindEmployee() {
        Employee employee = new Employee();
        employee.setName("Test");
        employee.setCity("User");

        Employee saved = employeeRepository.save(employee);
        Optional<Employee> found = employeeRepository.findById(saved.getId());

        assertTrue(found.isPresent());
        assertEquals("Test", found.get().getName());
    }
}

```

Unit Testing with Mockito VS Spring Context-based Test
======================================================

![](https://miro.medium.com/v2/resize:fit:2000/1*QzGO0nUp7Xcub5Ri7EH_4A.png)

Diffblue --- unit test generator plugin IntelliJ
==============================================

Diffblue is an AI-powered Java unit test generator. It integrates into IntelliJ IDEA (as a plugin) or can be used via CLI or Maven. The IntelliJ plugin auto-generates JUnit tests for existing Java code, helping improve test coverage and catch bugs early.

-   Install Diffblue Cover plugin from IntelliJ Marketplace.
-   Open your Java project in IntelliJ IDEA.
-   Right-click on a Java class → `Diffblue Cover` → `Write Tests`.
-   Auto-generates JUnit tests in `src/test/java`.
-   Supports batch test generation for multiple classes.
-   Works with Spring Boot, Maven, and CLI.
-   Useful for increasing test coverage and testing legacy code.
-   Requires valid license for full features.
