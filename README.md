Infosys Java + Spring Boot Interview Questions & Detailed Answers


---

✅ Core Java

1. Explain OOP principles with examples. Object-Oriented Programming (OOP) is a paradigm centered around objects rather than actions. It helps structure programs so that properties and behaviors are bundled into individual objects.

Encapsulation: Bundles data (fields) and methods into a single unit or class. It hides internal details from the outside world and provides access via public methods (getters/setters).

Example: A BankAccount class with private balance field and public deposit()/withdraw() methods.


Inheritance: Enables a class to inherit the properties and behaviors (methods) of another class. Promotes code reuse.

Example: Dog extends Animal, where Dog inherits methods like eat() from Animal.


Polymorphism: Allows objects to take many forms.

Compile-time polymorphism: Method overloading — multiple methods with same name but different signatures.

Runtime polymorphism: Method overriding — subclass provides specific implementation of a method from its parent.


Abstraction: Hides complex internal logic and shows only the necessary parts.

Example: Java's List interface lets you use an ArrayList without knowing its internal structure.



2. Difference between HashMap, Hashtable, and ConcurrentHashMap. These are all implementations of Map interface used for key-value data storage:

HashMap: Not synchronized. Allows one null key and multiple null values. Not thread-safe.

Hashtable: Thread-safe but synchronized completely, leading to lower performance. Doesn't allow null keys or values.

ConcurrentHashMap: Thread-safe using fine-grained locking (segments or buckets). Better performance in multi-threaded environments than Hashtable.


3. What are functional interfaces in Java 8? A functional interface has exactly one abstract method and can have multiple default/static methods. They're used in lambda expressions and method references.

Example:


@FunctionalInterface
interface MyFunc {
  void execute();
}

Java 8 predefined functional interfaces: Predicate, Function, Supplier, Consumer, etc.

4. How does equals() and hashCode() work? These are used when comparing or storing objects in collections like HashMap or HashSet.

equals(): Compares actual content of objects.

hashCode(): Returns an integer representation of the object. Objects that are equal must return the same hashCode. Failing to override both properly can cause issues in hash-based collections.


5. Final, finally, and finalize

final: Prevents modification. A final variable can't be reassigned, a final method can't be overridden, and a final class can't be extended.

finally: A block used with try-catch. It executes regardless of whether an exception is thrown.

finalize(): Method called before garbage collection (deprecated in newer Java versions).


6. Thread lifecycle and synchronization A Java thread can be in one of several states: New → Runnable → Running → Blocked → Terminated. Synchronization is used to control access to shared resources and prevent race conditions.

synchronized void method() { ... }

Alternatives: ReentrantLock, Semaphore, AtomicInteger

7. What is memory leak in Java and how to prevent it? A memory leak occurs when objects are no longer needed but are not garbage collected because references to them are still held. This leads to memory exhaustion.

Prevention: Use weak references where appropriate, avoid static references to large objects, clean up listeners or observers.



---

✅ Spring Boot

1. What is the role of @SpringBootApplication? It is a convenience annotation that combines @Configuration, @EnableAutoConfiguration, and @ComponentScan. It enables auto-configuration and component scanning, making Spring Boot applications easier to set up.

2. Explain auto-configuration mechanism. Spring Boot uses @EnableAutoConfiguration (internally triggered by @SpringBootApplication) to automatically configure beans based on classpath dependencies and defined properties. It relies on META-INF/spring.factories to load configuration classes. Example: If spring-boot-starter-data-jpa is in the classpath, it automatically configures EntityManager, DataSource, etc.

3. How do you secure a REST API? Use Spring Security:

