
# Dependency Injection in Spring

In software engineering, dependency injection (DI) is a technique in which an object receives other objects that it depends on. Together with its elaborate collection of annotations, dependency injection is the heart of the Spring framework. 

We will be using the debugger to explore the concept of DI. The same project of the previous tutorial will be used to go through this tutorial. The previously used Bird class will also be used, with minor modifications (an ID field was added).

As example, a simple Bird "database" will be used as data layer to make available to the application.

This is the `Bird` class:

```java
package nl.bioinf.model;

import java.util.Objects;

public class Bird {
    private Long id;
    private String name;
    private String status;

    public Bird(Long id, String name, String status) {
        this.id = Objects.requireNonNull(id);
        this.name = Objects.requireNonNull(name);
        this.status = status;
    }

    public Long getId() {
        return id;
    }

    public String getName() {
        return name;
    }

    public String getStatus() {
        return status;
    }

    @Override
    public String toString() {
        return "Bird{" +
                "name='" + name + '\'' +
                ", status='" + status + '\'' +
                '}';
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;

        Bird bird = (Bird) o;

        return getId() != null ? getId().equals(bird.getId()) : bird.getId() == null;
    }

    @Override
    public int hashCode() {
        return getId() != null ? getId().hashCode() : 0;
    }
}
```

Create a package named `dao`. In it, create an interface named BirdsRepository:

```java
package nl.bioinf.dao;

import nl.bioinf.model.Bird;

public interface BirdsRepository {
    void add(Bird bird);
    Bird findByName(String name);
    Bird findById(Long id);
}
```

Create a sub-package of `dao` named `dummy`.  In this package, create a class named `BirdsRepositoryDummyImpl` that implements the interface. It uses the Java Streams api that may be new to you. Try to figure out what is going on:

```java
package nl.bioinf.dao.dummy;

//imports omitted

public class BirdsRepositoryDummyImpl implements BirdsRepository {
    private Map<Long, Bird> birds = new HashMap<>();

    @Override
    public void add(Bird bird) {
        Objects.requireNonNull(bird);
        this.birds.put(bird.getId(), bird);
    }

    @Override
    public Bird findByName(String name) {
        final List<Bird> found = this.birds
                .values()
                .stream()
                .filter(b -> b.getName().equals(name))
                .collect(Collectors.toList());
        assert(found.size() <= 1);
        if (found.isEmpty()) {
            return null;
        } else {
            return found.get(0);
        }
    }

    @Override
    public Bird findById(Long id) {
        if (this.birds.containsKey(id)) {
            return birds.get(id);
        } else {
            return null;
        }
    }
}
```

Note that returning `null` from a method is generally considered bad practice. You should use the Optional class for such cases. However, for simplicity's sake I keep things as they are. Also note there are more sophisticated ways of "mocking" your data layer.

Let's first create some tests to see whether it works.

```java
package nl.bioinf.dao.dummy;

//imports omitted

class BirdsRepositoryDummyImplTest {
    private BirdsRepository birdsRepository;
    private Bird bird;

    @BeforeEach
    void setupDatabase() {
        this.birdsRepository = new BirdsRepositoryDummyImpl();
        this.bird = new Bird(1234L, "Steppe eagle", "extremely rare");
    }

    @Test
    void findByName_OK() {
        birdsRepository.add(this.bird);
        Bird found = birdsRepository.findByName("Steppe eagle");
        assertEquals(this.bird, found);
    }

    @Test
    void findByName_Null() {
        birdsRepository.add(this.bird);
        Bird found = birdsRepository.findByName("Tawny Eagle");
        assertNull(found);
    }

    @Test
    void findById() {
        birdsRepository.add(this.bird);
        Bird found = birdsRepository.findById(this.bird.getId());
        assertEquals(this.bird, found);
    }
}
```

It passes. Note that, when we want to test the application, we will want to run these tests no matter what the service layer implementation looks like. This is called an integration test, and it also uses Spring DI. We'll return to good testing practices in the Spring framework later.

Now let's consume the service. Here is a REST controller that we want to use to serve birds from the data layer:

```java
package nl.bioinf.webcontrol;

//imports omitted

@RestController
@RequestMapping(value="/birds")
public class BirdsController {

    @GetMapping(value = "/{id}")
    public Bird getBirdById(@PathVariable(value="id") Long id) {
        //I need to get hold of a BirdsRepository implementation here

        throw new UnsupportedOperationException("not implemented yet");
    }
}
```

