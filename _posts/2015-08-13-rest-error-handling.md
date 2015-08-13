---
layout: post
title: Error Handling in REST API

tags:
- rest devtools
- rest error handling
- rest API
- error handling

---

### Error handling in REST
Error handling is an important part of designing an API, the
usability of your API heavily depends on the granularity of the possible error outcomes (and how they are documented). <br/>
With REST we have Http status codes to differentiate between the error cases: <strong>4xx Client error</strong> and <strong>5xx Server error</strong>, 
see <a href="http://www.restapitutorial.com/httpstatuscodes.html">Http Status Codes</a>.

It is important to use status codes correctly for example use the status 404 for an error when a resource was not found instead of any other 4xx codes.

Most of the time you would like to return more data, especially when your Rest API serves the requests
of a UI client which is eager to show a relevant error message with details.<br/>
For example: <blockquote>The book with isbn '0684801469' was not found.</blockquote> (Sry Ernest). 

In such cases we should use an error response entity together with the error status code. 

### Using an ErrorDTO
The solution I am going to introduce is used in my favourite (and only) example application, the <a href="https://github.com/tamaslang/book-inventory-boot" target="_blank">book-inventory-boot</a>.<br/>
It is using my rest-devtools library that is available in maven central but the solution itself can be implemented without the library as well. <br/>
In the book inventory app a REST API is provided to manage books and a possible error is that a book was not found with the given isbn. 
Another type of error can be searching for a book with a non UTF-8 encoded string.

The first case is a trivial 404 (NOT&#95;FOUND) and  the second one is a 400 (BAD&#95;REQUEST) which points clearly out that you should not repeat the request with the same parameters.   

#### Classes for error handling in the library (rest-devtools)
The RestError class stores the details: the error code, the message and the http status. 

```java
public final class RestError {
    private String errorCode;
    private String errorMessage;
    private HttpStatus httpStatus;
    
    /* accessors */
}
```

An interface is defined to support the translation to RestError model.

```java
public interface TranslatableToRestError {
    RestError toRestError();
}
```

The runtime exception is subclassed to represent any exception happening in our REST API layer. <br/>
It contains a map "errorParams" to be able to pass on any additional information (e.g.: isbn for the book was not found).<br/>
Note that it implements the TranslatableToRestError interface so it can store the RestError model.
 
```java
public class RestException extends RuntimeException implements TranslatableToRestError {

    private RestError restError;

    @Override
    public RestError toRestError() {
        return restError;
    }

    private Map<String, Object> errorParams;

    public RestException(RestError restError, Map<String, Object> params) {
        this(restError);
        this.errorParams = params;
    }

    public RestException(RestError restError) {
        super(restError.getErrorMessage());
        this.restError = restError;
    }

    public Map<String, Object> getErrorParams() {
        return errorParams;
    }

}
```

Finally the ExceptionHandler controller advice is defined to catch and handle RestExceptions

```java
@ControllerAdvice
public class RestExceptionHandler extends ResponseEntityExceptionHandler implements Loggable {
  
  @ExceptionHandler(RestException.class)
  @ResponseBody
  public ResponseEntity<ErrorDTO> handleRestException(RestException ex){
      logger().debug("Handling rest exception", ex);
      /* RestUtils is used to instantiate an ErrorDTO */
      return new ResponseEntity<ErrorDTO>(
        RestUtils.createErrorDTOFromRestError(ex.toRestError(), ex.getErrorParams()), 
        ex.toRestError().getHttpStatus()
      );
  }
  
  /* catch other exceptions... */

}
```

#### Classes for error handling in the application
An enum contains all application specific errors; The code as enum key, the error message and the http status code as fields. 

```java
public enum RestErrors implements TranslatableToRestError {
    BOOK_NOT_FOUND("The book was not found.", HttpStatus.NOT_FOUND),
    URL_ENCODING_NOT_SUPPORTED("Url encoding not supported.", HttpStatus.BAD_REQUEST);


    RestErrors(String errorMessage, HttpStatus httpStatus){
        this.httpStatus=httpStatus;
        this.errorMessage = errorMessage;
    }

    private String errorMessage;

    private HttpStatus httpStatus;

    public HttpStatus getHttpStatus() {
        return httpStatus;
    }

    public String getErrorMessage() {
        return errorMessage;
    }

    @Override
    public RestError toRestError(){
        return new RestError(this.name(), errorMessage,httpStatus);
    }
}
```

Finally in the controller the appropriate exception is thrown with the details when a book is not found:

```java
@Service
public class BookServiceImpl implements Loggable, BookService {

    @Override
    public BookDTO findByIsbn(String isbn){
        logger().debug("Find by isbn '{}'",isbn);
        Book retrievedBook = bookRepo.findOne(isbn);
        if(retrievedBook == null){
            throw new RestException(RestErrors.BOOK_NOT_FOUND.toRestError(),
                    RestUtils.createParams(PNV.toPNV("isbn",isbn)));
        }
        return bookToBookDTO.apply(retrievedBook);

    }
    
```

The REST API call will return with status 404 and the following JSON payload:

```json
    {
      "errorCode":"BOOK_NOT_FOUND",
      "errorMessage":"The book was not found.",
      "params":{
        "isbn":"not-existing-isbn"
      }
    }
```

<br/>
Here is the full class diagram with all the classes participating in the error handling:
![placeholder]({{ site.url }}/assets/rest-error-handling-uml.png "Error handling classes")

<br/>
And the RestExceptionHandler defined as @ControllerAdvice:
![placeholder]({{ site.url }}/assets/rest-exception-handler-uml.png "Rest Exception Handler")

See the complete example in <a href="https://github.com/tamaslang/book-inventory-boot" target="_blank">book-inventory-boot</a> repository 
and check out <a href="http://httpstatusdogs.com/">this site</a> to memorize http status codes.

Happy error handling,<br/>
T.