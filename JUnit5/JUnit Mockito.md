

### ***Description

			-> Mockito is a mocking frame work used in Unit Testing. 
			-> THis will create a fake object (mocks)
			->controls their behaviour
			-> verifies interactions

### ***Advantage : 

				->The execution is faster compared to integration testing.
				->this avoid spring context loads
				->mocks the bean and create fake objects so the real method does not execute.
				-> avoid calling repository with db interactions instead mocking the result

### ***Rules:

			-> Mockito rules are when()  method allows one mock object  and then we can mock the result each time the mocked method has been called , we can setup multipe returns this way ( by adding more thenReturn() )

---

### ***Code Snippet


@Mock  
MockCrypto mockCrypto;  
@Before  
public void testBefore(){  
    Mockito.when(mockCrypto.encryptSign(Mockito.anyString())).thenReturn("hi hello");  
}  
  
@Test  
@Ignore  
public void testingBefore(){  
    String res = mockCrypto.encryptSign("hi magesh");  
    String resDecrypt = mockCrypto.decrypt("hi magesh");  
    Assert.assertEquals("testing before methods ","hi hello",res);  
    Mockito.verify(mockCrypto,Mockito.times(1)).encryptSign(Mockito.anyString());  
    Mockito.verify(mockCrypto,Mockito.times(1)).decrypt(Mockito.anyString());  
    InOrder inOrder = Mockito.inOrder(mockCrypto);  
    inOrder.verify(mockCrypto,Mockito.atLeastOnce()).encryptSign(Mockito.anyString());  
    inOrder.verify(mockCrypto,Mockito.atLeastOnce()).decrypt(Mockito.anyString());  
}


---




### ***Junit Mockito Features

***1. Mocking Objects
		@Mock -> will take a copy of the object and create a fake object
	Example 
			List<> list = Mockito.mock(List.class);
			list.add("Hello");
			verify(list).add("Hello");
		
 ***2. Stubbing Method Calls
			-> Defines what a mocked method should return
	Example
				when(userService.getUserName()).thenReturn("John");
				when(service.getData()).thenReturn("A", "B", "C");  // returns in order calls 1st call "A" , 2nd call "B", 3rd onwards "C"
				
***3. Verifying Method Calls
			-> verify the method has been called
	Example 
				Mockito.verify(mockCrypto,atLeastOnce()).encryptSign(Mockito.anyString());  
				Mockito.verify(mockCrypto,atMost(5)).encryptSign(Mockito.anyString());  
				Mockito.verify(mockCrypto,times(4)).encryptSign(Mockito.anyString());

***4. Argument Matchers
			-> Used when the argument value is not important.
	Example 
				when(repo.findUser(anyInt())).thenReturn(user);
Common matchers:

- `any()`

- `anyString()`

- `anyInt()`

- `eq()`

- `contains()`
