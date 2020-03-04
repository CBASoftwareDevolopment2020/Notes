# Symbol Tables and Search Trees

04-03-2020

## Symbol Tables

```java
interface SymbolTable<K,V> {
    V get(K key);
    int getSize
    default boolean contains(K key) {
        return get(key) != null;
    }
    default boolean isEmpty() {return getSize() == 0;}
}

interface MutableSymbolTable<K,V> extends SymbolTable<K,V> {
    void put(K key, V value);
    default void delete(K key) {put(key, null);}
}
```

### Familiar Symbol Tables

-   Java/Kotlin: Map<K,V>
-   C# IDictionary<K,V>
-   Javascript objects {}
-   Python dictionaries
-   Tries
-   Binary Search Trees
-   Sorted Lists with Binary Search

### Properties of equality

-   Reflective <img src="https://render.githubusercontent.com/render/math?math=\forall x \in \mathbb{M} \Leftrightarrow x=x">
-   Symetric <img src="https://render.githubusercontent.com/render/math?math=\forall x,y \in \mathbb{M} \Leftrightarrow x=y \rightarrow y=x">
-   Transitive <img src="https://render.githubusercontent.com/render/math?math=\forall x,y,z \in \mathbb{M} \Leftrightarrow x=y \wedge y=z \rightarrow x=z">

### Keys

All keys i symbol tables must:

-   support the equality operation, in Java that means:
-   overriding `boolean equals(Object other)`
-   often overriding in `hashCode()`

All keys in ordered symbol tables must:

-   support the equality operation
-   support ordering
    -   by implementing the Comparable interface
    -   or being alphabetic
    -   or otherwise...

### Properties of ordering

-   Reflective <img src="https://render.githubusercontent.com/render/math?math=\forall x \in \mathbb{M} \Leftrightarrow x \leq x">
-   Antisymetric <img src="https://render.githubusercontent.com/render/math?math=\forall x,y \in \mathbb{M} \Leftrightarrow x \leq y \wedge y \leq x \rightarrow y=x">
-   Transitive <img src="https://render.githubusercontent.com/render/math?math=\forall x,y,z \in \mathbb{M} \Leftrightarrow x \leq y \wedge y \leq z \rightarrow x \leq z">

### Alphabetic

```java
public interface Alphabetic {
    int getIndex();
    Alphabetic getNext();
    boolean hasMore();
}

class StringKey implements Alphabetic {
    private final String text;
    private int position = 0;
    StringKey (String text) {this.text = text;}
    @Override
    public int getIndex() {
        return text.charAt(positition) - ’A’;
    }
    @Override
    public Alphabetic getNext() {
        positition ++;
        return this;
    }
    @Override
    public boolean hasMore() {
        return positition<text.length();
    }
}
```

## Binary Searches

## Binary Search Trees
