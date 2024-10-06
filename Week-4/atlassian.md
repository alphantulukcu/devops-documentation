# Case 1 - JIRA Data Center

> We expect JIRA Software, one of Atlassian products, to work in Docker as a Data Center. The structure is expected to be created as follows:

    Services must be created with Docker Compose.
    Docker Swarm should not be used.
    Must be done using Atlassian's Docker Images. 3rd Party Image should not be used.
    NFS Structure should be set up and each node should be conjugate with each other.
    Requests coming to JIRA nodes using HAProxy or NGINX should be subjected to LoadBalancing.
    You are free to choose the database you want. (Don't forget to check compatibility.)

## Implementation

I created three files, which are as follow:

I created a Docker Compose file, which defines the services required to set up Jira Software as a Data Center in Docker.

I added two Jira nodes `jira_node_1` and `jira_node_2` using Atlassian's official Docker image for Jira Software. Each node has a unique JIRA_NODE_ID and shares the same environment variables for database configuration. Both nodes are configured to run in clustered mode by setting `CLUSTERED=true`, which enables Jira Data Center features. I also configured the nodes to use a shared NFS volume for Jira's shared application data, ensuring both nodes can access the same data.
I included a PostgreSQL database service (db) using the official PostgreSQL image. I configured the database service with necessary environment variables, such as `POSTGRES_USER`, `POSTGRES_PASSWORD`, and `POSTGRES_DB`. I also mapped a local directory to the container's data directory to persist the database data.
I added an NGINX service to handle load balancing between the two Jira nodes. This service depends on the Jira nodes and uses the official NGINX image. I mapped the custom nginx.conf file to the container's NGINX configuration directory. The NGINX service listens on ports 80 and 8080, directing incoming traffic to the Jira nodes using the load-balancing method defined in the nginx.conf file.
I defined a custom bridge network `jiranet` to ensure that all services can communicate with each other securely and efficiently. This network is shared among all services, ensuring isolated but interconnected containers.

```docker-compose.yml```:

```bash
version: '3'

services:
  jira_node_1:
    depends_on:
      - db
    image: atlassian/jira-software:latest
    container_name: jira_node_1
    volumes:
      - /Users/Shared/NFS:/var/atlassian/application-data/jira/shared
    ports:
      - '8081:8080'
    environment:
      - CLUSTERED=true
      - JIRA_NODE_ID=jira_node_1
      - ATL_JDBC_URL=jdbc:postgresql://db/jiradb
      - ATL_JDBC_USER=jira
      - ATL_JDBC_PASSWORD=jirajira
      - ATL_DB_DRIVER=org.postgresql.Driver
      - ATL_DB_TYPE=postgres72
      - ATL_DB_SCHEMA_NAME=public
      - ATL_CLUSTER_NAME=JIRACluster
    networks:
      - jiranet

  jira_node_2:
    depends_on:
      - jira_node_1
    image: atlassian/jira-software:latest
    container_name: jira_node_2
    volumes:
      - /Users/Shared/NFS:/var/atlassian/application-data/jira/shared
    ports:
      - '8082:8080'
    environment:
      - CLUSTERED=true
      - JIRA_NODE_ID=jira_node_2
      - ATL_JDBC_URL=jdbc:postgresql://db/jiradb
      - ATL_JDBC_USER=jira
      - ATL_JDBC_PASSWORD=jirajira
      - ATL_DB_DRIVER=org.postgresql.Driver
      - ATL_DB_TYPE=postgres72
      - ATL_CLUSTER_NAME=JIRACluster
    networks:
      - jiranet

  db:
    image: postgres:15
    networks:
      - jiranet
    volumes:
      - ./posgresqldata:/var/lib/postgresql/data
      - ./init-db.sh:/docker-entrypoint-initdb.d/init-db.sh
    ports:
      - '5432:5432'
    environment:
      - 'POSTGRES_USER=jira'
      - 'POSTGRES_PASSWORD=jirajira'
      - 'POSTGRES_DB=jiradb'

  nginx:
      image: nginx:latest
      volumes:
        - ./nginx.conf:/etc/nginx/nginx.conf
      ports:
        - "80:80"
        - "8080:8080"
      depends_on:
        - jira_node_1
        - jira_node_2
      networks:
        - jiranet

volumes:
  posgresqldata:

networks:
  jiranet:
    driver: bridge

```

```nginx.config```:

```bash
events {}

stream {
    upstream jiranet {
        least_conn;
        server jira_node_1:8080;
        server jira_node_2:8080;
    }

    server {
        listen 8080;
        proxy_pass jiranet;
    }
}


```

```init-db.sh```:

```bash
#!/bin/bash
psql -v ON_ERROR_STOP=1 --username "$POSTGRES_USER" --dbname "$POSTGRES_DB" <<-EOSQL
    GRANT ALL PRIVILEGES ON DATABASE jiradb TO jira;
    GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO jira;
    GRANT ALL PRIVILEGES ON ALL SEQUENCES IN SCHEMA public TO jira;
    GRANT ALL PRIVILEGES ON ALL FUNCTIONS IN SCHEMA public TO jira;
EOSQL

```


# Case 2 - Confluence Data Center

> We expect confluence Software, one of Atlassian products, to work in Docker as a Data Center. The structure is expected to be created as follows:

    Services must be created with Docker Compose.
    Docker Swarm should not be used.
    Must be done using Atlassian's Docker Images. 3rd Party Image should not be used.
    NFS Structure should be set up and each node should be conjugate with each other.
    Requests coming to confluence nodes using HAProxy or NGINX should be subjected to LoadBalancing.
    You are free to choose the database you want. (Don't forget to check compatibility.)

## Implementation

I created three files, which are as follow:

I created a Docker Compose file, which defines the services required to set up onfluence software as a Data Center in Docker.

I added two confluence nodes `confluence_node_1` and `confluence_node_2` using Atlassian's official Docker image for confluence Software. Each node has a unique CONFLUENCE_NODE_ID and shares the same environment variables for database configuration. Both nodes are configured to run in clustered mode by setting `CLUSTERED=true`, which enables confluence Data Center features. I also configured the nodes to use a shared NFS volume for confluence's shared application data, ensuring both nodes can access the same data.
I included a PostgreSQL database service (db) using the official PostgreSQL image. I configured the database service with necessary environment variables, such as `POSTGRES_USER`, `POSTGRES_PASSWORD`, and `POSTGRES_DB`. I also mapped a local directory to the container's data directory to persist the database data.
I added an NGINX service to handle load balancing between the two confluence nodes. This service depends on the confluence nodes and uses the official NGINX image. I mapped the custom nginx.conf file to the container's NGINX configuration directory. The NGINX service listens on ports 80 and 8080, directing incoming traffic to the confluence nodes using the load-balancing method defined in the nginx.conf file.
I defined a custom bridge network `confluencenet` to ensure that all services can communicate with each other securely and efficiently. This network is shared among all services, ensuring isolated but interconnected containers.

```docker-compose.yml```:

```bash
version: '3'

services:
  confluence_node_1:
    depends_on:
      - db
    image: atlassian/confluence-server:latest
    container_name: confluence_node_1
    volumes:
      - /Users/Shared/NFS2:/var/atlassian/application-data/confluence/shared
    ports:
      - '8091:8090'
    environment:
      - CLUSTERED=true
      - CONFLUENCE_NODE_ID=confluence_node_1
      - ATL_JDBC_URL=jdbc:postgresql://db/confluencedb
      - ATL_JDBC_USER=confluence
      - ATL_JDBC_PASSWORD=confluenceconfluence
      - ATL_DB_DRIVER=org.postgresql.Driver
      - ATL_DB_TYPE=postgresql
      - ATL_DB_SCHEMA_NAME=public
      - ATL_CLUSTER_TYPE=multicast
      - ATL_CLUSTER_NAME=confluence-cluster
      - ATL_CLUSTER_ADDRESS=230.0.0.1
      - ATL_CLUSTER_TTL=32
      - CONFLUENCE_HOME=//var/atlassian/application-data/confluence
      - ATL_PRODUCT_HOME_SHARED=/var/atlassian/application-data/confluence/shared
      - JVM_SUPPORT_RECOMMENDED_ARGS=-Dconfluence.cluster.node.name="node1"

    networks:
      - confluencenet

  confluence_node_2:
    depends_on:
      - confluence_node_1
    image: atlassian/confluence-server:latest
    container_name: confluence_node_2
    volumes:
      - /Users/Shared/NFS2:/var/atlassian/application-data/confluence/shared
    ports:
      - '8092:8090'
    environment:
      - CLUSTERED=true
      - CONFLUENCE_NODE_ID=confluence_node_2
      - ATL_JDBC_URL=jdbc:postgresql://db/confluencedb
      - ATL_JDBC_USER=confluence
      - ATL_JDBC_PASSWORD=confluenceconfluence
      - ATL_DB_DRIVER=org.postgresql.Driver
      - ATL_DB_TYPE=postgresql
      - ATL_CLUSTER_TYPE=multicast
      - ATL_CLUSTER_NAME=confluence-cluster
      - ATL_CLUSTER_ADDRESS=230.0.0.1
      - ATL_CLUSTER_TTL=32
      - CONFLUENCE_HOME=//var/atlassian/application-data/confluence
      - ATL_PRODUCT_HOME_SHARED=/var/atlassian/application-data/confluence/shared
      - JVM_SUPPORT_RECOMMENDED_ARGS=-Dconfluence.cluster.node.name="node2"

    networks:
      - confluencenet

  db:
    image: postgres:15
    networks:
      - confluencenet
    volumes:
      - ./posgresqldata:/var/lib/postgresql/data
      - ./init-db.sh:/docker-entrypoint-initdb.d/init-db.sh
    ports:
      - '5432:5432'
    environment:
      - 'POSTGRES_USER=confluence'
      - 'POSTGRES_PASSWORD=confluenceconfluence'
      - 'POSTGRES_DB=confluencedb'

  nginx:
    image: nginx:latest
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    ports:
      - "80:80"
      - "8090:8090"
    depends_on:
      - confluence_node_1
      - confluence_node_2
    networks:
      - confluencenet

volumes:
  posgresqldata:

networks:
  confluencenet:
    driver: bridge

```

```nginx.config```:

```bash
events {}

stream {
    upstream confluencenet {
        least_conn;
        server confluence_node_1:8090;
        server confluence_node_2:8090;
    }

    server {
        listen 8090;
        proxy_pass confluencenet;
    }
}

```

```init-db.sh```:

```bash
#!/bin/bash
psql -v ON_ERROR_STOP=1 --username "$POSTGRES_USER" --dbname "$POSTGRES_DB" <<-EOSQL
    GRANT ALL PRIVILEGES ON DATABASE confluencedb TO confluence;
    GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO confluence;
    GRANT ALL PRIVILEGES ON ALL SEQUENCES IN SCHEMA public TO confluence;
    GRANT ALL PRIVILEGES ON ALL FUNCTIONS IN SCHEMA public TO confluence;
EOSQL


```

# Case 3 - Bitbucket Data Center

> We expect Bitbucket Software, one of Atlassian products, to work in Docker as a Data Center. The structure is expected to be created as follows:
    Services must be created with Docker Compose.
    Docker Swarm should not be used.
    Must be done using Atlassian's Docker Images. 3rd Party Image should not be used.
    NFS Structure should be set up and each node should be conjugate with each other.
    Elasticsearch should be set up and each node should be conjugate with Bitbucket nodes
    Requests coming to JIRA nodes using HAProxy or NGINX should be subjected to LoadBalancing.
    You are free to choose the database you want. (Don't forget to check compatibility.)

## Implementation

I created three files, which are as follow:

I created a Docker Compose file, which defines the services required to set up onfluence software as a Data Center in Docker.

I added two bitbucket nodes `bitbucket_node_1` and `bitbucket_node_2` using Atlassian's official Docker image for bitbucket Software. Each node has a unique bitbucket_NODE_ID and shares the same environment variables for database configuration. Both nodes are configured to run in clustered mode, which enables bitbucket Data Center features. I also configured the nodes to use a shared NFS volume for bitbucket's shared application data, ensuring both nodes can access the same data.
I included a PostgreSQL database service (db) using the official PostgreSQL image. I configured the database service with necessary environment variables, such as `POSTGRES_USER`, `POSTGRES_PASSWORD`, and `POSTGRES_DB`. I also mapped a local directory to the container's data directory to persist the database data.
I added an NGINX service to handle load balancing between the two bitbucket nodes. This service depends on the bitbucket nodes and uses the official NGINX image. I mapped the custom nginx.conf file to the container's NGINX configuration directory. The NGINX service listens on ports 80 and 7990, directing incoming traffic to the bitbucket nodes using the load-balancing method defined in the nginx.conf file.
I defined a custom bridge network `bitbucketnet` to ensure that all services can communicate with each other securely and efficiently. This network is shared among all services, ensuring isolated but interconnected containers.

```docker-compose.yml```:

```bash
version: '3'

services:
  bitbucket_node_1:
    platform: linux/amd64  # Ensure correct architecture
    depends_on:
      - db
      - elasticsearch
    image: atlassian/bitbucket-server:latest
    container_name: bitbucket_node_1
    volumes:
      - /Users/Shared/NFS4:/var/atlassian/application-data/bitbucket/shared
    ports:
      - '7991:7990'
    working_dir: /opt/atlassian/bitbucket  # Set the correct working directory
    command: ["./bin/start-bitbucket.sh", "-fg"]
    environment:
      - ATL_JDBC_URL=jdbc:postgresql://db:5432/bitbucketdb
      - ATL_JDBC_USER=bitbucket
      - ATL_JDBC_PASSWORD=bitbucketbitbucket
      - ATL_DB_DRIVER=org.postgresql.Driver
      - ATL_DB_TYPE=postgresql
      - ELASTICSEARCH_URL=http://elasticsearch:9200
      - BITBUCKET_HOME=/var/atlassian/application-data/bitbucket
      - ATL_PRODUCT_HOME_SHARED=/var/atlassian/application-data/bitbucket/shared
      - JVM_SUPPORT_RECOMMENDED_ARGS=-Dcluster.node.name=bitbucket1
      - HAZELCAST_GROUP_NAME=bb_group
      - HAZELCAST_GROUP_PASSWORD=1234
      - HAZELCAST_NETWORK_MULTICAST=true
    networks:
      - bitbucketnet

  bitbucket_node_2:
    platform: linux/amd64  # Ensure correct architecture
    depends_on:
      - db
      - elasticsearch
      - bitbucket_node_1
    image: atlassian/bitbucket-server:latest
    container_name: bitbucket_node_2
    volumes:
      - /Users/Shared/NFS4:/var/atlassian/application-data/bitbucket/shared
    ports:
      - '7992:7990'
    working_dir: /opt/atlassian/bitbucket  # Set the correct working directory
    command: ["./bin/start-bitbucket.sh", "-fg"]
    environment:
      - ATL_JDBC_URL=jdbc:postgresql://db:5432/bitbucketdb
      - ATL_JDBC_USER=bitbucket
      - ATL_JDBC_PASSWORD=bitbucketbitbucket
      - ATL_DB_DRIVER=org.postgresql.Driver
      - ATL_DB_TYPE=postgresql
      - ELASTICSEARCH_URL=http://elasticsearch:9200
      - BITBUCKET_HOME=/var/atlassian/application-data/bitbucket
      - ATL_PRODUCT_HOME_SHARED=/var/atlassian/application-data/bitbucket/shared
      - JVM_SUPPORT_RECOMMENDED_ARGS=-Dcluster.node.name=bitbucket2
      - HAZELCAST_GROUP_NAME=bb_group
      - HAZELCAST_GROUP_PASSWORD=1234
      - HAZELCAST_NETWORK_MULTICAST=true
    networks:
      - bitbucketnet

  db:
    image: postgres:15
    networks:
      - bitbucketnet
    volumes:
      - postgresqldata:/var/lib/postgresql/data
      - ./init-db.sh:/docker-entrypoint-initdb.d/init-db.sh
    ports:
      - '5432:5432'
    environment:
      - POSTGRES_USER=bitbucket
      - POSTGRES_PASSWORD=bitbucketbitbucket
      - POSTGRES_DB=bitbucketdb

  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.10.2
    container_name: elasticsearch
    environment:
      - discovery.type=single-node
      - network.host=0.0.0.0
      - node.max_local_storage_nodes=2
    volumes:
      - elasticsearchdata:/usr/share/elasticsearch/data
    ports:
      - '9200:9200'
    networks:
      - bitbucketnet

  nginx:
    image: nginx:latest
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    ports:
      - "80:80"
      - "7990:7990"
    depends_on:
      - bitbucket_node_1
      - bitbucket_node_2
    networks:
      - bitbucketnet

volumes:
  postgresqldata:
  elasticsearchdata:

networks:
  bitbucketnet:
    driver: bridge


```

```nginx.config```:

```bash
events {}

stream {
    upstream bitbucketnet {
        least_conn;
        server bitbucket_node_1:7990;
        server bitbucket_node_2:7990;
    }

    server {
        listen 7990;
        proxy_pass bitbucketnet;
    }
}


```

```init-db.sh```:

```bash
#!/bin/bash
psql -v ON_ERROR_STOP=1 --username "$POSTGRES_USER" --dbname "$POSTGRES_DB" <<-EOSQL
    GRANT ALL PRIVILEGES ON DATABASE bitbucketdb TO bitbucket;
    GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO bitbucket;
    GRANT ALL PRIVILEGES ON ALL SEQUENCES IN SCHEMA public TO bitbucket;
    GRANT ALL PRIVILEGES ON ALL FUNCTIONS IN SCHEMA public TO bitbucket;
EOSQL

```


# Case 4 - JIRA Monitoring

> The purpose of this case is to monitor and interpret the activities occurring in `JIRA`.
    Prometheus and Grafana monitoring tools are expected to be used.
    Prometheus and Grafana should be powered up as a service with Docker Compose.
    JIRA needs to be installed.
    Plug-in can be installed in JIRA to receive the data stream in JIRA.
    Bonus:
    If the JIRA environment is monitored, it is expected that Docker will be monitored with the same monitoring tools

## Implementation

I created the `docker-compose.yml` file to orchestrate the deployment of multiple services, including `JIRA`, `PostgreSQL`, `NGINX`, `Prometheus`, and `Grafana`. I used Docker Compose to define each service, specifying the necessary images, volumes, ports, and environment variables. By setting up a bridge network, I ensured that all services could communicate effectively, particularly for monitoring purposes. The file is designed to manage these services cohesively, allowing `JIRA` to run in a clustered mode while being monitored by `Prometheus` and visualized in Grafana. I used the `Prometheus.yml` file to configure how `Prometheus` scrapes metrics from the various services deployed via Docker Compose. I created specific scrape jobs for `Prometheus` itself, `JIRA`, and Grafana, defining the intervals and endpoints for metric collection. The `JIRA` job points to the `Prometheus` Exporter plugin’s endpoint within `JIRA`, enabling `Prometheus` to gather detailed metrics about the `JIRA` instance. This configuration allows `Prometheus` to continuously monitor the health and performance of these services and make the data available for visualization in Grafana.

```docker-compose.yml```:

```bash
version: '3'

services:
  jira1:
    depends_on:
      - db
    image: atlassian/jira-software:latest
    container_name: jira1
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:rw
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
      - /Users/Shared/NFS:/var/atlassian/application-data/jira/shared
    ports:
      - '8081:8080'
    expose:
      - '8080'
    environment:
      - CLUSTERED=true
      - JIRA_NODE_ID=jira1
      - ATL_JDBC_URL=jdbc:postgresql://db/jiradb
      - ATL_JDBC_USER=jira
      - ATL_JDBC_PASSWORD=jirajira
      - ATL_DB_DRIVER=org.postgresql.Driver
      - ATL_DB_TYPE=postgres72
      - ATL_CLUSTER_NAME=JIRACluster
      - JVM_SUPPORT_RECOMMENDED_ARGS="-Dupm.plugin.upload.enabled=true"
    networks:
      - monitoring
      - jiranet

  db:
    image: postgres:15
    networks:
      - jiranet
    volumes:
      - ./posgresqldata:/var/lib/postgresql/data
      - ./init-db.sh:/docker-entrypoint-initdb.d/init-db.sh
    ports:
      - '5432:5432'
    environment:
      - 'POSTGRES_USER=jira'
      - 'POSTGRES_PASSWORD=jirajira'
      - 'POSTGRES_DB=jiradb'

  nginx:
    image: nginx:latest
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    ports:
      - "80:80"
      - "8080:8080"
    depends_on:
      - jira1
    networks:
      - jiranet

  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    restart: unless-stopped
    user: root
    volumes:
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
      - ./prometheus-data:/prometheus/
    expose:
      - 9090
    ports:
      - 9090:9090
    links:
      - jira1:jira1
      - grafana:grafana
    networks:
      - monitoring

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    user: root
    restart: unless-stopped
    volumes:
      - ./grafana:/var/lib/grafana
    ports:
      - 3000:3000
    expose:
      - 3000
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=123456
      - GF_USERS_ALLOW_SIGN_UP=true
    networks:
      - monitoring

networks:
  jiranet:
    driver: bridge
  monitoring:
    driver: bridge

volumes:
  posgresqldata:

```

```nginx.config```:

```bash
events {}

stream {
    upstream JIRAnet {
        least_conn;
        server JIRA1:8080;
    }

    server {
        listen 8080;
        proxy_pass JIRAnet;
    }
}

```

```Prometheus.yml```:

```bash
global:
  scrape_interval: 1m
  evaluation_interval: 1m

scrape_configs:
  - job_name: 'prometheus'
    scrape_interval: 1m
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'jira1'
    scrape_interval: 1m
    metrics_path: '/plugins/servlet/prometheus/metrics'
    params:
      token: ['case-1']
    static_configs:
      - targets: ['jira1:8080']

  - job_name: 'grafana'
    scrape_interval: 1m
    static_configs:
      - targets: ['grafana:3000']
    metrics_path: '/metrics'


```


