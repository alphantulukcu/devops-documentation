# ELK Stack 

## **ELK Stack** 

Elastic Stack is an open-source suite of products from Elastic designed for searching, analyzing, and visualizing data in real-time. The ELK stack includes Elasticsearch, Logstash, and Kibana.

## Elasticsearch

Elasticsearch is a distributed search and analytics engine built on Apache Lucene. It is widely used for log analytics, full-text search, and various other data-related use cases.

Companies like Uber, Udemy, Slack, Netflix, and LinkedIn use Elasticsearch.

### **Elasticsearch Setup**
Can be installed via RPM packages, Docker, or directly on various operating systems.

Docker setup includes single-node and multi-node configurations.

**Advantages:** Platform-independent, real-time search, scalable, extensive documentation, and open-source.

**Disadvantages:** Lacks multilingual support for request/response data, not suitable for handling large volumes like Hadoop, and has a steep learning curve for enterprise solutions.


> **Node:** An instance of Elasticsearch.

> **Cluster:** A group of nodes working together.

> **Document:** A unit of data stored in Elasticsearch.

> **Index:** A collection of documents.

> **Shard:** A partition of an index.

> **Replica:** A copy of a shard for fault tolerance.

### **Elasticsearch Architecture:**
  Cluster architecture where each node is aware of others and can handle HTTP requests. The master node manages indexing and cluster state.

## Logstash

Logstash is an open-source data processing pipeline that ingests, transforms, and sends data to various outputs, commonly Elasticsearch. It supports over 200 plugins.

### Logstash Setup
Installation involves adding a public signing key and setting up repositories. It can be installed on both Debian and CentOS systems.

### **Beats**
Lightweight data shippers used to send data to Logstash or directly to Elasticsearch. Filebeat is an example used for log files.

### **Log Parsing**
  - Logs are parsed using GROK patterns, which match text patterns in logs to extract structured data.

## Kibana

Kibana is a data visualization tool that integrates with Elasticsearch, allowing users to create visualizations, dashboards, and perform real-time data analysis.

Provides customizable dashboards, supports various visualizations (e.g., histograms, pie charts), and is used for monitoring data processed by Logstash.

### **Kibana Setup**

Installed similarly to Elasticsearch and Logstash by adding a repository and running a yum install command.

- **Index Management:**
  Manages indices in Elasticsearch, including the status and amount of data. Index patterns are used to define how data is indexed and queried.

- **Dashboard Management:**
  Visualizations created in Kibana can be organized into dashboards for comprehensive data analysis and monitoring.

# Case 1: Parsing Files with GROK

Using Kibana's Grok Debugger to parse log files and visualize the data in Kibana.

`docker-compose.yml`:
```yml
version: "3"

services: 
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.8.1
    environment: 
      - discovery.type=single-node
      - xpack.security.enabled=false
    ports: 
      - 9200:9200
    networks: 
      - logging-network
    hostname: elasticsearch

  logstash:
    image: docker.elastic.co/logstash/logstash:8.8.1
    depends_on: 
      - elasticsearch
    ports:
      - 5000:5000
      - 9600:9600
    volumes:
      - /Users/alphantulukcu/Desktop/OBSS/Project/case1/logstash/pipeline/logstash.conf:/usr/share/logstash/pipeline/logstash.conf:ro 
    networks:
      - logging-network

  kibana:
    image: docker.elastic.co/kibana/kibana:8.8.1
    depends_on:
      - elasticsearch
    ports: 
      - 5601:5601
    networks: 
      - logging-network

networks: 
  logging-network:
    driver: bridge

```
`logstash.conf`:
```bash
input {
  file {
    path => "/usr/share/logstash/pipeline/example2.log"
    start_position => "beginning"
    sincedb_path => "/dev/null"
    type => "example2"
  }
  file {
    path => "/usr/share/logstash/pipeline/example3.log"
    start_position => "beginning"
    sincedb_path => "/dev/null"
    type => "example3"
  }
}

filter {
  if [type] == "example2" {
    grok {
      match => {
        "message" => "%{TIMESTAMP_ISO8601:timestamp} %{LOGLEVEL:loglevel} %{GREEDYDATA:message}"
      }
    }
    date {
      match => [ "timestamp", "ISO8601" ]
    }
  }

  if [type] == "example3" {
    grok {
      match => {
        "message" => "%{MONTHNUM:month}/%{MONTHDAY:day} %{TIME:time} %{LOGLEVEL:loglevel}%{SPACE}%{DATA:location}: %{GREEDYDATA:message}"
      }
    }
    date {
      match => [ "timestamp", "MM/dd HH:mm:ss" ]
      timezone => "UTC"
    }
  }
}

output {
  stdout { codec => rubydebug }
  elasticsearch {
    hosts => ["http://elasticsearch:9200"]
    index => "examplelogs-%{+YYYY.MM.dd}"
  }
}

```


![alt text](../screenshots/elk-case-1.png)

# Case 2: ELK Stack Implementation with GitLab API 

Processing and visualizing data from the GitLab API in your ELK Stack.

