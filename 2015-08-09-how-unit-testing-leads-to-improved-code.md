## How unit testing leads to improved code
In Test Driven Development, the purpose of unit testing is to help us design our classes and not just to validate the correctness of our code. In this article I want to demonstrate how unit testing forces us to write better code, with help of an example. I will use Mockito for mocking.

First, let me define the problem domain I’ll be using in the example. Suppose we have an online booking portal where customers make reservations for travel or accommodation. Whenever a new reservation is created, its details are added to an XML which is kept at some location. Periodically, we need to fetch all the reservations that have been created in our system and send for printing.

A novice implementation of this class might look like this:

```java
public class ReservationPrinter {
  
  public int printBatchReservation() {
    String reservationsXml = fetchReservationsFromLocation();
    // XMLTools = some utility class to read XMLs
    Node reservations = XMLTools.parseXml(reservationsXml);
    NodeList reservationList = XMLTools.getNodeListForXPath(reservations, "/list/reservation");
    if(reservationList == null) return 0;
    int printCount = 0;

    for(Node reservation : reservationList) {
      String phoneNr = XMLTools.getNodeForXPath(reservation, "/customer/phone");
      String customerName = XMLTools.getNodeForXPath(reservation, "/customer/name");
      String reservationId = XMLTools.getNodeForXPath(reservation, "/id");
      
      if(isPhoneNrValid(phoneNr)) {
        sendForPrinting(reservationId, customerName, phoneNr);
        printCount++;
      }
    }
    return printCount; //returns number of reservations printed
  }
  
  public void sendForPrinting(String id, String name, String phoneNr) { .. }
  
  private String fetchReservationsFromLocation() {
     String url = ‘some network url’;
     …  
  }
  
  private boolean isPhoneNrValid(String phoneNr) { .. }
}
```
Inside `printBatchReservation` method, a private method `fetchReservationsFromLocation` is called that fetches the reservation XML by making a network call. The XML is parsed to get a list of reservations. Before printing a reservation, a validation check is performed on one of its attributes, say, telephone number. `printBatchReservation` returns the count of reservations that were sent for printing.

Before writing unit tests, we need to decide what do we actually want to test in this class. For the purpose of this article, I’ll skip `sendForPrinting`. So there are two things that will make good subjects for unit testing:

* From any given XML, are all valid reservations are sent for printing?
* Does `isPhoneNrValid` work correctly?

Private methods can’t be stubbed or unit tested directly (because they can’t be invoked outside of their class). With `printBatchReservation`, there is a different problem altogether. To test a function we need to be able to treat it like a black box, to which we pass some input and then validate the output against some expectation. But in this class, the method `printBatchReservation` is fetching the XML via a private helper method. If we can find a way to stub this method, we can then supply our own XML, against which we can perform our tests. Because private methods are invisible outside their classes, stubbing  `printBatchReservation` is out of the question. Making it public just for the sake of testing will break encapsulation.

But what if we move this method to a separate class dedicated solely for fetching reservations? This way not only do we make our `ReservationPrinter` class testable, we also separate the responsibility to make the network call to its own dedicated class.

```java
public class ReservationRetriever
{
  public String fetchReservationsFromLocation() 
  {
     //String url = "http://localhost?8080/reservationService";
     
    System.out.println("!!!----ReservationRetriever----Making database/network call...  ");
    // implementation
  }
}
```
This way `ReservationRetriever` can take care of networking and we can run integration tests on it, while `ReservationPrinter` is now more focused on printing reservations.

```java
public class ReservationPrinter {
  private ReservationRetriever retriever;
  
  public ReservationPrinter(ReservationRetriever retriever)
  {
    this.retriever. = retriever;  
  }
  
  public int printBatchReservation() {
    String reservationsXml = retriever.fetchReservationsFromLocation();
    // XMLTools = some utility class to read XMLs
    Node reservations = XMLTools.parseXml(reservationsXml);
    NodeList reservationList = XMLTools.getNodeListForXPath(reservations, "/list/reservation");
    if(reservationList == null) return 0;
    int printCount = 0;

    for(Node reservation : reservationList) {
      String phoneNr = XMLTools.getNodeForXPath(reservation, "/customer/phone");
      String customerName = XMLTools.getNodeForXPath(reservation, "/customer/name");
      String reservationId = XMLTools.getNodeForXPath(reservation, "/id");
      
      if(isPhoneNrValid(phoneNr)) {
        sendForPrinting(reservationId, customerName, phoneNr);
        printCount++;
      }
    }
    return printCount; //returns number of reservations printed
  }
  
  public void sendForPrinting(String id, String name, String phoneNr) { .. }
  
  private String fetchReservationsFromLocation() {
     String url = ‘some network url’;
     …  
  }
  
  private boolean isPhoneNrValid(String phoneNr) { .. }
}
```
Now we have a private member of type `ReservationRetriever` in our printer class. Notice that instead of instantiating `ReservationRetriever` object inside `ReservationPrinter`, the object is being passed to the constructor. If the printer class instantiated the retriever object internally, then we would still face our original problem. In order to stub `fetchReservationsFromLocation` method, so that we can perform tests against our own test XML, we need to be able to mock the `ReservationRetriever` object. So it is better if printer class accepts the retriever object from outside instead of creating it itself. That way we can pass it a mocked retriever object during testing. This concept of passing the object instead of instantiating it within the class is nothing but **Dependency Injection**. This is how dependency injection makes testing easy.

With our dependency separated nicely that we can easily mock it, we can now proceed with writing our test class. With Mockito as our mocking framework, the code will look something like this:

```java
import static org.mockito.Mockito.*;
import org.junit.*;

public class ReservationPrinterTest
{
  @Mock
  private ReservationRetriever mockedReservationRetriever;

  @InjectMocks
  private ReservationPrinter reservationPrinter;

  @Test
  public void allValidReservationsArePrinted() 
  {
    String testXml = "test XML with valid details of two reservations";
    
    doNothing().when(reservationPrinter).sendForPrinting(anyString(), anyString(), anyString(), anyString());
    when(mockedReservationRetriever.fetchReservationsFromLocation()).thenReturn(testXml); //
    
    reservationPrinter.printBatchReservation();
    
    Assert.assertTrue(reservationPrinter.printBatchReservation() == 2);    
  }
}
```
How to write test for the private method that validates phone number? One option is to test it indirectly via the test method we just wrote. In the test XML we can add details of a reservation with invalid phone number. We expect that this reservation will not be sent for printing and therefore will not be added in the print count returned by `printBatchReservation`. So by making assertions on print count, we can verify if the private method is working correctly or not.

Another option would be to move the phone number validator to a separate utility class and make it public static. This way, the method can be reused throughout our application.

Taking a look back, we started out with a single class and ended up with two classes and a possible addition to a utility class. Writing unit tests forced us to change our design to better adhere to the single responsibility principle. Test driven development follows a cycle of **code, test, refactor, test…** If we find it difficult to write tests against our code, it is an indication that the code is probably too convoluted and we need simplify it. Not only does it improves the testability of the code, but it also makes it easy to maintain.
