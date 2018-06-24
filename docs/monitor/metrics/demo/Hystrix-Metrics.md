
# Hystrix 数据 publish 到 Metrics



## Maven依赖
```
<properties>
    <hystrix.version>1.5.12</hystrix.version>
    <metrics.version>3.2.6</metrics.version>
</properties>

<dependencies>

    <dependency>
        <groupId>com.netflix.hystrix</groupId>
        <artifactId>hystrix-codahale-metrics-publisher</artifactId>
        <version>${hystrix.version}</version>
    </dependency>

<dependencies>
```

## Configuration

```

import com.codahale.metrics.ConsoleReporter;
import com.codahale.metrics.MetricRegistry;
import com.netflix.hystrix.contrib.codahalemetricspublisher.HystrixCodaHaleMetricsPublisher;
import com.netflix.hystrix.strategy.HystrixPlugins;
import com.netflix.hystrix.strategy.metrics.HystrixMetricsPublisher;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

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
    public ConsoleReporter consoleReporter(MetricRegistry metricRegistry) {
        ConsoleReporter reporter = ConsoleReporter.forRegistry(metricRegistry)
                /*
                 * 将速率转换为给定的时间单位。
                 */
                .convertRatesTo(TimeUnit.SECONDS)
                /*
                 * 将持续时间转换为给定的时间单位。
                 */
                .convertDurationsTo(TimeUnit.MILLISECONDS)
                .build();

        /*
         * 每10秒在控制台打印一次
         */
        reporter.start(10, TimeUnit.SECONDS);

        return reporter;
    }

}

```

## 控制台输出

程序启动之后，每隔10s，控制台就会输出一下信息：

```
18-1-29 15:34:08 ===============================================================

-- Gauges ----------------------------------------------------------------------
....
gauge.response.health
             value = 6.0
gauge.response.info
             value = 18.0

-- Counters --------------------------------------------------------------------
counter.status.200.health
             count = 2
counter.status.200.info
             count = 1
```

> 如果对应的 HystrixCommand 有访问量，则会包含大量的 Hystrix 指标


## Read More

- [输出hystrix指标到dropwizard metrics](https://www.jianshu.com/p/cb83057a998f)
-  
- [hystrix-codahale-metrics-publisher](https://github.com/Netflix/Hystrix/tree/master/hystrix-contrib/hystrix-codahale-metrics-publisher)
-  
- Metrics [Console Reporter](http://metrics.dropwizard.io/4.0.0/getting-started.html#console-reporter)