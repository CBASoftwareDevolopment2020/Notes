# [Elementary Sorting](https://datsoftlyngby.github.io/soft2020spring/ALG/week-07/#2-elementary-sorting)

12-02-2020

## Analysis

### Utilities

#### StopWatch

```java
public class Stopwatch implements AutoCloseable {
    private final PrintStream out;
    private final long nanos = System.nanoTime();

    public Stopwatch( PrintStream out) { this.out = out; }
    public Stopwatch() { this ( System.out ); }

    public double step() {
        return ( System.nanoTime() - nanos )/1 _000_000.0;
    }

    @Override
    public void close() {
        out.printf(" %5.6 f\n", step());
    }
}
```

```java
try ( Stopwatch sw = new Stopwatch()) {
    Thread.sleep(1000);
    System.out.printf(" %5.6 f\n", sw.step());
    Thread.sleep(500);
}
```

#### FileUtility

```java
public class FileUtility {
    public static String [] toStringArray (
        String path,
        String del // delimiterPattern
    ) throws IOException {
        return Files.lines ( Paths.get( path ))
        .flatMap ( line -> Stream.of( line.split(del )))
        .filter ( word -> ! word.isEmpty())
        .map( word -> word.toLowerCase())
        .toArray ( String[]:: new );
    }
}
```

### Experiments

#### Doubling Ratio Experiment

```java
public class DoublingTest
{
    public static double timeTrial(int N)
    { // Time ThreeSum.count() for N random 6-digit ints.
        int MAX = 1000000;
        int[] a = new int[N];
        for (int i = 0; i < N; i++)
            a[i] = StdRandom.uniform(-MAX, MAX);
        Stopwatch timer = new Stopwatch();
        int cnt = ThreeSum.count(a);
        return timer.elapsedTime();
    }
    public static void main(String[] args)
    { // Print table of running times.
        for (int N = 250; true; N += N)
        { // Print time for problem size N.
            double time = timeTrial(N);
            StdOut.printf("%7d %5.1f\n", N, time);
        }
    }
}
```

## Abstract Data Types

### Bags

add: O(1)  
isEmpty: O(1)  
size: O(1)  
iterating: O(n)

### Queues

First in First out

add:O(1)  
isEmpty: O(1)  
size: O(1)  
iterating: O(n)

### Stacks

Last in First out

add: O(1)  
isEmpty: O(1)  
size: O(1)  
iterating: O(n)

### Lists

add: O(1)  
isEmpty: O(1)  
size: O(1)  
iterating: O(n)

#### Expandable arrays

Arrays are by nature of fixed length. How can we make them expandanble and still have direct memory access? When adding a new element to a full array we could:

1.  Copy the array to an array one bigger
2.  Copy the array to an array m elements bigger
3.  Copy the array to an array of double size

Average time for adding an element will be:

1.  O(n) - makes sense all elements are copied at each insert
2.  O(n) - all elements are copied each mth time O(n/m) = O(n)
3.  O(1) - how can that be?

## Sorting

### Insertion

To sort a stack of cards using insertion sort.

-   take an unsorted stack of cards
-   take one card from the unsorted stack and put in a new sorted stack
-   while there are still cards in the unsorted stack
    -   take a card from the unsorted stack
    -   while the new picked card is of higher rank than the top card in the ordered stack
        -   flip the top card of the order stack into a third temporary stack (without spoiling the order)
    -   put the card taken from the unordered stack on top of the ordered stack
    -   replace the flipped cards on top of the oredered stack.

### Selection

To sort a stack of cards using selection sort

-   take an unsorted stack of cards
-   while there are still cards in the unsorted stack
    -   Pick the top card in the unsorted stack
    -   while there are still cards in the unsorted stack
        -   compare the card picked with the top card in the unsorted stack
        -   put the card with the lowest rank in a new unordered stack, keep the other card on the hand
    -   put the card you have in your hand on the top of the ordered stack (is empty the first time)
    -   use the new unsorted stack as unsorted stack
