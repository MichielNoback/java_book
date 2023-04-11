# Encapsulation revisited

In this post, I want to revisit one of the most basic of Object Oriented Programming (OOP) concepts: encapsulation.

It is a principle my students often fail to see the use for. And I must admit some 
of my colleagues as well – especially those who are really into Python. They are of the opinion that NOTHING should be hidden. Just be careful when programming. 

Well, my response is: some people also argue that Evolution is just a theory that you can simply disagree with!

So what does encapsulation exactly mean? My good friend Wikipedia tells me that 
“It allows selective hiding of properties and methods in an object by building 
an impenetrable wall to protect the code from accidental corruption“. Key here 
is the word **_corruption_** of course. Let’s look at an example in code. Note: of 
course there are Javadoc comments missing, and some other issues with the code 
can be discussed, but I have taken the liberty to keep things simple for the 
purpose of demonstrating encapsulation.

Here is class `SequenceManager`, the main class of a GUI application managing 
biological sequences:

```java

package encapsulation;


public final class SequenceManager {

    public static void main(final String[] args) {
        SequenceManager mainObject = new SequenceManager();
        mainObject.start();
    }
    
    private void start() {
        SequenceCollection sc = new SequenceCollection();
        sc.addSequence(new Sequence("Gene 1", "GAATTC", "gi|12345"));
        sc.addSequence(new Sequence("Gene 2", "GAATTC", "gi|98765"));
        
        MyCoolGUIPanel guiPanel = new MyCoolGUIPanel(sc);
        guiPanel.showSequenceCollection();
        
        //also add to other panels of the GUI
    }
}
```

Nothing funny going on here. A SequenceCollection is created and stocked (of course, in real life this will come from some file or database), and subsequently passed to some GUI panels for displaying and managing. 
The `Sequence` class looks quite like any POJO you will ever meet:

```java

package encapsulation;

public class Sequence {
    private String name;
    private String sequence;
    private final String id;

    public Sequence(String name, String sequence, String id) {
        this.name = name;
        this.sequence = sequence;
        this.id = id;
    }

    public Sequence(String name, String id) {
        this.name = name;
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getSequence() {
        return sequence;
    }

    public void setSequence(String sequence) {
        this.sequence = sequence;
    }

    public String getId() {
        return id;
    }
}
```

The only noteworthy thing here is the omission of a setter for the Sequence ID, 
and the id member being final (often a good choice for identifiers of bean 
objects).

Now, the collection class managing our sequences:  

```java

package encapsulation;
import java.util.HashMap;

public class SequenceCollection {
    private HashMap<String, Sequence> sequences;
    
    /**
     * constructor instantuiates the collection
     */
    public SequenceCollection() {
        this.sequences = new HashMap<>();
    }
    
    /**
     * adds a sequence to the collection
     * @param seq 
     */
    public void addSequence(Sequence seq) {
        this.sequences.put(seq.getId(), seq);
    }
    
    /**
     * getter for the sequence hash
     * @return sequences
     */
    public HashMap<String, Sequence> getSequences() {
        return this.sequences;
    }
}
```

This class contains the origin of a world of pain. The class seems to be simple 
enough, providing a method to add a sequence and a getter offering a ways to 
iterate over the collection. Can you see why this is the origin of a world of 
pain? No? Let’s have a look at the GUI JPanel displaying some aspect of this 
collection (yes I know swing is becoming a bit outdated for GUIs).

```java
package encapsulation;
import javax.swing.JPanel;

public class MyCoolGUIPanel extends JPanel{
    
    private SequenceCollection seqColl;

    public MyCoolGUIPanel(SequenceCollection sc) {
        this.seqColl = sc;
    }
    
    public void showSequenceCollection() {
        /*
        * panel logic here
        */
    }
    
    private void removeButtonClicked(String seqID) {
        //the remove sequence button was clicked so a sequence is removed
        seqColl.getSequences().remove(seqID);
    }
}
```

So what happens when somebody decides to remove a sequence from the collection? 
The getter on the SequenceCollection object is called to access the underlying 
Collection class, and the wretched object is removed.  

WITHOUT ANYONE ELSE BEING AWARE OF THIS!  

Other JPanels, the SequenceCollection object itself – nobody knows a Sequence 
object has met its maker! Having a getter offering access to a Collection like 
what we have seen in the SequenceCollection class and the HashMap is like dropping 
your bank card onto the floor of a bar with the pin code written on the back. 
You can bet somebody is going to take the invitation to trash your account.

So what is the solution? Exactly, encapsulate your inner workings. There are 
several simple ways to do this. Here is the simplest one:

```java
public Map<String, Sequence> getSequences() {
    return Collections.unmodifiableMap(sequences);
}

```

The Javadocs have this to say about the `Collections.unmodifiableMap` method: 
“Returns an unmodifiable view of the specified collection. This method allows 
modules to provide users with “read-only” access to internal collections. 
Query operations on the returned collection “read through” to the specified 
collection, and attempts to modify the returned collection, whether direct or 
via its iterator, result in an UnsupportedOperationException.” So this line

`seqColl.getSequences().remove(seqID);`  

will throw such an Exception. What I like slightly better through is an iterator:  

```java
public Iterator<Sequence> getSequenceIterator() {
    return this.sequences.values().iterator();
}
```

Or, when you really want to have the joy of the Java for(each) loop:

```java
public List<Sequence> getSequenceList() {
    return new ArrayList<Sequence>(this.sequences.values());
}
```

All these strategies will keep your data safe, but the last two also hide 
the inner workings of your SequenceCollection class so these should be 
preferred.  
