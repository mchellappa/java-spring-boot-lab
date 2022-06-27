# Cupcake Shop Spring Boot App
As part of solutioning for the Cup Cake shop, this app will: 
1. Expose a web endpoint to add a Cup Cake Order. 
1. Expose a web endpoint to retrieve a Cup Cake Order.

## Lab Steps
1. Clone the project.
1. git clone https://github.com/JHDevOps/JHUniversityWeek_java-spring-boot-lab.git
1. Open the code in favorite IDE/Spring STS.


## Lets get Coding
We will write all the code under src folder and the tests under test folder. You will notice the main class is already created.
Lets think how we will develop and what will be in this app. We will need 
- a controller
- a service interface 
- message models for Order and return model might be needed
- a repository extending JPA repository



Lets create/implement package `repository` under `src\main\java\com.jh.uniweek.cupcakeshop`.Inside the package name `CupCakeOrderRepository`.

```
package com.jh.uniweek.cupcakeshop.repository;

import org.springframework.data.jpa.repository.JpaRepository;
import com.jh.uniweek.cupcakeshop.model.Order;

public interface CupCakeOrderRepository extends JpaRepository<Order, Long>{

}

```

Lets implement package `service` under `src\main\java\com.jh.uniweek.cupcakeshop`. Implement an Interface `ICupCakeOrderService` inside the package.

```
List<Order> getOrders();
Order getOrder(Long orderId);
Order addOrder(Order order);
String deleteOrder(Order order);

```



Let us create model classes, 

Implement package `model` under `src\main\java\com.jh.uniweek.cupcakeshop`. Implement class `Order`  inside `model`. 

Order class

```
package com.jh.uniweek.cupcakeshop.model;

import java.math.BigDecimal;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.Table;

import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;
import lombok.ToString;


@Getter
@Setter
@AllArgsConstructor
@NoArgsConstructor
@ToString
@Builder
@Entity
@Table(name = "orders")
public class Order {

	@Id
    @GeneratedValue(strategy = GenerationType.AUTO)
	private long id;
	@Column(name = "orderNumber", nullable = false)
    private Integer orderNumber;
    @Column(name = "cupcake", nullable = false)
    private String cupcake;
    @Column(name = "customer", nullable = false)
    private String customer;
    @Column(name = "price", nullable = false)
    private BigDecimal price;

}

```

Now we can implement the service class,
Implement the Interface as Class `CupCakeOrderServiceImpl`

```

@Autowired
private CupCakeOrderRepository cupCakeOrderRepository;

@Override
public List<Order> getOrders() {
    return cupCakeOrderRepository.findAll();
}

@Override
public Order getOrder(Long orderId) {
    Order order = cupCakeOrderRepository.findById(orderId)
                .orElseThrow();
            return order;
}

@Override
public Order addOrder(Order order) {
    return cupCakeOrderRepository.save(order);
}

@Override
public String deleteOrder(Order order) {
    cupCakeOrderRepository.delete(order);
    return "success";
}


```

Finally lets create the web api endpoint that we need to expose to take in the order with Controller.
Create a package under `src\main\java\com.jh.uniweek.cupcakeshop` with name `controller.` Create a class `CupCakeOrderController`in that package.


Lets add the dependency

```
@Autowired
ICupCakeOrderService cupCakeOrderService;

```

Lets add the controller endpoints

```
@GetMapping("/orders")
public List<Order> getCapCakeOrders() {
    return cupCakeOrderService.getOrders();
}

@GetMapping("/orders/{id}")
public Order getCupCakeOrder(@PathVariable(value = "id") Long orderId) {
    return cupCakeOrderService.getOrder(orderId);
}

@PostMapping("/orders")
public Order saveCapCakeOrder(@RequestBody Order order) {
    return cupCakeOrderService.addOrder(order);
}

@DeleteMapping("/orders/{id}")
public Map <String,Boolean > deleteCapCakeOrder(@PathVariable(value = "id") Long orderId)
{
    Order order = cupCakeOrderService.getOrder(orderId);

    cupCakeOrderService.deleteOrder(order);
    Map < String, Boolean > response = new HashMap < > ();
    response.put("deleted", Boolean.TRUE);
    return response;
}
```

To write a test follow a Given - When - Then thought process.
we have, 
- Given - you have a Order Request
- When - you invoke the API Endpoint
- Then - you get a success Order sent status from your service

Lets Write the test case, In the created a Class within `src\test\java\com.jh.uniweek.cupcakeshop` folder and there is  `CupCakeOrderControllerTest` class

Add the dependency Injection

```
@Autowired private MockMvc mockMvc;
@MockBean private ICupCakeOrderService cupCakeOrderService;

```

Lets Include a positive Test scenario

```

@Test
public void postOrder_ReturnsResponseWithSuccessCode() throws Exception {

Order order = getTestOrder();

String serializedMessageRequest = serializeModel(order);

Mockito.when(cupCakeOrderService.addOrder(any(Order.class))).thenReturn(order);

performPostWithBody(serializedMessageRequest)
    .andExpect(status().isOk())
    .andExpect(
        MockMvcResultMatchers.jsonPath("$.orderNumber").value(333))
    .andExpect(MockMvcResultMatchers.jsonPath("$.customer").value("John"));
}

private Order getTestOrder() {
Order order 
    = Order.builder().cupcake("vanilla").customer("John").orderNumber(333).price(new BigDecimal(10.01)).build();
return order;
}

```

Lets Include a negative Test scenario

```

@Test
public void postOrder_ReturnsResponseBadRequest() throws Exception {

    Order order = getTestOrder();
    order.setOrderNumber(null);
    String serializedMessageRequest = serializeModel(order);

    performPostWithBody(serializedMessageRequest).andExpect(status().isBadRequest());
}


private String serializeModel(Order order) throws JsonProcessingException {
ObjectWriter writer = new ObjectMapper().writer();
return writer.writeValueAsString(order);
}

private ResultActions performPostWithBody(String requestBody) throws Exception {
MockHttpServletRequestBuilder postRequest = buildEventPostRequest(requestBody);
return mockMvc.perform(postRequest);
}

private static MockHttpServletRequestBuilder buildEventPostRequest(String stringContent) {
return MockMvcRequestBuilders.post("/api/v1/orders")
    .content(stringContent)
    .accept(MediaType.APPLICATION_JSON)
    .contentType(MediaType.APPLICATION_JSON);
}


```


Run your test case and see it passes :)

## Parameterize the properties
We can parameterize the properties such as database configurations outside the code. This would help in changing them without impacting the code.
Inside `src/resources/application.properties` add below parameters,

```
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=password
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
spring.h2.console.enabled=true

```

## Build the code and run the application
1. Build your app. In terminal navigate to the project folder then run `mvn clean install`

## Run the application
1. In terminal navigate to the project folder then run `mvn spring-boot:run`
1. Navigate to H2 database to see the application in action.
1. Open the url endpoint in the Chrome/Edge.
    http://localhost:8080/h2-console/login.jsp?jsessionid=383b6efca153fb507048dc1202d89c43
1. Use the password as "password"






