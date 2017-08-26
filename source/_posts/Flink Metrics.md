---
title: "Flink Metrics"
date: 2017-08-01 21:00:00
comments: true
categories: "flink"
tags: [flink, metrics]
---

# Metrics
Flink采用metric收集、暴露系统的监控指标。通常在编写程序的时候，通常会记录日志以便事后分析，在很多情况下是发现问题以后，再通过分析日志定位问题，这是一种事后的静态分析。metric能够帮助我们了解整个系统在当前，或者某一时刻运行的情况。

比如，数据库监控：  

1. 最近一段时间内慢SQL的查询次数；
2. 查询时数据库的缓存命中率；
3. cpu、内存的负载情况；
4. 查询、删除、插入在数据库表中间的分布

实时或者准实时的收集这些指标信息，便于我们快速对系统做出响应以及优化，这是保障系统健康运行的重要手段。这些实时性能参数信息，对于一些高级应用场景，比如服务的熔断机制（统计服务调用失败和成功的比例）、告警机制，只有做到实时监控才能提供这些数据，才能实现这种提高系统稳健性的功能。

在Java中有一个开源的名为Metrics的项目，它能够捕获JVM以及应用层面的性能参数，他的作者Coda Hale介绍了什么是Mertics并且为什么Metrics在应用程序系统中很有必要，视频[YouTube](https://www.youtube.com/watch?v=czes-oa0yik)。


# Metrics注册
Fink通过调用getRuntimeContext().getMetricGroup()访问metric系统中继承RichFunction的任何函数。通过这个方法可以创建或者注册一个Metrics。

# Metics类型
Metrics提供了Gauge、Counter、Meter、Histogram、Timer等度量工具类以及Health Check功能。
## Counter
Counter是一个64位的计数器。可以通过调用inc()/inc(long n)或者dec()/dec(long n)增加或者减少统计值。通过counter(String name)在MetricGroup中创建或注册一个Counter。
```
public class MyMapper extends RichMapFunction<String, Integer> {
  private Counter counter;

  @Override
  public void open(Configuration config) {
    this.counter = getRuntimeContext()
      .getMetricGroup()
      .counter("myCounter");
  }

  @public Integer map(String value) throws Exception {
    this.counter.inc();
  }
}
```
当然也可以自己实现一个Counter:
```
public class MyMapper extends RichMapFunction<String, Integer> {
  private Counter counter;

  @Override
  public void open(Configuration config) {
    this.counter = getRuntimeContext()
      .getMetricGroup()
      .counter("myCustomCounter", new CustomCounter());
  }
}
```

## Gauge
最简单的度量指标，只有一个简单的返回值。Flink中需要实现org.apache.flink.metrics.Gauge interface接口。返回的值对类型没有限制，通过调用gauge(String name, Gauge gauge)在MetricGroup中创建&注册Gauge。
```
public class MyMapper extends RichMapFunction<String, Integer> {
  private int valueToExpose;

  @Override
  public void open(Configuration config) {
    getRuntimeContext()
      .getMetricGroup()
      .gauge("MyGauge", new Gauge<Integer>() {
        @Override
        public Integer getValue() {
          return valueToExpose;
        }
      });
  }
}
```
注：reporters会将返回的对象转换为String，所以需要实现toString()函数。

## Histograms
Histrogram是用来度量流数据中Value的分布情况，Histrogram可以计算最大/小值、平均值，方差，分位数（如中位数，或者95th分位数），如75%,90%,98%,99%的数据在哪个范围内。通过调用histogram(String name, Histogram histogram)在MetricGroup上注册一个Histogram。
```
public class MyMapper extends RichMapFunction<Long, Integer> {
  private Histogram histogram;

  @Override
  public void open(Configuration config) {
    this.histogram = getRuntimeContext()
      .getMetricGroup()
      .histogram("myHistogram", new MyHistogram());
  }

  @public Integer map(Long value) throws Exception {
    this.histogram.update(value);
  }
}
```
Flink没有提供默认的Histogram的实现类，但是提供了[Wrapper](https://github.com/apache/flink/blob/master/flink-metrics/flink-metrics-dropwizard/src/main/java/org/apache/flink/dropwizard/metrics/DropwizardHistogramWrapper.java)调用Codahale/DropWizard使用histograms。使用Wrapper需在pom.xml中添加一下依赖：
```
<dependency>
      <groupId>org.apache.flink</groupId>
      <artifactId>flink-metrics-dropwizard</artifactId>
      <version>1.2.1</version>
</dependency>
```
通过以下方式注册一个Codahale/DropWizard histogram：
```
public class MyMapper extends RichMapFunction<Long, Integer> {
  private Histogram histogram;

  @Override
  public void open(Configuration config) {
    com.codahale.metrics.Histogram histogram =
      new com.codahale.metrics.Histogram(new SlidingWindowReservoir(500));

    this.histogram = getRuntimeContext()
      .getMetricGroup()
      .histogram("myHistogram", new DropwizardHistogramWrapper(histogram));
  }
}
```
## Meter
Meter是一种只能自增的计数器，通常用来度量一系列事件发生的比率。他提供了平均速率，以及指数平滑平均速率，以及采样后的1分钟，5分钟，15分钟速率。Flink中通过markEvent(long n)注册一个需要监控的事件。调用meter(String name, Meter meter)在MetricGroup上注册。
```
public class MyMapper extends RichMapFunction<Long, Integer> {
  private Meter meter;

  @Override
  public void open(Configuration config) {
    this.meter = getRuntimeContext()
      .getMetricGroup()
      .meter("myMeter", new MyMeter());
  }

  @public Integer map(Long value) throws Exception {
    this.meter.markEvent();
  }
}
```
同样可以添加一下依赖调用Codahale/DropWizard meters的包装类：
```
<dependency>
      <groupId>org.apache.flink</groupId>
      <artifactId>flink-metrics-dropwizard</artifactId>
      <version>1.2.1</version>
</dependency>
```
采用一下方法注册一个包装类：
```
public class MyMapper extends RichMapFunction<Long, Integer> {
  private Meter meter;

  @Override
  public void open(Configuration config) {
    com.codahale.metrics.Meter meter = new com.codahale.metrics.Meter();

    this.meter = getRuntimeContext()
      .getMetricGroup()
      .meter("myMeter", new DropwizardMeterWrapper(meter));
  }
}
```

## Timer
Metrics 本身还提供Timer监控请求的速率和处理时间（Flink中暂时未提供这个监控）。

## 其他
Scope：在Flink中监控范围分系统（System Scope）、用户级别（User Scope）。系统提供了一下常用的监控如cpu、内存、线程、垃圾回收、网络、类加载、检查点、集群、IO等监控。同时用户可以通过采用上面提及的范式进行埋点，统计监控系统相关的性能。

Reporter：收集了这么多数据之后，我们需要把数据时实的动态展示或者保存起来。Flink Metric提供了多种的数据报告接口。Flink通过Reporter对外暴露监控指标。Flink提供JMX (org.apache.flink.metrics.jmx.JMXReporter)、Ganglia (org.apache.flink.metrics.ganglia.GangliaReporter)、Graphite (org.apache.flink.metrics.graphite.GraphiteReporter)、StatsD (org.apache.flink.metrics.statsd.StatsDReporter)四个Reporter。

Dashboard：Flink提供了Dashboard可视化收集的metrics。

具体的使用方法，请参考[Flink metrics](https://ci.apache.org/projects/flink/flink-docs-release-1.2/monitoring/metrics.html)。  

需要熟悉Flink源码，请参考org.apache.flink.metrics包，下面定义了一些接口和meter的可视化实例MeterViewTest。查看使用例子：StatsDReporterTest（testStatsDMetersReporting）可以采用单步调试的形式查看如何注册和发收metrics消息。

# 参考链接：
[Metrics介绍和Spring的集成](http://colobu.com/2014/08/08/Metrics-and-Spring-Integration/)  
[使用Metrics监控应用程序的性能](http://www.cnblogs.com/yangecnu/p/Using-Metrics-to-Profiling-WebService-Performance.html)  
[Metrics, Metrics, Everywhere - Coda Hale](https://www.youtube.com/watch?v=czes-oa0yik)  
[Flink metrics](https://ci.apache.org/projects/flink/flink-docs-release-1.2/monitoring/metrics.html)  
[codahale metrics](https://github.com/codahale/metrics)