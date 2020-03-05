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

array is sorted

time complexity: O(log n)

### Exponential Search

time complexity: O(log p)

### Interpolation Search

time complexity: O(log log n)

## Binary Search Trees

### Heap properties

Heaps are semisorted binary trees:

-   The root of a heap holds the extreme element (maximum/minimum)
-   The branches of a heap are:
    -   Heaps themselves
    -   Empty nodes
-   Heaps are balanced
-   Filled from “left” to “right”

### Binary Search Tree properties

Binary Search Trees:

-   The root of a bst i larger than any element in its left sub-tree
-   The root of a bst i smaller than (or equal to) any element in its right sub-tree
-   The branches of a bst are:
    -   Binary search trees them selves
    -   Empty nodes
-   Binary search trees are or are not balanced

```java
public class BSTree {
    private int root;
    private BSTree left = null;
    private BSTree right = null;
    private static BSTree add(BSTree tree, int value) {
        if (tree == null)
            return new BSTree(value);
        return tree.add(value);
    }
    public BSTree(int root) {this.root = root;}
    public BSTree add(int value) {
    if (value < root)
        left = add(left, value);
    if (value >= root)
        right = add(right, value);
    return this;
    }
    void print(PrintStream out, String indent) {
    if (left != null)
        left.print(out, indent +" ␣␣");
    out.println(indent + root);
    if (right != null)
        right.print(out, indent + "␣␣");
    }
}

public static void main(String[] args) {
    SearchTree st = new SearchTree(7);
    st.add(5).add(25).add(17).add(3).add(1);
    st.print(System.out, "");
}
```
