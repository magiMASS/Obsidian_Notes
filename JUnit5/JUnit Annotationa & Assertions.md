
### ***Annotations Overview

***Annotations :

		@Test  
		-> This will execute the test method once the test class is running.
		@BeforeEach 
		-> This will make the method runs before each time test method runs.
		@AfterEach 
		-> This will make the method runs after each time test method runs.
		@BeforeAll 
		-> This will make the static method run once before all the before each  methods and test methods.
		@AfterAll 
		-> this will make the static methods run only once after all aftereach methods and test methods.
		@Ignore 
		-> This will make the test methods to be skipped during the tests execution

***Execution order :***

@BeforeClass
	@Before
		@Test
	@After
	@Before
		@Test
	@After
@AfterClass
		

---



### ***Assertion Explanation

***Assertions : 


			Assert.asserEquals();   
				-> This methods requires 3 parameters, 1.msg ,2.expected, 3.acutal
				-> This will assert / check for the content of the actual object to be compared with the expected objects value to be exactly equal.
			
			Assert.assertNotEquals();
				-> This methods requires 3 parameters, 1.msg ,2.expected, 3.acutal
				-> This will assert / check for the content of the actual object to be compared with the expected objects value to be not equal.
			
			Assert.assertTrue() 
				-> This methods requires 1 parameters, 1.conditional statement
				-> This methods will check conditional statements and check if it is true.
				
			Assert.assertFalse() 
				-> This methods requires 1 parameters, 1.conditional statement
				-> This methods will check conditional statements and check if it is false.
				
			Assert.assertNull()		
				-> This methods requires 1 parameters, 1.conditional statement
				-> This methods will check the object is null		
				 
			Assert.assertNotNull()	
				-> This methods requires 1 parameters, 1.conditional statement
				-> This methods will check the object is not null	
					 
			Assert.assertSame()		
				-> This methods requires 2 parameters, 1.obj1 , 2.obj2
				-> This methods will check the object is same (ie. same reference or same memory address)
				
			Assert.assertNotSame()		
				-> This methods requires 2 parameters, 1.obj1 , 2.obj2
				-> This methods will check the object is not same (ie. different reference or different memory address)
				
			Assert.asserArrayEquals(); 
				-> This methods requires 3 parameters, 1.msg ,2.expected, 3.acutal
				-> This will check for both arrays having same length and after that will chaeck each index having same value


***For next Lesson click -> [[JUnit Mockito]]

---
### ***Code Snippet


***Sample Code : 

  
@SpringBootTest  
@RunWith(SpringRunner.class)  
public class TestBefore {  
    @Mock  
    MockCrypto mockCrypto;  
    @Before  
    public void testBefore(){  
        Mockito.when(mockCrypto.encryptSign(Mockito.anyString())).thenReturn("hi hello");  
    }  
  
    @Test  
    @Ignore    public void testingBefore(){  
        String res = mockCrypto.encryptSign("hi magesh");  
        String resDecrypt = mockCrypto.decrypt("hi magesh");  
        Assert.assertEquals("testing before methods ","hi hello",res);  
        Mockito.verify(mockCrypto,Mockito.times(1)).encryptSign(Mockito.anyString());  
        Mockito.verify(mockCrypto,Mockito.times(1)).decrypt(Mockito.anyString());  
        InOrder inOrder = Mockito.inOrder(mockCrypto);  
        inOrder.verify(mockCrypto,Mockito.atLeastOnce()).encryptSign(Mockito.anyString());  
        inOrder.verify(mockCrypto,Mockito.atLeastOnce()).decrypt(Mockito.anyString());  
    }  
  
      
    @Test(timeout = 2000)  
    public void testTimeOut(){  
        try {  
            String name1 = null;  
            String name2 = "mani";  
            assertAll(  
                    () -> Assert.assertTrue("validate name" ,"vijay".equals("vijay")),  
                    () -> Assert.assertNull(name1),  
                    ()->Assert.assertNotNull(name2),  
                    ()->Assert.assertNotSame(name1,name2),  
                    ()->Assert.assertSame(name1,name2));  
  
            Thread.sleep(1000);  
  
        }catch (InterruptedException e){  
//            e.printStackTrace();  
        }  
    }  
}