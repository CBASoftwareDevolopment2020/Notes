# [Unit Tests and testability](https://datsoftlyngby.github.io/soft2020spring/TST/week-08/#2-unit-tests-and-testability)

17-02-2020

## Unit test framework

-   Assert - metoder
-   Hvilke metoder er test
-   Defineres i en klasse
-   Prefix - typisk test
-   Annotation
-   I kode direkte

```java
public class AssertionException extends RuntimeException {
    private Object actual;
    private Object expected;
    public AssertionException(String message, Object actual, Object expected) {
        super(message);
        this.actual = actual;
        this.expected = expected;
    }
    public Object getActual() { return actual; }
    public Object getExpected() { return expected; }
}
```

```java
public class Actions {
    public static void assertEquals(String message, Object actual, Object expected){
        if (actual != null && actual.equals(expected) || actual == expected) return;
        throw new AssertionException(message, actual, expected);
        // if((actual==null||!actual.equals(expected))&&actual!=expected) throw new AssertionException(message, actual, expected);
    }
}
```

```java
//importing static methods
import static Actions.*
public class FooTest {
    public void testFoo(){
        String foo = "Foo";
        String result = foo.toUpperCase();
        assertEquals("Result should be upper case", result, "FOO");
        assertEquals("Length should not differ", result.length, foo.length);
    }
    public void testBar(){
        System.out.println("Bar");
    }
}
```

```java
public class Runner {
    public void run(Class klass) {
        Method[] methods = klass.getMethods();
        int failed = 0;
        int succes = 0;
        for (Method method : methods) {
            if (!method.getName().startsWith("test")) continue;
            try {
                Object instance = klass.getDeclaredConstructor().newInstance();
                method.invoke(instance);
                succes++;
            } catch(InvocationTargetException ite) {
                if (ite.getCause() instanceof AssertionException){
                    AssertionException ae = (AssertionException) ite.getCause();
                    System.out.println(method.getName() + ": " + ae.getMessage());
                    System.out.println(method.getName() + ": Failed test: expected " + ae.getExpected() + ", but got " + ae.getActual());
                }
                failed++;
            } catch (Exception e) {
                failed++;
                e.printStackTrace();
            }
        }
        System.out.print("Succes: " + succes + ", Fail: ");
        if (failed > 0) System.out.println(failed + "\nSome tests failed");
        else System.out.println("0\nAll tests completed successfully");
    }
}
```

```java
import java.lang.reflect.Method
public class Program {
    public static void main(String[] args) throws Exception{
        Method[] methods = FooTest.class.getMethods();
        FooTest test = new FooTest();
        for (Method method : methods) {
            if (!method.getName().startsWith("test")) continue;
            System.out.println("Invoking test method: " + method.getName());
            method.invoke(test);
        }
    }
}
```

## Mocking

```java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.util.Arrays;
import java.util.HashMap;
import java.util.Map;
import java.util.stream.Collectors;
public class Mocker implements InvocationHandler {
    private Map<String,Object> expectations = new HashMap<>();
    private static Mocker mockerInvoked = null;
    private static Method methodInvoked = null;
    public static Object mock(Class klass){
        ClassLoader loader = ClassLoader.getSystemClassLoader();
        return Proxy.newProxyInstance(loader,new Class[]{ klass },new Mocker());
    }
    private static String keyOf(Method method) {
        return method.toString() + ":" +  Arrays.stream(method.getParameterTypes())
                                                .map(paramType -> paramType.toString())
                                                .collect(Collectors.joining("#"));
    }
    public static void when(Object value, Object result) {
        System.out.println("when called with " + result);
        if (mockerInvoked == null || methodInvoked == null) throw new RuntimeException("Hovsa");
        String key = keyOf(methodInvoked);
        mockerInvoked.expectations.put(key, result);
        methodInvoked = null;
        mockerInvoked = null;
    }
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        String key = keyOf(method);
        if (expectations.containsKey(key)) {
            return expectations.get(key);
        }
        mockerInvoked = this;
        methodInvoked = method;
        System.out.println("You called " + method.getName() + " without expectations");
        return null;
    }
    public static void main(String[] args) {
        Bank bank = (Bank)Mocker.mock(Bank.class);
        Mocker.when(bank.getName(),"KurtsBank");
        Mocker.when(bank.getManager(),"Sonja");
        System.out.println(bank.getName() + " managed by " + bank.getManager());
    }
}
```

```java
public interface Bank {
    String getName();
    String getManager();
}
```
