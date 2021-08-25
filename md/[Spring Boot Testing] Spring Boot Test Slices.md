# [Spring Boot Testing] Spring Boot Test Slices

> https://rieckpil.de/spring-boot-test-slices-overview-and-usage/

Spring Boot Test Slices allows you to write tests for specific parts of your application in isolation without bootstrapping the whole Spring Context

Technically this is achieved by creating a Spring Context with only a subset of beans by applying only specific auto-configurations.





## 1. `@WebMvcTest`

Using this annotation, you'll get a Spring Context that includes components required for testing Spring MVC parts of your application.



What's **part** of the Spring Test Context: `@Controller`, `@ControllerAdvice`, `@JsonComponent`, `Converter`, `Filter`, `WebMvcConfigurer`



What's **not part** of the Spring Test Context: `@Service`, `@Component`, `@Repository` beans



Furthermore, there is also great support if you secure your endpoints with Spring Security. The annotation will auto-configure your security rules, and if you include the Spring Security Test dependency, you can easily mock the authenticated user.



As this annotation provides a mocked servlet environment, there is no port to access your application with, e.g., a `RestTemplate`. Therefore, you rather use the auto-configured `MockMvc` to access your endpoints:



```java
@WebMvcTest(ShoppingCartController.class)
class ShoppingCartControllerTest {
 
  @Autowired
  private MockMvc mockMvc;
 
  @MockBean
  private ShoppingCartRepository shoppingCartRepository;
 
  @Test
  public void shouldReturnAllShoppingCarts() throws Exception {
    when(shoppingCartRepository.findAll()).thenReturn(
      List.of(new ShoppingCart("42",
        List.of(new ShoppingCartItem(
          new Item("MacBook", 999.9), 2)
        ))));
 
    this.mockMvc.perform(get("/api/carts"))
      .andExpect(status().isOk())
      .andExpect(jsonPath("$[0].id", Matchers.is("42")))
      .andExpect(jsonPath("$[0].cartItems.length()", Matchers.is(1)))
      .andExpect(jsonPath("$[0].cartItems[0].item.name", Matchers.is("MacBook")))
      .andExpect(jsonPath("$[0].cartItems[0].quantity", Matchers.is(2)));
  }
}
```



Usually, you mock any dependent bean of your controller endpoint using `@MockBean`.

If you write reactive applications with WebFlux, there is also `@WebFluxTest`.





## 2. `@DataJpaTest`

With this annotation, you can test any JPA-related parts of your application. A good example is to verify that a native query is working as expected.



What's **part** of the Spring Test Context: `@Repository`, `EntityManager`, `TestEntityManager`, `DataSource`

What's **not part** of the Spring Test Context: `@Service`, `@Component`, `@Controller` beans



By default, this annotation tries to auto-configure use an embedded database (e.g., H2) as the `DataSource`:



```java
@DataJpaTest
class BookRepositoryTest {
 
  @Autowired
  private DataSource dataSource;
 
  @Autowired
  private EntityManager entityManager;
 
  @Autowired
  private BookRepository bookRepository;
 
  @Test
  public void testCustomNativeQuery() {
    assertEquals(1, bookRepository.findAll().size());
 
    assertNotNull(dataSource);
    assertNotNull(entityManager);
  }
}
```



While an in-memory database might not be a good choice to verify a native query using proprietary features, you can disable this auto-configuration with:

```java
@AutoConfigureTestDatabase*(replace = AutoConfigureTestDatabase.Replace.NONE)
```

  and use, e.g., [Testcontainers](https://rieckpil.de/howto-write-spring-boot-integration-tests-with-a-real-database/) to create a PostgreSQL database for testing.





In addition to the auto-configuration, all tests run inside a transaction and get rolled back after their execution.





## 3. `@JsonTest`

What comes next is a more unknown test slice annotation that helps to test JSON serialization: `@JsonTest`



What's **part** of the Spring Test Context: `@JsonComponent`,`ObjectMapper`, `Module` from Jackson or similar components when using JSONB or GSON

What's **not part** of the Spring Test Context: `@Service`, `@Component`, `@Controller`, `@Repository`





If you have a more complex serialization logic for your Java classes or make use of several Jackson annotations:

```java
public class PaymentResponse {
 
  @JsonIgnore
  private String id;
 
  private UUID paymentConfirmationCode;
 
  @JsonProperty("payment_amount")
  private BigDecimal amount;
 
  @JsonFormat(
    shape = JsonFormat.Shape.STRING, pattern = "yyyy-MM-dd|HH:mm:ss", locale = "en_US")
  private LocalDateTime paymentTime;
 
}
```



You can use this test slice to verify the JSON serialization of your Spring Boot application:

```java
@JsonTest
class PaymentResponseTest {
 
  @Autowired
  private JacksonTester<PaymentResponse> jacksonTester;
 
  @Autowired
  private ObjectMapper objectMapper;
 
  @Test
  public void shouldSerializeObject() throws IOException {
 
    assertNotNull(objectMapper);
 
    PaymentResponse paymentResponse = new PaymentResponse();
    paymentResponse.setId("42");
    paymentResponse.setAmount(new BigDecimal("42.50"));
    paymentResponse.setPaymentConfirmationCode(UUID.randomUUID());
    paymentResponse.setPaymentTime(LocalDateTime.parse("2020-07-20T19:00:00.123"));
 
    JsonContent<PaymentResponse> result = jacksonTester.write(paymentResponse);
 
    assertThat(result).hasJsonPathStringValue("$.paymentConfirmationCode");
    assertThat(result).extractingJsonPathNumberValue("$.payment_amount").isEqualTo(42.50);
    assertThat(result).extractingJsonPathStringValue("$.paymentTime").isEqualTo("2020-07-20|19:00:00");
    assertThat(result).doesNotHaveJsonPath("$.id");
  }
}
```





## 4. `@RestClientTest`

Next comes a hidden gem when you want to test your HTTP clients with a local HTTP server.



What's **part** of the Spring Test Context: your HTTP client using `RestTemplateBuilder`, `MockRestServiceServer`, Jackson auto-configuration

What's **not part** of the Spring Test Context: `@Service`, `@Component`, `@Controller`, `@Repository`



Using the `MockRestServiceServer` you can now mock different HTTP responses from the remote system:

```java
@RestClientTest(RandomQuoteClient.class)
class RandomQuoteClientTest {
 
  @Autowired
  private RandomQuoteClient randomQuoteClient;
 
  @Autowired
  private MockRestServiceServer mockRestServiceServer;
 
  @Test
  public void shouldReturnQuoteFromRemoteSystem() {
    String response = "{" +
      "\"contents\": {"+
        "\"quotes\": ["+
          "{"+
            "\"author\": \"duke\"," +
            "\"quote\": \"Lorem ipsum\""+
          "}"+
        "]"+
      "}" +
    "}";
 
    this.mockRestServiceServer
      .expect(MockRestRequestMatchers.requestTo("/qod"))
      .andRespond(MockRestResponseCreators.withSuccess(response, MediaType.APPLICATION_JSON));
 
    String result = randomQuoteClient.getRandomQuote();
 
    assertEquals("Lorem ipsum", result);
  }
}
```

