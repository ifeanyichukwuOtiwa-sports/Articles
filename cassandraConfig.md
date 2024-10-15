```java
@Configuration
@EnableReactiveCassandraRepositories(
    basePackages = "com.example.repository.ds1",
    reactiveCassandraTemplateRef = "cassandraTemplate1"
)
public class CassandraConfig1 extends AbstractCassandraConfiguration {

    @Override
    protected String getKeyspaceName() {
        return "keyspace1";
    }

    @Bean
    public ReactiveCassandraTemplate cassandraTemplate1(ReactiveCassandraSessionFactoryBean sessionFactory, CassandraConverter converter) {
        return new ReactiveCassandraTemplate(sessionFactory.getObject(), converter);
    }

    @Bean
    public ReactiveCassandraSessionFactoryBean sessionFactory1() {
        ReactiveCassandraSessionFactoryBean sessionFactory = new ReactiveCassandraSessionFactoryBean();
        sessionFactory.setCluster(cluster1().getObject());
        sessionFactory.setKeyspaceName(getKeyspaceName());
        return sessionFactory;
    }

    @Bean
    public CassandraClusterFactoryBean cluster1() {
        CassandraClusterFactoryBean cluster = new CassandraClusterFactoryBean();
        cluster.setContactPoints("localhost");
        cluster.setPort(9042);
        return cluster;
    }
}
```


```java
@Configuration
@EnableReactiveCassandraRepositories(
    basePackages = "com.example.repository.ds2",
    reactiveCassandraTemplateRef = "cassandraTemplate2"
)
public class CassandraConfig2 extends AbstractCassandraConfiguration {

    @Override
    protected String getKeyspaceName() {
        return "keyspace2";
    }

    @Bean
    public ReactiveCassandraTemplate cassandraTemplate2(ReactiveCassandraSessionFactoryBean sessionFactory, CassandraConverter converter) {
        return new ReactiveCassandraTemplate(sessionFactory.getObject(), converter);
    }

    @Bean
    public ReactiveCassandraSessionFactoryBean sessionFactory2() {
        ReactiveCassandraSessionFactoryBean sessionFactory = new ReactiveCassandraSessionFactoryBean();
        sessionFactory.setCluster(cluster2().getObject());
        sessionFactory.setKeyspaceName(getKeyspaceName());
        return sessionFactory;
    }

    @Bean
    public CassandraClusterFactoryBean cluster2() {
        CassandraClusterFactoryBean cluster = new CassandraClusterFactoryBean();
        cluster.setContactPoints("localhost");
        cluster.setPort(9042);
        return cluster;
    }
}

```

```java
package com.example.repository.ds1;

import org.springframework.data.cassandra.repository.ReactiveCassandraRepository;
import org.springframework.stereotype.Repository;
import com.example.model.Entity1;

@Repository
public interface Entity1Repository extends ReactiveCassandraRepository<Entity1, String> {
}

```

```java
package com.example.repository.ds2;

import org.springframework.data.cassandra.repository.ReactiveCassandraRepository;
import org.springframework.stereotype.Repository;
import com.example.model.Entity2;

@Repository
public interface Entity2Repository extends ReactiveCassandraRepository<Entity2, String> {
}

```