# Unit Test with Spring Boot. #

### To test a controller ###

First step is to use `@WebMvcTest(HelloWorldController.class)` and `@RunWith(SpringRunner.class)` over the test class. 

```
@RunWith(SpringRunner.class)
@WebMvcTest(HelloWorldController.class)
public class HelloWorldControllerTest {


}

```

When we create a `@WebMvcTest()` it creates a bean of  `MockMvc mockMvc;` which can be used with `@Autowired`.

    @Autowired
	private MockMvc mockMvc;


An example of testing HelloWorld Controller

    @Test
	public void HelloWorld_basic() throws Exception {
		
        // Request Builder
		RequestBuilder requestBuilder = MockMvcRequestBuilders
				.get("/helloworld")
				.accept(MediaType.APPLICATION_JSON);

        // Call the URI
		MvcResult result = mockMvc.perform(requestBuilder).andReturn();

        // Check the result as string
		assertEquals("hello world",result.getResponse().getContentAsString());
		
	}

## Json Asserts ##

    String actualResponse = "{\"id\":1,\"name\":\"Ball\",\"price\":10,\"quantity\":100}";
	
	@Test
	public void jsonAssert_StrictTrue_ExactMatchExceptForSpaces() throws JSONException {
		String expectedResponse = "{\"id\": 1, \"name\":\"Ball\", \"price\":10, \"quantity\":100}";
		JSONAssert.assertEquals(expectedResponse, actualResponse, true);
	}

	@Test
	public void jsonAssert_StrictFalse() throws JSONException {
		String expectedResponse = "{\"id\": 1, \"name\":\"Ball\", \"price\":10}";
		JSONAssert.assertEquals(expectedResponse, actualResponse, false);
	}

	@Test
	public void jsonAssert_WithoutEscapeCharacters() throws JSONException {
		String expectedResponse = "{id:1, name:Ball, price:10}";
		JSONAssert.assertEquals(expectedResponse, actualResponse, false);
	}

In `jsonAssert_StrictTrue_ExactMatchExceptForSpaces()` this function the strict is true then it will be matching the reponse exactly.

In `jsonAssert_StrictFalse()` this function the strict is false then it may or may not be exactly matching the response. But the value of each parameter should match the reponse exactly the same.

In `jsonAssert_WithoutEscapeCharacters()` this function we can see that we can use the string without escape character like this `String expectedResponse = "{id:1, name:Ball, price:10}";`. This is easy to read.

## Write a Unit test mocking the Business layer ##

Create the mock bean of the service.

    @MockBean
	private ItemBusinessService businessService;

And then mock it when used.

    @Test
	public void itemFromBusinessService_basic() throws Exception {
		
		when(businessService.retrieveHardcodedItem()).thenReturn(
				new Item(2,"item2",10,100));
		
		RequestBuilder requestBuilder = MockMvcRequestBuilders
				.get("/item-from-business-service")
				.accept(MediaType.APPLICATION_JSON);
		
		MvcResult result = mockMvc.perform(requestBuilder)
				.andExpect(status().isOk())
				.andExpect(content().json("{name:item2,id: 2,price:10,quantity:100}"))
				.andReturn();
    }
		
In the above code , `when(businessService.retrieveHardcodedItem()).thenReturn(new Item(2,"item2",10,100));` , here we mock the data that when `businessService.retrieveHardcodedItem()` this function is called then is should return a new Item. Else other code is same.

## **Unit test for Web, Business and Data layer** ##

* ### _Web Layer or the Controller_ ###

It is creating a mock of the business layer to test the controller.

    @Test
	public void retrieveAllItems_basic() throws Exception {

	    when(businessService.retrieveAllItems()).thenReturn(
				Arrays.asList(new Item(2,"item2",10,100),new Item(3,"item3",20,200)));
		
		RequestBuilder requestBuilder = MockMvcRequestBuilders
				.get("/all-items-from-database")
				.accept(MediaType.APPLICATION_JSON);
		
		MvcResult result = mockMvc.perform(requestBuilder)
				.andExpect(status().isOk())
				.andExpect(content().json("[{name:item2,id:2,price:10,quantity:100},{name:item3,id:3,price:20,quantity:200}]"))
				.andReturn();
		
	}

