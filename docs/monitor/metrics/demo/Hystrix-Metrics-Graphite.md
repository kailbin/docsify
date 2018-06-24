
# Hystrix 数据push到 Graphite


[TOC]


## Maven 依赖

```
<properties>
    <hystrix.version>1.5.12</hystrix.version>
    <metrics.version>3.1.4</metrics.version>
</properties>

<dependencies>

    <dependency>
        <groupId>com.netflix.hystrix</groupId>
        <artifactId>hystrix-codahale-metrics-publisher</artifactId>
        <version>${hystrix.version}</version>
        <exclusions>
            <exclusion>
                <groupId>com.codahale.metrics</groupId>
                <artifactId>metrics-core</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    
    <dependency>
        <groupId>io.dropwizard.metrics</groupId>
        <artifactId>metrics-core</artifactId>
        <version>${metrics.version}</version>
    </dependency>
    
    <dependency>
        <groupId>io.dropwizard.metrics</groupId>
        <artifactId>metrics-graphite</artifactId>
        <version>${metrics.version}</version>
    </dependency>

<dependencies>
```

## Configuration

```
import com.codahale.metrics.MetricRegistry;
import com.codahale.metrics.graphite.Graphite;
import com.codahale.metrics.graphite.GraphiteReporter;
import com.codahale.metrics.graphite.GraphiteSender;
import com.netflix.hystrix.contrib.codahalemetricspublisher.HystrixCodaHaleMetricsPublisher;
import com.netflix.hystrix.strategy.HystrixPlugins;
import com.netflix.hystrix.strategy.metrics.HystrixMetricsPublisher;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.net.InetSocketAddress;
import java.util.concurrent.TimeUnit;

@Configuration
public class HystrixMetricsPublisherConfig {

    @Bean
    HystrixMetricsPublisher hystrixMetricsPublisher(MetricRegistry metricRegistry) {
        HystrixCodaHaleMetricsPublisher publisher = new HystrixCodaHaleMetricsPublisher(metricRegistry);
        HystrixPlugins.getInstance().registerMetricsPublisher(publisher);
        return publisher;
    }

    @Bean
    public GraphiteReporter graphiteReporter(MetricRegistry metricRegistry) {
        final GraphiteReporter reporter = GraphiteReporter.forRegistry(metricRegistry).build(graphite());
        reporter.start(1, TimeUnit.SECONDS);
        return reporter;
    }

    @Bean
    GraphiteSender graphite() {
    	/*
         * graphite 地址
         */
        return new Graphite(new InetSocketAddress("172.16.2.220", 32003));
    }

}
```

## Read More

- [Storing Months of Historical Metrics from Hystrix in Graphite](https://dzone.com/articles/storing-months-historical)
- 
- Docker 镜像 [kamon/grafana_graphite](https://hub.docker.com/r/kamon/grafana_graphite/)
	- 	**80: the Grafana web interface.**
	- 	**81: the Graphite web port**
	- 	**2003: the Graphite data port**
	- 	8125: the StatsD port.
	- 	8126: the StatsD administrative port.
	- 	`docker run -d -p 380:80 -p 381:81 -p 32003:2003 -p 38125:8125 -p 38126:8126   kamon/grafana-graphite`