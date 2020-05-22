**micrometer**
>这里只做大致步骤索引，以免忘了

1. micrometer主要是用springboot集成,添加依赖

     ```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        
        <dependency>
            <groupId>io.micrometer</groupId>
            <artifactId>micrometer-registry-prometheus</artifactId>
        </dependency>
     ```
2. 开启所有的endpoint

    #开启所有的endpoint
    management:
      endpoints:
        web:
          exposure:
            include: '*'
      metrics:
        tags:
          application: ${spring.application.name}

3. Application增加RegistryCustomizer,然后启动项目能访问actuator/prometheus就可以了
     ```
        @Bean
        MeterRegistryCustomizer<MeterRegistry> configuer(@Value("${spring.application.name}")String applicationName){
            return registry -> registry.config().commonTags("application",applicationName);
        }
     ```

4. 需要用到docker，这里不详细说
    > docker docker pull prom/prometheus<br/>
     docker pull grafana/grafana<br/>
     新建一个prometheus.yml文件，文件内容如下

    ```shell script
    scrape_configs:
     # 任意写，建议英文，不要包含特殊字符
    - job_name: 'hadoopDemo'
      # 多久采集一次数据
      scrape_interval: 15s
      # 采集时的超时时间
      scrape_timeout: 10s
      # 采集的路径是啥
      metrics_path: '/actuator/prometheus'
      # 采集服务的地址，设置成上面Spring Boot应用所在服务器的具体地址。
      static_configs:
      - targets: ['192.168.100.21:8080']
    ```
5. 启动docker
```shell script
docker run -d -p 9090:9090 -v $(pwd)/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus --config.file=/etc/prometheus/prometheus.yml
docker run -d -p 3000:3000 grafana/grafana
```

    访问http://192.168.100.52:9090/，可以看到相关监控信息，
    访问http://192.168.100.52:3000使用admin/admin登录grafana
    点击Configuration添加一个DataResource,这里添加Prometheus，指定url（http://192.168.100.52:9090）
    然后点击import 在grafana官网找一个想要的模板，详情里面有个id，比如4701，然后点击load点击保存就能看到我们想要的仪表盘了
   
![示例](content/images/20200522153244.png)