As you can see, in order to serve birds, a reference to a `BirdsRepository` implementation needs to be present. The `BirdsController` class has a dependency on a `BirdsRepository` implementation. There are several ways to satisfy this dependency. Whichever technique you choose, **_never make a class dependent on an implementation; always use abstractions_**.

The dependency could be passed into the constructor or into a setter. But which object will be responsible for passing that dependency? Alternatively, you may have learned patterns like these:

```java
BirdsRepository repo = BirdsRepositoryDummyImpl.getInstance()
//OR, more loosely coupled and thus better
BirdsRepository repo = BirdsRepositoryFactory.getInstance();
```

Which makes the `BirdsController` class dependent on a factory class (or worse, a specific implementation). A general guideline in OO design is that you want to have as few dependencies as possible: **_loose coupling_** is the term for that (see [wikipedia](https://en.wikipedia.org/wiki/Loose_coupling)).

Supporting loose coupling is exactly what the Spring framework is build for: _**Wiring the objects that make up the backbone of your application with the objects they depend upon**_.

Let's have a look at the Spring way of doing this.

First, we'll add a `@Component` annotation to the `BirdsRepositoryDummyImpl` class.

```java
@Component
public class BirdsRepositoryDummyImpl implements BirdsRepository {
    public BirdsRepositoryDummyImpl() {
        //add a single bird to be able to serve
        this.birds.put(1111L, new Bird(1111L, "Long-eared owl", "fairly common"));
    }
    //rest of code omitted
```
What this `@Component` annotation does is telling the Spring container that this class should be instantiated as a component of the application.
Once you've done that, you should never take that responsibility back: never use `new BirdsRepositoryDummyImpl()` in your code. Spring is now responsible for that.
Now, whenever your request an object of type `BirdsRepository` you will get the single instance of this class that Spring instantiates.

For testing purposes, I already add a single bird within the constructor. I know, this is not the official way to do this. But I want to stay focussed on the topic here, which is dependency injection.

Next, we'll tell the Spring container we'll be needing a component of type `BirdsRepository`, using the `@Autowired` annotation.

```java
@RestController
@RequestMapping(value="/birds")
public class BirdsController {

    @Autowired
    private BirdsRepository birdsRepository;

    @GetMapping(value = "/{id}")
    public Bird getBirdById(@PathVariable(value="id") Long id) {
        return birdsRepository.findById(id);
    }
}
```

Go to the endpoint [http://localhost:8080/birds/1111](http://localhost:8080/birds/1111) and you should receive this:

```json
{
    "id": 1111,
    "name": "Long-eared owl",
    "status": "fairly common"
}
```

I used the interface type, not the implementation, to get hold of a `BirdsRepositoryDummyImpl` instance. Spring will look up all implementations and if there is only one, it will instantiate that class. If there are more implementations (or none), it will throw an error. More on how to solve that later.

This is the simplest form of Autowiring; IntelliJ suggests it is not the best way. Alternatively, I could have used Autowiring of the constructor, like below.
In some cases, autowiring a field gives `null` whereas autowiring the constructor works. See this [blog post](https://reflectoring.io/constructor-injection/) for a discussion on pros and cons of different types of DI.

```java
@RestController
@RequestMapping(value="/birds")
public class BirdsController {

    private final BirdsRepository birdsRepository;

    @Autowired
    public BirdsController(BirdsRepository birdsRepository) {
        this.birdsRepository = birdsRepository;
    }

    @GetMapping(value = "/{id}")
    public Bird getBirdById(@PathVariable(value="id") Long id) {
        return birdsRepository.findById(id);
    }
}
```

Finally, I could also have used setter-based injection:

```java
@RestController
@RequestMapping(value="/birds")
public class BirdsController {

    private BirdsRepository birdsRepository;

    @Autowired
    private void setBirdsRepository(BirdsRepository repo) {
        this.birdsRepository = repo;
    }

    @GetMapping(value = "/{id}")
    public Bird getBirdById(@PathVariable(value="id") Long id) {
        return birdsRepository.findById(id);
    }
}
```

Perhaps surprisingly, the setter can be made `private`.

This concludes the three ways of doing dependency injection using the spring framework.

Use this technique to build the backbone of your application. Once you've annotated a class as a `@Component` or other related ones such as `@Service`, `@DataSource`, **_never ever construct them yourself_**.

Note that this is primarily used for classes of which the application needs only a single instance; you are not likely going to annotate your data objects in this way.

