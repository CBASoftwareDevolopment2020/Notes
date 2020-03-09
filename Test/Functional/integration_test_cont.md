# [Integration Tests cont.](https://datsoftlyngby.github.io/soft2020spring/TST/week-11/#5-system-tests-and-acceptance-tests)

09-03-2020

## Strategies

### Big Bang

**[Big Bang](http://softwaretestingfundamentals.com/integration-testing/)** is an approach to Integration Testing where all or most of the units are combined together and tested at one go. This approach is taken when the testing team receives the entire software in a bundle. So what is the difference between Big Bang Integration Testing and System Testing? Well, the former tests only the interactions between the units while the latter tests the entire system.

### Top Down

**Top Down** is an approach to Integration Testing where top-level units are tested first and lower level units are tested step by step after that. This approach is taken when top-down development approach is followed. Test Stubs are needed to simulate lower level units which may not be available during the initial phases.

### Bottom Up

**Bottom Up** is an approach to Integration Testing where bottom level units are tested first and upper-level units step by step after that. This approach is taken when bottom-up development approach is followed. Test Drivers are needed to simulate higher level units which may not be available during the initial phases.

### Sandwich

**Sandwich/Hybrid** is an approach to Integration Testing which is a combination of Top Down and Bottom Up approaches. This approach is very useful for regressions test.

## Testing Interfaces

```java
package ...contract ;
public class ChoirManagerHolder {
    public static ChoirManager manager;
}

import static ...contract.ChoirManagerHolder.*;
public class ChoirManagerTest {
    @Test
    public void testListMembers() {
        assumeThat(manager, not(nullValue()));
        Collection<MemberSummary> members = manager.listMembers();
        assertThat(members, not(nullValue()));
    }
}
```

Contract POM

```xml
<plugin>
    <artifactId>maven-jar-plugin</artifactId>
    <version>2.3.2</version>
    <executions>
        <execution>
            <id>default-jar</id>
            <phase>package</phase>
            <goals>
                <goal>jar</goal>
            </goals>
        </execution>
        <execution>
            <id>test-jar</id>
            <phase>package</phase>
            <goals>
                <goal>test-jar</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

Backend POM

```xml
<dependency>
    <groupId>dk.cphbusiness.choir</groupId>
    <artifactId>choir-contract</artifactId>
    <version>1.0.1</version>
    <type>test-jar</type>
    <scope>test</scope>
</dependency>
```
