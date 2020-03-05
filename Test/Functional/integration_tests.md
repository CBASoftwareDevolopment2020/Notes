# [Integration Tests](https://datsoftlyngby.github.io/soft2020spring/TST/week-10/#4-integration-tests)

02-03-2020

## Partitions

### Equivalence Partitions(classes)

| <span style="color:red">Error</span> | F    | D     | C     | B     | A      | <span style="color:red">Error</span> |
| ------------------------------------ | ---- | ----- | ----- | ----- | ------ | ------------------------------------ |
| infinity:-1                          | 0:64 | 65:69 | 70:79 | 80:89 | 90:100 | 101:infinity                         |

Boundary value analysis

-   White-box Code
-   Back-box Specification

Edge cases

-   Numbers
-   Strings, empty strings, null, buffer overruns
-   Dates, 23/25, hours, days, timezones
-   Collections 0-1-many

State Transition Testing

-   State Transition Diagram
-   Matrix
-   0-switch coverage ... n-switch coverage
-   Decision Tables

## Dependencies

### Relations between Objects

```java
public class Raffle {
    private Set <Integer> tickets;
    public int getCount() {return tickets.size();}
    public Raffle() {
        tickets = Set.of(7 , 9 , 13);
    }
}

@Test
public void testThreeTickets() {
    Raffle raffle = new Raffle();
    assertEquals(3, raffle.getTicketCount());
}
```

#### Create a seam

##### Pass in a collaborating object

```java
public class Raffle {
    private Set <Integer> tickets;
    public int getCount() {return tickets.size();}
    public Raffle(Set<Integer> tickets) {
        this.tickets = tickets;
    }
}

@Test
public void testThreeTickets() {
    Raffle raffle = new Raffle(Set.of(7,9,13,47,11));
    assertEquals(5, raffle.getTicketCount());
}
```

Also known as dependency injection

-   When the dependant object isn't short lived
-   When the collaborator and the dependant object is at the same abstraction level
-   Could also be done using a mutator(setter), but that creates temporal coupling

###### Temporal coupling

```java
Raffle raffle = new Raffle();
raffle.setTickets(Set.of(7, 9 ,13 ,47 ,11))
```

##### Create a factory method that can be overridden

```java
public class Raffle {
    private Set<Integer> tickets;
    public int getCount() {return tickets.size();}
    public Raffle() {
        this.tickets = createTickets();
    }
    protected Set<Integer> createTickets() {
        return Set.of(7 , 9 , 13);
    }
}

public class FiveTicketRaffle extends Raffle {
    @Override
    protected Set<Integer> createTickets() {
        return Set.of(7, 9, 13, 47, 11);
    }
}

@Test public void testFiveTickets () {
    Raffle raffle = new FiveTicketRaffle();
    assertEquals (5, raffle.getTicketCount());
}
```

###### Consructor Leaks

Calling overridable methods from the constructor

##### Provide an external factory or builder

```java
public class TicketFactory {
    private final int count;
    public TicketFactory(int count) {
        this.count = count;
    }
    public Set<Integer> createTickets() {
        return IntStream.range(1, 10)
        .boxed()
        .collect(Collectors.toSet());
    }
}

public class Raffle {
    private Set <Integer> tickets;
    public int getCount() {return tickets.size();}
    public Raffle (TicketFactory factory) {
        this.tickets = factory.createTickets();
    }
}

@Test public void testFiveFactorizedTickets() {
    Raffle raffle = new Raffle(new TicketFactory (5));
    assertEquals (5, raffle.getTicketCount());
}
```

### System Resource Dependencies

-   Files
    -   Avoid input to indirect input
    -   Abstract away input/output code
    -   Separate input/output code from data processing code
-   System Clock

### Dependencies between layers

-   Dependency inversion
-   Observer pattern

### Dependencies between Tiers

-   Typically black-box
-   All the problems from layers, convoluted and intertwined
-   Remember to test the communication even when it is abstracted away
