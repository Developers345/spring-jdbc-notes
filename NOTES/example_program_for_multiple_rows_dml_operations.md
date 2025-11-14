# Java Config Spring JDBC Example (Multiple Rows and DML Operations)

## BikeDAO.java

```java
// we want to perform database operation, so we need to use JdbcTemplate inside the class
@Repository
public class BikeDAO {

    private final String GET_ALL_BIKES = "select bike_no,bike_name,bike_color,bike_model from bike";
    private final String INSERT_BIKE_DATA = "insert into bike values(?,?,?,?)";

    @Autowired
    private JdbcTemplate jdbcTemplate;

    public List<BikeBo> getAllBikes() {
        return jdbcTemplate.query(GET_ALL_BIKES, rowMapper);
    }

    public int addBike(BikeBo bikeBo) {
        return jdbcTemplate.update(
                INSERT_BIKE_DATA,
                bikeBo.getBikeNo(),
                bikeBo.getBikeName(),
                bikeBo.getBikeColor(),
                bikeBo.getBikeModel()
        );
    }

    /*
    Lambda expressions are suitable when the method is used only once.
    If used multiple times, lambda expressions are not recommended.

    (rs, rowNum) -> {
        BikeBo bikeBo = new BikeBo();
        bikeBo.setBikeNo(rs.getInt(1));
        bikeBo.setBikeName(rs.getString(2));
        bikeBo.setBikeColor(rs.getString(3));
        bikeBo.setBikeModel(rs.getString(4));
        return bikeBo;
    }
    */

    /*
    We can go for anonymous inner class or private inner class

    NOTE:
    - we should not iterate ResultSet object
    - we should not close the ResultSet manually
    */

    RowMapper<BikeBo> rowMapper = new RowMapper<BikeBo>() {
        @Override
        public BikeBo mapRow(ResultSet rs, int rowNum) throws SQLException {
            BikeBo bikeBo = new BikeBo();
            bikeBo.setBikeNo(rs.getInt(1));
            bikeBo.setBikeName(rs.getString(2));
            bikeBo.setBikeColor(rs.getString(3));
            bikeBo.setBikeModel(rs.getString(4));
            return bikeBo;
        }
    };
}
````

---

## JavaConfig.java

```java
@Configuration
@PropertySource("classpath:db.properties")
@ComponentScan(basePackages = {"com.spring.jdbc.javaconfig"})
class JavaConfig {

    @Autowired
    private Environment env;

    @Bean
    public DataSource dataSource() {
        DriverManagerDataSource dataSource = new DriverManagerDataSource();
        dataSource.setDriverClassName(env.getProperty("driverClassName"));
        dataSource.setUrl(env.getProperty("url"));
        dataSource.setUsername(env.getProperty("username"));
        dataSource.setPassword(env.getProperty("password"));
        return dataSource;
    }

    @Bean
    public JdbcTemplate jdbcTemplate(DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }
}
```

---

## db.properties

```properties
driverClassName=com.mysql.cj.jdbc.Driver
url=jdbc:mysql://localhost:3306/rapido
username=root
password=root
```

---

## BikeBo.java

```java
public class BikeBo {

    protected int bikeNo;
    protected String bikeName;
    protected String bikeColor;
    protected String bikeModel;

    public int getBikeNo() { return bikeNo; }
    public void setBikeNo(int bikeNo) { this.bikeNo = bikeNo; }

    public String getBikeName() { return bikeName; }
    public void setBikeName(String bikeName) { this.bikeName = bikeName; }

    public String getBikeColor() { return bikeColor; }
    public void setBikeColor(String bikeColor) { this.bikeColor = bikeColor; }

    public String getBikeModel() { return bikeModel; }
    public void setBikeModel(String bikeModel) { this.bikeModel = bikeModel; }

    @Override
    public String toString() {
        return "BikeBo{" +
                "bikeNo=" + bikeNo +
                ", bikeName='" + bikeName + '\'' +
                ", bikeColor='" + bikeColor + '\'' +
                ", bikeModel='" + bikeModel + '\'' +
                '}';
    }
}
```

---

## Test.java

```java
public class JDBCConfigTest {

    public static void main(String[] args) {

        ApplicationContext context = new AnnotationConfigApplicationContext(JavaConfig.class);

        BikeDAO bikeDAO = context.getBean("bikeDao", BikeDAO.class);

        BikeBo bikeBoObject = new BikeBo();
        bikeBoObject.setBikeNo(4);
        bikeBoObject.setBikeName("Pluser");
        bikeBoObject.setBikeColor("Green");
        bikeBoObject.setBikeModel("B4");

        int status = bikeDAO.addBike(bikeBoObject);
        System.out.println("row updated: " + status);

        List<BikeBo> bikeBos = bikeDAO.getAllBikes();
        Stream sBikes = Stream.of(bikeBos);
        // bikeBos.stream().forEach(System.out::println);
        sBikes.forEach(System.out::println);
    }
}
```

---

## Output

```
row updated: 1
[BikeBo{bikeNo=1, bikeName='Honda', bikeColor='Red', bikeModel='B1'},
 BikeBo{bikeNo=2, bikeName='Hero', bikeColor='Blue', bikeModel='B2'},
 BikeBo{bikeNo=3, bikeName='Hero Honda', bikeColor='White', bikeModel='B3'},
 BikeBo{bikeNo=4, bikeName='Pluser', bikeColor='Green', bikeModel='B4'}]
```
