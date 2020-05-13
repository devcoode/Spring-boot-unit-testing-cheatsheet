# Basic of Mockito #

## assertEquals ##

**`assertEquals(expectedResult,actualResult);`**

This method checks that the two objects are equals or not. If they are not, an AssertionError without a message is thrown. Incase if both expected and actual values are null, then this method returns equal.

**Example**

    @Test
	void calculateSum_basic() {
		SomeBusinessImpl business = new SomeBusinessImpl();
		int actualResult = business.calculateSum(new int[]{1,2,3});
		int expectedResult = 6;
		assertEquals(expectedResult,actualResult);
	}
	
	@Test
	void calculateSum_empty() {
		SomeBusinessImpl business = new SomeBusinessImpl();
		int actualResult = business.calculateSum(new int[]{});
		int expectedResult = 0;
		assertEquals(expectedResult,actualResult);
	}
	
	@Test
	void calculateSum_onevalue() {
		SomeBusinessImpl business = new SomeBusinessImpl();
		int actualResult = business.calculateSum(new int[]{5});
		int expectedResult = 5;
		assertEquals(expectedResult,actualResult);
	}

## **Note:** _Never use Stubs use **Mockito**_ ##

	@Test
	void calculateSumUsingDataService_basic() {
		SomeBusinessImpl business = new SomeBusinessImpl();

		SomeDataService dataServiceMock = mock(SomeDataService.class);
		when(dataServiceMock.retrieveAllData()).thenReturn(new int[]{1,2,3});
		business.setSomeDataService(dataServiceMock);

		int actualResult = business.calculateSumUsingDataService();
		int expectedResult = 6;
		assertEquals(expectedResult,actualResult);
	}

This can be used as in fewer line with annotation:

	@InjectMocks
	SomeBusinessImpl business;
	
	@Mock
	SomeDataService dataServiceMock;

	@Test
	public void calculateSumUsingDataService_basic() {
		when(dataServiceMock.retrieveAllData()).thenReturn(new int[]{1,2,3});
		int actualResult = business.calculateSumUsingDataService();
		int expectedResult = 6;
		assertEquals(expectedResult,actualResult);
	}

In the above code `SomeBusinessImpl business;` is the class where logic is implemented. Here we have to annotate it with `@InjectMocks`.

`SomeDataService dataServiceMock;` this is the service repository from where data should be comming. This should be annotated with `@Mock`.

And now it can we used inside `@Test` fucntion easily.

## Check for multiple values

	@Test
	public void returnDifferentValues() {
		when(mock.size()).thenReturn(5).thenReturn(10);
		assertEquals(5,mock.size());
		assertEquals(10,mock.size());
	}

Use `.thenReturn(5).thenReturn(10)` will return twice in respective order like `assertEquals(5,mock.size()); assertEquals(10,mock.size());` this.

## Use with GenericParameter ##

	@Test
	public void returnWithGenreicParameter() {
		when(mock.get(anyInt())).thenReturn("harsh");
		assertEquals("harsh",mock.get(0));
		assertEquals("harsh",mock.get(1));
	}

Use `import static org.mockito.Mockito.anyInt;` for any Integer.This `anyInt` is called an agument matcher.

![1.png](https://github.com/harsh3105/Spring-boot-unit-testing-cheatsheet/blob/master/Screenshot/1.png)

All the Agument Matcher which can be used in place of `anyInt` are shown in the image above.

## verify ##

If the method which we are testing is not returning some thing. It might be calling another function or something else. Example, `someDataService.storeSum(sum); ` then we might have to use verificataion method.

	@Test
	public void verificationBasics() {
		String value = mock.get(0);
		String value1 = mock.get(1);
		verify(mock).get(0);
		verify(mock,times(1)).get(anyInt());
		verify(mock,atLeast(1)).get(anyInt());
		verify(mock,atLeastOnce()).get(anyInt());
		verify(mock,atMost(1)).get(anyInt());
		verify(mock,never()).get(anyInt());
	}

`verify` method is used to check if the method is called or not. All the method's which can be used with verify as given in the above code.

If we want to get argument which are passes in some fuction. We have to use `ArgumentCaptor`. This can be used like the code given below.

	@Test
	public void argumentCapturing() {
		//SUT
		mock.add("SomeString");
		
		ArgumentCaptor<String> captor = ArgumentCaptor.forClass(String.class);
		verify(mock).add(captor.capture());
		
		assertEquals("SomeString",captor.getValue());
	}

If the some function is called multiple time's and your have get the value each time then, we have to use `captor.getAllValues();`. And we have to use `times` while verifying with the method `verify`.

	@Test
	public void multipleArgumentCapturing() {
		
		mock.add("SomeString1");
		mock.add("SomeString2");
		
		ArgumentCaptor<String> captor = ArgumentCaptor.forClass(String.class);
		verify(mock, times(2)).add(captor.capture());
		
		List<String> allValues = captor.getAllValues();
		assertEquals("SomeString1",allValues.get(0));
		assertEquals("SomeString2",allValues.get(1));
	}

## Spy ##

When have to use the original dependency and not to mock and want to find out what is happen with it. We can also call verify with this.

	@Test
	public void spying() {
		ArrayList arrayListSpy = spy(ArrayList.class);
		arrayListSpy.add("test0"); 
		System.out.println(arrayListSpy.get(0)); //test0
		System.out.println(arrayListSpy.size()); //1
		arrayListSpy.add("test1"); 
		arrayListSpy.add("test2");
		System.out.println(arrayListSpy.size()); //3
		when(arrayListSpy.size()).thenReturn(5);
		System.out.println(arrayListSpy.size()); //5
	}


## Links ##

* [FAQ](https://github.com/mockito/mockito/wiki/FAQ)
* [PowerMock](https://github.com/in28minutes/MockitoTutorialForBeginners/tree/master/src/main/java/com/in28minutes/powermock)
* [Important Tips](https://github.com/in28minutes/in28minutes-initiatives/blob/master/The-in28Minutes-TroubleshootingGuide-And-FAQ/quick-start.md)