* ### _Business Layer or the Controller_ ###

To run test on business layer we don't require the spring framework. We have to use Mockito here in this case. Example is show below.

    @RunWith(MockitoJUnitRunner.class)
    public class ItemBusinessServiceTest {
	
        @InjectMocks
        private ItemBusinessService business;
        
        @Mock
        private ItemRepository repo;
        
        
        @Test
        public void abc() {
            when(repo.findAll()).thenReturn(
                    Arrays.asList(new Item(2,"item2",10,100),new Item(3,"item3",20,200)));
            List<Item> items = business.retrieveAllItems();
            assertEquals(1000,items.get(0).getValue());
            assertEquals(4000,items.get(1).getValue());
		
	    }
    }

* ### _Data Layer or the Controller_ ###

By using the `@DataJpaTest` first the data.sql is ran and all the data is pushed to H2 data base and the testing is done.

```
@RunWith(SpringRunner.class)
@DataJpaTest
public class ItemRepositoryTest {
	
	@Autowired
	private ItemRepository repo; 
	
	@Test
	public void testFindAll() {
		List<Item> items = repo.findAll();
		assertEquals(3,items.size());
	}

}

```

## Integration Test using @SpringBootTest ##

* ItemControllerIT

```
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment=WebEnvironment.RANDOM_PORT)
public class ItemControllerIT {
	
	@Autowired
	private TestRestTemplate restTemplate;
	
	@Test
	public void contextLoads() throws JSONException {
		String response = this.restTemplate.getForObject("/all-items-from-database", String.class);
		JSONAssert.assertEquals("[{id:100001},{id:100002},{id:100003}]", response, false);
	}

}

```

## Hamcrest

```
@Test
public void learning() {
		List<Integer> numbers = Arrays.asList(12,15,45);
		
		assertThat(numbers, hasSize(3));
		assertThat(numbers, hasItems(12,45));
		assertThat(numbers, everyItem(greaterThan(10)));
		assertThat(numbers, everyItem(lessThan(100)));
		
		assertThat("", isEmptyString());
		assertThat("ABCDE", containsString("BCD"));
		assertThat("ABCDE", startsWith("ABC"));
		assertThat("ABCDE", endsWith("CDE"));
				
	}
```

## AssertJ ##

```
@Test
	public void learning() {
		List<Integer> numbers = Arrays.asList(12,15,45);
		
		//assertThat(numbers, hasSize(3));
		assertThat(numbers).hasSize(3)
						.contains(12,15)
						.allMatch(x -> x > 10)
						.allMatch(x -> x < 100)
						.noneMatch(x -> x < 0);
		
		assertThat("").isEmpty();
		assertThat("ABCDE").contains("BCD")
						.startsWith("ABC")
						.endsWith("CDE");		
	}
```

## JsonPath ##

```
	@Test
	public void learning() {
		String responseFromService = "[" + 
				"{\"id\":10000, \"name\":\"Pencil\", \"quantity\":5}," + 
				"{\"id\":10001, \"name\":\"Pen\", \"quantity\":15}," + 
				"{\"id\":10002, \"name\":\"Eraser\", \"quantity\":10}" + 
				"]";
		
		DocumentContext context = JsonPath.parse(responseFromService);
		
		int length = context.read("$.length()");
		assertThat(length).isEqualTo(3);
		
		List<Integer> ids = context.read("$..id");

		assertThat(ids).containsExactly(10000,10001,10002);
		
		System.out.println(context.read("$.[1]").toString());
		System.out.println(context.read("$.[0:2]").toString());
		System.out.println(context.read("$.[?(@.name=='Eraser')]").toString());
		System.out.println(context.read("$.[?(@.quantity==5)]").toString());
    }
```