`docker-compose.yml`:
```yml
version: "3"

services: 
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.8.1
    environment: 
      - discovery.type=single-node
      - xpack.security.enabled=false
    ports: 
      - 9200:9200
    networks: 
      - logging-network
    hostname: elasticsearch

  logstash:
    image: docker.elastic.co/logstash/logstash:8.8.1
    depends_on: 
      - elasticsearch
    ports:
      - 5000:5000
      - 9600:9600
    volumes:
      - /Users/alphantulukcu/Desktop/OBSS/Project/case2/logstash/pipeline/logstash.conf:/usr/share/logstash/pipeline/logstash.conf:ro 
    networks:
      - logging-network

  kibana:
    image: docker.elastic.co/kibana/kibana:8.8.1
    depends_on:
      - elasticsearch
    ports: 
      - 5601:5601
    networks: 
      - logging-network

networks: 
  logging-network:
    driver: bridge


```
`logstash.conf`:
```bash
input {
  http_poller {
    urls => {
      gitlab_projects => {
        method => get
        url => "https://gitlab-codecamp24.obss.io/api/v4/projects"
        headers => {
          Accept => "application/json"
          "PRIVATE-TOKEN" => "glpat-qkjypd-jpWmKUabsC57Q"
        }
      }
    }
    request_timeout => 60
    schedule => { every => "5m" }
    codec => "json"
    target => "http_poller"
  }
}

filter {
  split {
    field => "[http_poller][result]"
  }
  grok {
    match => { "[http_poller][result][name_with_namespace]" => "%{GREEDYDATA:project_namespace}" }
  }
  mutate {
    rename => { "[http_poller][result]" => "[project]" }
  }
}

output {
  elasticsearch {
    hosts => ["http://elasticsearch:9200"]
    index => "gitlab_projects"
    ssl_verification_mode => "full"
  }
  stdout { codec => rubydebug }
}
```

![alt text](../screenshots/elk-case-2.png)

# Case 3: ELK Stack Implementation with OpenWeatherMap API

Collecting and visualizing weather data from different cities using the OpenWeatherMap API.

`docker-compose.yml`:
```yml
version: "3"

services: 
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.8.1
    environment: 
      - discovery.type=single-node
      - xpack.security.enabled=false
    ports: 
      - 9200:9200
    networks: 
      - logging-network
    hostname: elasticsearch

  logstash:
    image: docker.elastic.co/logstash/logstash:8.8.1
    depends_on: 
      - elasticsearch
    ports:
      - 5000:5000
      - 9600:9600
    volumes:
      - /Users/alphantulukcu/Desktop/OBSS/Project/case3/logstash/pipeline/logstash.conf:/usr/share/logstash/pipeline/logstash.conf:ro 
    networks:
      - logging-network

  kibana:
    image: docker.elastic.co/kibana/kibana:8.8.1
    depends_on:
      - elasticsearch
    ports: 
      - 5601:5601
    networks: 
      - logging-network

networks: 
  logging-network:
    driver: bridge

```
`logstash.conf`:
```bash
input {
  http_poller {
    urls => {
      new_york => "http://api.openweathermap.org/data/2.5/weather?q=New%20York,US&appid=6581b1a96569014ce414d3f0ad8816bf"
      london => "http://api.openweathermap.org/data/2.5/weather?q=London,UK&appid=6581b1a96569014ce414d3f0ad8816bf"
      tokyo => "http://api.openweathermap.org/data/2.5/weather?q=Tokyo,JP&appid=6581b1a96569014ce414d3f0ad8816bf"
      paris => "http://api.openweathermap.org/data/2.5/weather?q=Paris,FR&appid=6581b1a96569014ce414d3f0ad8816bf"
      sydney => "http://api.openweathermap.org/data/2.5/weather?q=Sydney,AU&appid=6581b1a96569014ce414d3f0ad8816bf"
      berlin => "http://api.openweathermap.org/data/2.5/weather?q=Berlin,DE&appid=6581b1a96569014ce414d3f0ad8816bf"
      moscow => "http://api.openweathermap.org/data/2.5/weather?q=Moscow,RU&appid=6581b1a96569014ce414d3f0ad8816bf"
      rio_de_janeiro => "http://api.openweathermap.org/data/2.5/weather?q=Rio%20de%20Janeiro,BR&appid=6581b1a96569014ce414d3f0ad8816bf"
      cape_town => "http://api.openweathermap.org/data/2.5/weather?q=Cape%20Town,ZA&appid=6581b1a96569014ce414d3f0ad8816bf"
      mumbai => "http://api.openweathermap.org/data/2.5/weather?q=Mumbai,IN&appid=6581b1a96569014ce414d3f0ad8816bf"
    }
    request_timeout => 60
    schedule => { every => "30s" }
    codec => "json"
  }
}

filter {
  mutate {
    add_field => { "[@metadata][city]" => "%{[name]}" }
  }
  mutate {
    rename => { "main" => "weather_data" }
  }
}

output {
  elasticsearch {
    hosts => ["http://elasticsearch:9200"]
    index => "weather_data"
  }
  stdout { codec => rubydebug }
}

```
![alt text](../screenshots/elk-case-3.png)

# Case 4: APM Agent Implementation

Implementing an APM Agent in a web server to monitor and visualize performance metrics in Kibana.

```docker-compose.yml```:

```yml
version: "3"
services:
    elasticsearch:
      image: docker.elastic.co/elasticsearch/elasticsearch:7.8.0
      ports:
        - "9200:9200"
        - "9300:9300"
      environment:
        - discovery.type=single-node
    kibana:
      image: docker.elastic.co/kibana/kibana:7.8.0
      ports:
        - "5601:5601"
      links:
        - elasticsearch
      depends_on:
        - elasticsearch
    apm-server:
      image: docker.elastic.co/apm/apm-server:7.8.0
      ports:
        - "8200:8200"
      volumes:
        - ./.apm-server.yml:/usr/share/apm-server/apm-server.yml
      depends_on:
        - elasticsearch
      links:
        - elasticsearch
        - kibana
```

```.apm-server.yml```:

```yml
apm-server:
  host: apm-server:8200
  frontend.enabled: true
  frontend.allow_origins: "*"
  kibana:
    enabled: true
    host: "http://kibana:5601"

output.elasticsearch:
  enabled: true
  hosts: ["elasticsearch"]
  username: elastic
  password: elastic

setup.kibana:
  host: "kibana"
````

![alt text](../screenshots/elk-case-4.png)

