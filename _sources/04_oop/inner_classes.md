# Inner classes

While we have done so almost always, there is a  misconception about how many classes can exist in one source file.
**_There can and must be a single top-level public class per source file._**
A source file can, however, have many static and non-static inner classes, non-public classes, anonymous inner classes and interface implementers, and lambda's of course.

Here is an example showing some inner class types.

```java
package snippets.oop;

//top level public class -- public is mandatory here
public class Outer {

    /* A non-static inner class that needs an instance of Outer to exist.
    * Usually it is extremely tightly bound to the Outer class logic. */
    private class Inner {
        public void doIt() {
            System.out.println("doing it inner");
            // It has access to instance fields of the outer class!
            usableTwo.use();
        }
    }

    /*
    * A static (although not explicitly stated) interface
     */
    private interface Usable {
        public void use();
    }

    /* static implementer of inner interface */
    private static class UsableUsable implements Usable {
        @Override
        public void use() {
            System.out.println("doing it the other way");
        }
    }

    /* an inner anonymous implementer of inner interface attached to an instance variable */
    private Usable usableOne = new Usable() {
        public void use() {
            System.out.println("doing it this way");
        }
    };

    /* idem, same interface */
    private Usable usableTwo = new Usable() {
        public void use() {
            System.out.println("doing it that way");
        }
    };

    /* a non-anonymous inner class instance */
    private Usable usableThree = new UsableUsable();

    public static void main(String[] args) {
        Outer outer = new Outer();
        outer.usableOne.use();
        outer.usableTwo.use();
        outer.usableThree.use();
        outer.useInners();
    }

    public void useInners() {
        Inner i = new Inner();
        i.doIt();
    }
} 
```

outputs 

<pre class="console_out">
doing it this way
doing it that way
doing it the other way
doing it inner
doing it that way
</pre>

Some general rules on the use of inner classes:  

- Use an inner class when it is tightly bound to the Outer class, and there is no reason for existence outside the outer class (for instance, the class `Map.Entry` of the Java Map interface). 
- Use an anonymous inner class when there is no scenario of reusability in other contexts.
- Only use a non-static inner class when it should be bound to an instance of the outer class, else use a static inner class (a nested class).
