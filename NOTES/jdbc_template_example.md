# Example Program

## BikeDAO.java

```java
public class BikeDAO {

    // if my query returns one result as a primitive type then i have to use queryForObject
    private final static String GET_BIKE_COUNT="select count(1) from bike";
    private JdbcTemplate jdbcTemplate;

    public void setJdbcTemplate(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    public int getNoOfBikes() {
        return jdbcTemplate.queryForObject(GET_BIKE_COUNT, Integer.class);
    }
}
````

---

## application-context.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/rapido"/>
        <property name="username" value="root"/>
        <property name="password" value="root"/>
    </bean>

    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <bean id="bikeDao" class="com.spring.jdbc.configuration.BikeDAO">
        <property name="jdbcTemplate" ref="jdbcTemplate"/>
    </bean>

</beans>
```

---

## JDBCConfigTest.java

```java
public class JDBCConfigTest {

    public static void main(String[] args) {

        ApplicationContext context = new ClassPathXmlApplicationContext("application-context.xml");

        BikeDAO bikeDAO = context.getBean("bikeDao", BikeDAO.class);
        int bikeCount = bikeDAO.getNoOfBikes();
        System.out.println("bikeCount: " + bikeCount);
    }
}
```

---

## Output

```
bikeCount: 7
```
