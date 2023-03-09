# JDBC

* JDBC(Java Database Connectivity)
	* Java에서 RDBMS와 자바의 연결고리 역할을 수행해주는 API.
	* 각각의 DBMS 벤더에서 제공해주는 JDBC Driver를 가지고 연동을 수행해야 함.
	* 관련 클래스
		* DriverManager: JDBC Driver 집합을 관리하는 클래스.
		* Connection: DB와 Java 애플리케이션과의 연결고리를 관리해주는 인터페이스. DriverManager의 getConnection 메서드를 이용하여 구현체에 해당하는 인스턴스를 가져올 수 있음.
		* DataSource: Connection Pool에 Connection 구현체 인스턴스를 저장 및 관리할 때 사용하는 인터페이스. Connection 구현체 인스턴스를 매번 가져오지 않고, 필요할 때 가져와서 사용 가능.
		* PreparedStatement: Connection 구현체 인스턴스를 가지고 SQL문을 수행할 때 사용하는 인터페이스. execute 메서드를 이용하여 SQL문 수행 가능.

<figure><img src="./images/jdbc.png" alt=""></figure>

<br>

* DBCP(DataBase Connection Pool)
	* JDBC를 이용한 DB 연동 시, 커넥션(Connection)을 여러 개 미리 만들어놓고 필요할 때마다 가져다 쓰는 것.
	* 처음에 몇 개를 만들어놓을지, 얼마나 응답이 없으면 에러를 던질지 등의 설정을 할 수 있음.
	* 애플리케이션 성능에 핵심적인 역할을 수행함.
	* Spring Boot 2.x+에서는 HikariCP를 기본 DBCP로 사용함.

<br>

* Postgres를 사용한 JDBC 연동 예제
	* 해당하는 JDBC Driver 의존성을 반드시 추가해야 함.
```
build.gradle

dependencies {
    implementation 'org.postgresql:postgresql:42.5.4'
}


@SpringBootApplication
public class App {

    public static void main(String[] args) throws SQLException {
				String url = "jdbc:postgresql://localhost:5432/postgres";

				Properties properties = new Properties();
				properties.put("user", "postgres");
				properties.put("password", "postgres");

				Connection connection = DriverManager.getConnection(url, properties);

				System.out.println(connection);

				Statement statement = connection.createStatement();
				String query = "CREATE TABLE people (id int, name varchar(255), age int)";
				// String query = "SELECT * FROM people";
				ResultSet resultSet = statement.executeQuery(query);

				while (resultSet.next()) {
						String name = resultSet.getString("name");
						System.out.println(name);
				}
		}
}
```

<br>

* JdbcTemplate
	* Spring에서 JDBC를 사용하기 위한 Tool로써, JDBC 코어 패키지의 중심 클래스라 할 수 있음.
	* JDBC 사용을 쉽게 할 수 있도록 도와주며, 그에 따른 오류를 방지하는데 도움을 줄 수 있음.
	* 해당하는 JDBC Driver와 spring-boot-starter-jdbc 의존성을 추가해줘야 함.
	* application.yml에 연동이 이루어질 DB 정보를 구성해줘야 함.
	* SELECT문 사용 시 query 메서드를 사용하며, INSERT/UPDATE/DELETE문 사용 시 update 메서드를 사용함.
```
build.gradle

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-jdbc'
    developmentOnly 'org.springframework.boot:spring-boot-devtools'
    runtimeOnly 'org.postgresql:postgresql'
}


application.yml

spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/postgres
    username: postgres
    password: postgres


@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
public class Post {

    private String id;

    private String title;

}


@Component
@RequiredArgsConstructor
public class AppRunner implements CommandLineRunner {

    private final JdbcTemplate jdbcTemplate;

    private final TransactionTemplate transactionTemplate;

    @Override
    public void run(String... args) throws Exception {
        String query = "SELECT * FROM people WHERE name LIKE ?";

        System.out.println("-".repeat(80));

        jdbcTemplate.query(query, resultSet -> {
            while (resultSet.next()) {
                String name = resultSet.getString("name");
                System.out.println(name);
            }
            return null;
        }, "%우%");

        System.out.println("-".repeat(80));
    }

//    @Override
//    public void run(String... args) throws Exception {
//        transactionTemplate.execute(status -> {
//            String sql = """
//                    INSERT INTO people(name, age, gender) VALUES(?, ?, ?)
//                    """;
//            jdbcTemplate.update(sql, "홍길동", 15, "남");
//            return null;
//        });
//    }
}


@Component
@RequiredArgsConstructor
public class PostDao {

    private final JdbcTemplate jdbcTemplate;

    public List<Post> findAll() {
        List<Post> posts = new ArrayList<>();

        String query = "SELECT * FROM posts";
        jdbcTemplate.query(query, resultSet -> {
            while (resultSet.next()) {
                String id = resultSet.getString("id");
                String title = resultSet.getString("title");
                
                Post post = new Post(id, title);
                posts.add(post);
            }
        });

        return posts;
    }

}
```

<br>

* 참고
  * 인프런 <스프링 데이터 JPA> - 백기선
