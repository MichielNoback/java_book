# Inner classes

While we have done so almost always, there is a  misconception about how many classes can exist in one source file.
**_There can be only one public top-level class per source file._**
A source file can, however, have many static and non-static inner classes, non-public classes, anonymous inner classes and interface implementers.

Here is an example showing some inner class types.

```java
package snippets.oop;

//top level public class
public class Outer {
    //inner anonymous implementer of inner interface
    private Usable usable = new Usable() {
        public void use() {
            System.out.println("doing it this way");
        }
    };
    //idem, same interface
    private Usable usableTwo = new Usable() {
        public void use() {
            System.out.println("doing it that way");
        }
    };
    //non-anonymous inner class instance
    private Usable usable3 = new UsableUsable();

    public static void main(String[] args) {
        Outer outer = new Outer();
        outer.usable.use();
        outer.usableTwo.use();
        outer.usable3.use();
        outer.useInners();
    }

    public void useInners() {
        Inner i = new Inner();
        i.doIt();
    }

    /* A non-static inner class that needs an instance of Outer to exist.
    * Usually it is extremely tightly bound to the Outer class logic*/
    private class Inner {
        public void doIt() {
            System.out.println("doing it inner");
            //access to instance fields!
            usableTwo.use();
        }
    }

    /*
    * A static (although not explicity stated) interface
     */
    private interface Usable {
        public void use();
    }

    /*static implementer*/
    private static class UsableUsable implements Usable {
        @Override
        public void use() {
            System.out.println("doing it the other way");
        }
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
- Use an inner class when it is tightly bound to the Outer class, and there is no reason for existence outside the outer class. 
- Use an anonymous inner class when there is no scenario of reusability
- only use a non-static inner class when it should be bound to an instance of the outer class, else use a static inner class (a nested class)
