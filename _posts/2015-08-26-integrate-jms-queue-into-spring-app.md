---
layout: post
title: Integrate JMS queue into a Spring Application

tags:
- spring boot
- jms queue
---

### Introduction 
To represent
Book App - 

![placeholder]({{ site.url }}/assets/book_manager_queue.png "Book Manager Queue")

### Dependencies

pom.xml 

```xml
<!-- JMS -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jms</artifactId>
</dependency>
<dependency>
    <groupId>javax.jms</groupId>
    <artifactId>jms-api</artifactId>
</dependency>
<dependency>
    <groupId>org.apache.activemq</groupId>
    <artifactId>activemq-core</artifactId>
    <version>${activemq-core.version}</version>
</dependency>
```

### JMS Configuration

JmsConfiguration.java

```java
@EnableJms
@Configuration
public class JmsConfiguration {

    @Autowired
    private BeanFactory springContextBeanFactory;
    
    @Bean
    public DefaultJmsListenerContainerFactory containerFactory(ConnectionFactory connectionFactory) {
        DefaultJmsListenerContainerFactory factory =
                new DefaultJmsListenerContainerFactory();
        factory.setConnectionFactory(connectionFactory);
        factory.setDestinationResolver(new BeanFactoryDestinationResolver(springContextBeanFactory));
        factory.setConcurrency("3-10");
        return factory;
    }

    @Bean
    public JmsTemplate jmsTemplate(@Qualifier("inQueueDestination") Destination defaultDestination, ConnectionFactory connectionFactory) throws JMSException {
        JmsTemplate jmsTemplate = new JmsTemplate(connectionFactory);
        jmsTemplate.setDefaultDestination(defaultDestination);
        return jmsTemplate;
    }

}
```

### Active MQ for test

application.yml

```java
jms:
  queue:
    name: in-queue
```

ActiveMqConfiguration.java

```java
@Configuration
public class ActiveMqConfiguration {

    public static final String ADDRESS = "vm://localhost";

    private BrokerService broker;

    @Bean(name="inQueueDestination")
    public Destination inQueueDestination(@Value("${jms.queue.name}") String inQueueName)
            throws JMSException {
        return new ActiveMQQueue(inQueueName);
    }

    @PostConstruct
    public void startActiveMQ() throws Exception {
        broker = new BrokerService();
        // configure the broker
        broker.setBrokerName("activemq-broker");
        broker.setDataDirectory("target");
        broker.addConnector(ADDRESS);
        broker.setUseJmx(false);
        broker.setUseShutdownHook(false);
        broker.start();
    }

    @PreDestroy
    public void stopActiveMQ() throws Exception {
        broker.stop();
    }

    @Bean
    public ConnectionFactory connectionFactory() {
        return new ActiveMQConnectionFactory(ADDRESS + "?broker.persistent=false");
    }
}
```

### Listening to queue messages

InQueueListener.java

```java
@Component
public class InQueueListener  implements Loggable{

    private final BookService bookService;

    @Autowired
    public InQueueListener(BookService bookService) {
        this.bookService = bookService;
    }

    @JmsListener(containerFactory = "containerFactory", destination = "inQueueDestination",
                 selector = "Operation = 'Create'")
    public void processCreateBookMessage(BookDTO book) throws JMSException{
        bookService.createNew(book);
    }

    @JmsListener(containerFactory = "containerFactory", destination = "inQueueDestination",
                 selector = "Operation = 'Update'")
    public void processUpdateBookMessage(BookDTO book) throws JMSException{
        bookService.update(book.getIsbn(), book);
    }

    @JmsListener(containerFactory = "containerFactory", destination = "inQueueDestination",
                 selector = "Operation = 'Delete'")
    public void processDeleteBookMessage(BookDTO book) throws JMSException{
        bookService.delete(book.getIsbn());
    }

}
```

## Testing - sending messages to queue

![placeholder]({{ site.url }}/assets/book_manager_queue_mock.png "Book Manager Queue with Mock Service layer")


```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = Application.class, loader = SpringApplicationContextLoader.class)
@WebAppConfiguration
public class InQueueListenerIntegrationTest {

    @Autowired(required = false)
    private JmsTemplate jmsTemplate;

    @Autowired
    private InQueueListener inQueueListener;


    @Autowired(required = false)
    @Qualifier("inQueueDestination")
    private Destination inQueueDestination;

    @Mock
    private BookService mockBookService;

    @Captor
    private ArgumentCaptor<BookDTO> bookArgumentCaptor;

    @Captor
    private ArgumentCaptor<String> isbnArgumentCaptor;
    
    @Before
    public void setUp(){
        MockitoAnnotations.initMocks(this);
        ReflectionTestUtils.setField(inQueueListener, "bookService", mockBookService);
    }
    
```

```java

    @Test
    public void testSendCreateBookMessage(){
        BookDTO book =  new BookDTO("isbn", "title", "author");
        jmsTemplate.convertAndSend(inQueueDestination, book, Message -> {
            return OperationHeader.CREATE.applyToMessage(Message);
        });
        // verify
        verify(mockBookService).createNew(bookArgumentCaptor.capture());
        assertEquals(book.getIsbn(), bookArgumentCaptor.getValue().getIsbn());
        assertEquals(book.getTitle(), bookArgumentCaptor.getValue().getTitle());
        assertEquals(book.getAuthor(), bookArgumentCaptor.getValue().getAuthor());
    }

    @Test
    public void testSendUpdateBookMessage(){
        BookDTO book =  new BookDTO("isbn", "title", "author");
        jmsTemplate.convertAndSend(inQueueDestination, book, Message -> {
            return OperationHeader.UPDATE.applyToMessage(Message);
        });
        // verify
        verify(mockBookService).update(isbnArgumentCaptor.capture(),bookArgumentCaptor.capture());
        assertEquals(book.getIsbn(), isbnArgumentCaptor.getValue());
        assertEquals(book.getIsbn(), bookArgumentCaptor.getValue().getIsbn());
        assertEquals(book.getTitle(),bookArgumentCaptor.getValue().getTitle());
        assertEquals(book.getAuthor(),bookArgumentCaptor.getValue().getAuthor());
    }

    @Test
    public void testSendDeleteBookMessage(){
        BookDTO book =  new BookDTO("isbn", "title", "author");
        jmsTemplate.convertAndSend(inQueueDestination, book, Message -> {
            return OperationHeader.DELETE.applyToMessage(Message);
        });
        // verify
        verify(mockBookService).delete(book.getIsbn());
    }
       
```

### Summary
Link to repository with branch

T.