Disable CSRF (since it's a stateless REST API)

Permit /auth/login

Authenticate other endpoints using JWT

Use a custom filter (OncePerRequestFilter) to intercept requests, extract token, validate it, and set authentication in the context.


4. Explain @Component, @Service, @Repository

@Component: Generic stereotype for any Spring-managed bean.

@Service: Specialization of @Component for service-layer beans.

@Repository: DAO-layer bean; provides automatic translation of JDBC exceptions to Spring's DataAccessException.


5. Dependency injection types

Constructor injection: Preferred for immutability and testability.

Field injection: Uses reflection, hard to test.

Setter injection: Optional dependencies.


@Autowired
public MyService(MyRepository repo) { ... }

6. Use of application.properties/yaml These files store configuration for the app such as database URL, port, logging, credentials, etc. Spring supports profile-specific config like application-dev.yml and loads them using @Profile annotations or spring.profiles.active.

7. Calling external API Spring provides:

RestTemplate (blocking)

WebClient (non-blocking)


WebClient.create().get().uri("http://example.com").retrieve().bodyToMono(String.class);


---

✅ JSP / Servlets / J2EE

1. Difference between Servlet and JSP

Servlet: Java code embedded in classes, good for control logic.

JSP: HTML pages with embedded Java, compiled into Servlets by the container. Ideal use: Servlet for controller logic, JSP for views (MVC).


2. Servlet lifecycle

1. init() — Initializes servlet.


2. service() — Handles request/response.


3. destroy() — Cleanup before shutdown.



3. Session management

Cookies: Store session IDs

HttpSession: Server-side session store

URL Rewriting: Pass session ID in URL


4. RequestDispatcher Used to forward or include request to another resource.

RequestDispatcher rd = request.getRequestDispatcher("/next");
rd.forward(request, response);


---

✅ Hibernate / JPA

1. get() vs load()

get(): Immediately hits DB, returns null if not found.

load(): Returns proxy, hits DB only on access. Throws ObjectNotFoundException if missing.


2. JPA relationships Used to map relational associations:

@OneToOne

@OneToMany

@ManyToOne

@ManyToMany Specify mappedBy, cascade, fetch for control over behavior.


3. Lazy vs Eager Loading

Lazy: Delay loading until accessed (better for performance).

Eager: Load immediately (use with caution on large associations).


4. Caching in Hibernate

First Level: Enabled by default per session.

Second Level: Shared across sessions; requires provider (Ehcache, Redis). Improves performance by avoiding repeated DB calls.


5. N+1 Problem Occurs when a query loads parent entities and then fires one query per child (inefficient). Fix:

Use fetch join

Use @EntityGraph

Use batch-size configuration



---

✅ SQL / Database

1. Second highest salary

SELECT MAX(salary)
FROM employees
WHERE salary < (SELECT MAX(salary) FROM employees);

Alternative: Use DENSE_RANK/ROWNUM/ROW_NUMBER.

2. Normalization

1NF: No repeating groups, atomic columns.

2NF: No partial dependency (non-key attribute depends on part of composite key).

3NF: No transitive dependency (non-key attribute depends on another non-key attribute).


3. Indexing Indexes improve SELECT performance. Types: B-tree, Bitmap, Composite Caution: Indexes slow down write operations.

4. Join across 3 tables

SELECT * FROM Employee e
JOIN Department d ON e.dept_id = d.id
JOIN Location l ON d.loc_id = l.id
WHERE l.region = 'APAC';

5. Query Optimization

Use EXPLAIN PLAN to analyze

Add indexes to WHERE/JOIN keys

Avoid SELECT *

Use pagination



---

✅ Microservices

1. Monolith vs Microservices

Monolith: All features deployed together, tightly coupled.

Microservices: Decentralized, independently deployable components. Benefits: Scalability, independent deployments. Challenges: Complexity, network latency.


2. Communication between services

REST (HTTP)

Message Brokers (RabbitMQ, Kafka)

gRPC (binary protocol) Use FeignClient for REST in Spring Cloud.


3. Circuit Breaker Pattern

Prevent cascading failures.

Uses timeout + fallback logic.

Tools: Resilience4j (modern), Hystrix (deprecated)


@CircuitBreaker(name = "inventoryService", fallbackMethod = "fallback")

4. Configuration in Microservices

Externalize using Spring Cloud Config

Centralized config repository (Git/YAML)

Use encryption for sensitive values


5. Securing internal communication

JWT tokens between services

API Gateway with shared secrets

mTLS (mutual TLS) for critical internal APIs



---

✅ Miscellaneous

1. Writing unit tests

Use JUnit + Mockito


@Test
void testSave() {
  when(repo.save(any())).thenReturn(obj);
  assertEquals(obj, service.save(obj));
}

Use @MockBean, @WebMvcTest, @DataJpaTest where needed


2. CI/CD Pipeline

Code push triggers build → test → deploy

Use Jenkins, GitHub Actions, GitLab CI

Use Docker and Kubernetes for containerized deployment


3. Production Troubleshooting

Use logs, metrics, alerts

Tools: ELK Stack (Elasticsearch, Logstash, Kibana), Prometheus, Grafana, Zipkin for tracing

Look for memory leaks, thread dumps, error patterns


4. Communication with remote teams

Tools: Slack, Jira, Confluence, Git

Best Practices:

Daily standups, sprint planning, retros

Detailed documentation

Active async communication and regular check-ins





The document now contains in-depth explanations for all questions—no one-liners. Each answer elaborates on the concept with examples, reasoning, and practical details.

Let me know if you'd like:

Mock interview Q&A

Behavioral/HR round questions

Project-based questions and how to explain them in interviews


