# [Test Doubles](https://datsoftlyngby.github.io/soft2020spring/TST/week-09/#3-test-doubles)

24-02-2020

## Stubs

Problem: Nondeterministic collaborator object(s)

<table>
<td>

```java
//wrong

class AmericanWeather {
    private IOTWeatherStation station = new IOTWeatherStation();
    public double temperatureInF() {
        return 1.8 * station.getTemperature() + 32.0;
    }
}

@Test
public void testTemperatureInF() {
    AmericanWeather usw = new AmericanWeather();
    assertEquals (?, usw.temperatureInF(), 0.001);
}
```

</td>
<td>

```java
//right

interface WeatherStation{
    double getTemperature();
}

class AmericanWeather {
    private WeatherStation station;
    AmericanWetaher(WeatherStation station){
        this.station = station;
    }
    public double temperatureInF () {
        return 1.8 * station.getTemperature() + 32.0;
    }
}

class WeatherStationStub implements WeatherStation{
    @override
    public double getTemperature(){
        return 10.0;
    }
}

@Test
public void testTemperatureInF() {
    AmericanWeather usw = new AmericanWeather(new WeatherStationStub());
    assertEquals (50.0, usw.temperatureInF(), 0.001);
}
```

</td>
</table>

-   Do not overcomplicate
-   Use more stubs to get flexibility
-   Use stubs to avoid side effects

```java
class LauncherStub implements Launcher {
    @Override
    public void launchNucleareMissiles() {
        // Do nothing
    }
}
```

## Fakes

Problem: Collaborator objects with state

```java
public class Member {
    static Map <Long, Member > items = new HashMap<>();
    static long nextId = 0;
    public static Collection<Member> list() {
        return items.values();
    }
    public static Member find(long id) {
        return items.get(id);
    }
    public Member (...) {
        id = ++ nextId;
        ...
        items.put(id, this);
    }
}
```

## Mocks

-   Verifies the behaviour
-   Test your Sequence Diagrams
-   Fail on expectations not met

```java
//jMock
import org.jmock.Mockery;
import org.jmock.Expectations;
public class PublisherTest extends TestCase {
    Mockery context = new Mockery();
    public void testOneSubscriberReceivesAMessage() {
        final Subscriber subscriber = context.mock(Subscriber.class);
        Publisher publisher = new Publisher(subscriber);
        final String message = "message";
        // expectations
        context.checking(new Expectations(){{
            oneOf(subscriber).receive(message);
            }});
        // execute
        publisher.publish(message);
        // verify
        context.assertIsSatisfied();
    }
}
```

## Spies

Mock that:

-   registers behaviour
-   does not fail

## Dummies

Used to fill in dependencies
