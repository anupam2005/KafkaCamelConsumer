# KafkaCamelConsumer
A Kafka Consumer which converts AVRO data into java objects and inserts them into a Database.

# Approach

 Kafka consumer in Spring Boot using Apache Camel to consume Avro data, convert it into Java objects, and insert it into an Oracle database. Here's the code along with explanations and considerations:

1. Maven Dependencies:

XML
<dependencies>
    <dependency>
        <groupId>org.apache.camel</groupId>
        <artifactId>camel-core</artifactId>
        <version>3.11.0</version>
    </dependency>
    <dependency>
        <groupId>org.apache.camel</groupId>
        <artifactId>camel-avro</artifactId>
        <version>3.11.0</version>
    </dependency>
    <dependency>
        <groupId>org.apache.camel</groupId>
        <artifactId>camel-jdbc</artifactId>
        <version>3.11.0</version>
    </dependency>
    <dependency>
        <groupId>com.oracle.database.jdbc</groupId>
        <artifactId>ojdbc8</artifactId>
        <version>21.1.0.0</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <version>2.7.1</version>
    </dependency>
</dependencies>
Use code with caution.
2. Spring Boot Application Class:

Java
@SpringBootApplication
public class KafkaToOracleApplication {

    public static void main(String[] args) {
        SpringApplication.run(KafkaToOracleApplication.class, args);
    }
}
Use code with caution.
3. Kafka Consumer with Avro and JDBC:

Java
@Configuration
@EnableRoute
public class KafkaConsumerRoute extends RouteBuilder {

    @Value("${kafka.bootstrap-servers}")
    private String kafkaBootstrapServers;

    @Value("${kafka.topic}")
    private String kafkaTopic;

    @Value("${avro.schema.registry}")
    private String avroSchemaRegistry;

    @Value("${jdbc.url}")
    private String jdbcUrl;

    @Value("${jdbc.username}")
    private String jdbcUsername;

    @Value("${jdbc.password}")
    private String jdbcPassword;

    @Override
    public void configure() throws Exception {
        from("kafka:" + kafkaTopic + "?brokers=" + kafkaBootstrapServers + "&valueDeserializer=org.apache.avro.data.reflection.ReflectDatumReader&groupId=kafka-to-oracle-group")
                .unmarshal().avro(avroSchemaRegistry) // Deserialize Avro data
                .bean(MyAvroToJavaBeanConverter.class, "convert") // Convert to Java object
                .to("jdbc:dataSource?dataSource=#dataSourceBean") // Insert into Oracle database
        ;

        // Configure JDBC datasource bean
        bean("dataSourceBean", DataSourceCreator.createDataSource(jdbcUrl, jdbcUsername, jdbcPassword));
    }
}
Use code with caution.
4. Avro to Java Object Converter (Optional):

If your Avro schema is complex or doesn't directly map to a Java class, you might need a custom converter:

Java
public class MyAvroToJavaBeanConverter {

    public MyJavaBean convert(GenericData.Record record) {
        // Implement conversion logic using record fields and Java bean properties
        MyJavaBean bean = new MyJavaBean();
        bean.setField1(record.get("field1").toString());
        // ... add other conversions based on your schema and bean properties
        return bean;
    }
}
Use code with caution.
5. Configuration Properties:

Replace placeholders with your actual values:
kafka.bootstrap-servers: Kafka broker addresses
kafka.topic: Kafka topic name
avro.schema.registry: Avro schema registry URL
jdbc.url, jdbc.username, jdbc.password: Oracle database connection details
6. Considerations:

Ensure the Avro schema is registered and accessible by the consumer.
Adapt the MyAvroToJavaBeanConverter if needed for your specific schema and data.
Handle potential errors and exceptions in your consumer route.
Consider using a more advanced configuration management tool like Spring Cloud Config for managing sensitive credentials.
By following these steps and customizing the code to your specific needs, you should have a functional Kafka consumer that ingests Avro data, converts it to Java objects, and inserts it into your Oracle database using Spring Boot and Apache Camel.

Sources
github.com/YuliaOsadchuk/camelTest
github.com/harish3030/AWS_Lambda
github.com/1ujin/SpringBootDemo
github.com/BogdanMLD/-trainee
github.com/1294074374/o2o_springboot
github.com/Lxinglei/webim-backend subject to license (Apache - 2.0)
