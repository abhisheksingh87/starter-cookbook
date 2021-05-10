+++
date = "2021-01-29T14:45:27-06:00"
title = "Spring Boot Testing best practices"
summary = "Testing strategies recommendations."
tags = ["application development", "database", "datasource", "hibernate", "spring-data", "spring", "spring-boot", "testing", "junit"]
taxonomy = ["TEST-DRIVEN-DEVELOPMENT"]
weight = 4
+++

## Context
This recipe consolidates general recommendations to keep in mind while writing spring boot tests.

## Prerequisite

* Junit 5
* Mockito
* Spring-Boot-Test

## F.I.R.S.T Principles of Testing:
1. F - Fast
2. I - Independent
3. R - Repeatable
4. S - Self Validating
5. T - Timely

## Isolate the functionality to be tested by limiting the context of loaded frameworks/components.

1. As a first step in creating tests, it is sufficient to use jUnit without loading any additional frameworks.  **You only need to annotate your test with @Test**.
   In the code snipped below, there is no database interactions, and CustomerRepository loads data from the classpath.

    ```java
    public class CustomerServiceTest {
    
        private CustomerService customerService = new CustomerService();
    
        @Test
        public void testFindById() {
             //given
            Customer expectedCustomer = Customer.builder()
                                                .customerId("123")
                                                .firstName("alex")
                                                .lastName("smith")
                                                .build();
            //then
            assertEquals(expectedCustomer, CustomerRepository.findById("123"));
        }
    }
    ```

1. As a next step, consider adding _mock_ frameworks, like **mockito** if you have some interactions with external resources.

    ```java
    @ExtendsWith(MockitoExtension.class)
    @TestInstance(Lifecycle.PER_CLASS)
    public class CustomerServiceTest {
    
        private CustomerService customerService;
    
        @Mock
        private CustomerRepository customerRepository;
    
        @BeforeAll
        public void init() {
            customerService = new CustomerService(customerRepository);
        }
    
        @Test
        public void testFindById() {
            //given
            Customer expectedCustomer = Customer.builder()
                                                .customerId("123")
                                                .firstName("alex")
                                                .lastName("smith")
                                                .build();
            //when
            when(customerRepository.findById("123")).thenReturn(expectedCustomer);
            Customer actualCustomer = CustomerService.findById("id");
            
            //then        
            verify(customerRepository, times(1)).findById(any(Customer.class));
            assetEquals(actualCustomer, expectedCustomer);
        }
    }
    ```

## Load _slices_ of functionality when [testing spring boot applications](https://spring.io/blog/2016/04/15/testing-improvements-in-spring-boot-1-4).

1.  **@SpringBootTest** annotation loads whole application, but it is better to limit Application Context only to a set of spring components that participate in test scenario, by listing them in annotation declaration.

    ```java
    @ExtendWith(SpringExtension.class)
    @SpringBootTest(classes = {CustomerRepository.class, CustomerService.class})
    public class CustomerServiceWithRepoTest {
    
        @Autowired
        private CustomerService customerService;
    
        @Test
        public void testFindById() {
            //given
            Customer expectedCustomer = Customer.builder()
                                                .customerId("123")
                                                .firstName("alex")
                                                .lastName("smith")
                                                .build();
            //when
            Customer actualCustomer = CustomerService.findById("id");
    
            //then
            assetEquals(actualCustomer, expectedCustomer);
        }
    }
    ```

1. **@DataJpaTest** only loads @Repository spring components, and will greatly improve performance by not loading @Service, @Controller, etc.

    ```java
    @ExtendWith(SpringExtension.class)
    @DataJpaTest
    public class MapTests {
    
        @Autowired
        private CustomerRepository repository;
    
        @Test
        public void testfindByfirstName() {
             //given
            Customer expectedCustomer = Customer.builder()
                                                .customerId("123")
                                                .firstName("mark")
                                                .lastName("smith")
                                                .build();
            //when
            Customer actualCustomer = repository.findByFirstName("mark");
            
            //then
            assertThat(actualCustomer).isEqualTo(expectedCustomer);
        }
    }
    ```

## Running Database related tests gotchas.
1. Sometimes, `Table Already Exist` exception is thrown when testing with H2 database. This is an indication that H2 is not cleared between test invocations (because Application Context is Cached?). This behavior could occur if multiple qualifying schema-.sql files are located in the classpath.
   It is a good practice to mock the beans that are involved in db interactions, and turn off spring boot test db initialization for the spring profile that tests runs.  Please strongly consider this when testing Controllers. 
   Alternatively, you can try to declare your table creation DDL in schema.sql files as `CREATE TABLE IF NOT EXISTS`.

    ```properties
    spring.datasource.initialize=false
    
    spring.autoconfigure.exclude=org.springframework.boot.autoconfigure.orm.jpa.HibernateJpaAutoConfiguration,\
        org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration,\
        org.springframework.boot.autoconfigure.jdbc.DataSourceTransactionManagerAutoConfiguration,\
        org.springframework.boot.autoconfigure.data.web.SpringDataWebAutoConfiguration
    ```
## API/Controller tier testing.

1. Use **@WebMvcTest** to test rest APIs exposed through Controllers.  Only list controllers that are being tested.
   _Note: Spring beans used by controller should be mocked._ **@WebMvcTest** is only going to scan the controller you've defined and the MVC infrastructure
    ```java
    @ExtendWith(SpringExtension.class)
    @WebMvcTest(CustomerServiceController.class)
    public class CustomerServiceControllerTests {
    
        @Autowired
        private MockMvc mvc;
    
        @MockBean
        private CustomerService CustomerService;
    
        @Test
        public void testFindById() {
            //given
             Customer expectedCustomer = Customer.builder()
                                                .customerId("123")
                                                .firstName("mark")
                                                .lastName("smith")
                                                .build();
            given(this.CustomerService.findById("123")).willReturn(expectedCustomer);
            
            //then
            this.mvc.perform(get("/customer/123")
                .accept(MediaType.JSON)
                .andExpect(status().isOk());
        }
    }
    ```

## References
- [The up to date documentation on Spring Boot testing](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-testing.html)
