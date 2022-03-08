# Install Wazuh & ELK Stack (Elasticsearch, Logstash & Kibana)
Wazuh SIEM with ELK Stack in Production Deployment mode to monitoring VPS

Before installing wazuh & els stack you should read this documentation to know about requirements
https://documentation.wazuh.com/current/installation-guide/requirements.html



1. Clone the Wazuh repository to your system:

		git clone https://github.com/wazuh/wazuh-docker.git -b v4.1.5 --depth=1
    
2. Change to wazuh docker directory:

		cd wazuh-docker
    
3. Edit default configuration of **production-cluster.yml**. You can delete all default configuration and paste new configuration (CTRL + K) to delete per line conf
  
 		nano prodution-cluster.yml

4. You can change default with this conf.

        # Wazuh App Copyright (C) 2021 Wazuh Inc. (License GPLv2)
        version: '3.7'

        services:
          wazuh-master:
            image: wazuh/wazuh-odfe:4.1.5
            hostname: wazuh-master
            restart: always
            ports:
              - "1515:1515"
              - "514:514/udp"
              - "55000:55000"
            environment:
              - DISABLE_INSTALL_DEMO_CONFIG=true
              - ELASTICSEARCH_URL=https://elasticsearch:9200
              - ELASTIC_USERNAME=edradmin
              - ELASTIC_PASSWORD=Superm@n123
              - FILEBEAT_SSL_VERIFICATION_MODE=none
              - SSL_CERTIFICATE_AUTHORITIES=/etc/ssl/root-ca.pem
              - SSL_CERTIFICATE=/etc/ssl/filebeat.pem
              - SSL_KEY=/etc/ssl/filebeat.key
              - API_USERNAME=acme-user
              - API_PASSWORD=MyS3cr37P450r.*-
            volumes:
              - ossec-api-configuration-new:/var/ossec/api/configuration
              - ossec-etc-new:/var/ossec/etc
              - ossec-logs-new:/var/ossec/logs
              - ossec-queue-new:/var/ossec/queue
              - ossec-var-multigroups-new:/var/ossec/var/multigroups
              - ossec-integrations-new:/var/ossec/integrations
              - ossec-active-response-new:/var/ossec/active-response/bin
              - ossec-agentless-new:/var/ossec/agentless
              - ossec-wodles-new:/var/ossec/wodles
              - filebeat-etc-new:/etc/filebeat
              - filebeat-var-new:/var/lib/filebeat
              - ./production_cluster/ssl_certs/root-ca.pem:/etc/ssl/root-ca.pem
              - ./production_cluster/ssl_certs/filebeat.pem:/etc/ssl/filebeat.pem
              - ./production_cluster/ssl_certs/filebeat.key:/etc/ssl/filebeat.key
              - ./production_cluster/wazuh_cluster/wazuh_manager.conf:/wazuh-config-mount/etc/ossec.conf

          wazuh-worker:
            image: wazuh/wazuh-odfe:4.1.5
            hostname: wazuh-worker
            restart: always
            environment:
              - DISABLE_INSTALL_DEMO_CONFIG=true
              - ELASTICSEARCH_URL=https://elasticsearch:9200
              - ELASTIC_USERNAME=edradmin
              - ELASTIC_PASSWORD=Superm@n123
              - FILEBEAT_SSL_VERIFICATION_MODE=full
              - SSL_CERTIFICATE_AUTHORITIES=/etc/ssl/root-ca.pem
              - SSL_CERTIFICATE=/etc/ssl/filebeat.pem
              - SSL_KEY=/etc/ssl/filebeat.key
            volumes:
              - worker-ossec-api-configuration-new:/var/ossec/api/configuration
              - worker-ossec-etc-new:/var/ossec/etc
              - worker-ossec-logs-new:/var/ossec/logs
              - worker-ossec-queue-new:/var/ossec/queue
              - worker-ossec-var-multigroups-new:/var/ossec/var/multigroups
              - worker-ossec-integrations-new:/var/ossec/integrations
              - worker-ossec-active-response-new:/var/ossec/active-response/bin
              - worker-ossec-agentless-new:/var/ossec/agentless
              - worker-ossec-wodles-new:/var/ossec/wodles
              - worker-filebeat-etc-new:/etc/filebeat
              - worker-filebeat-var-new:/var/lib/filebeat
              - ./production_cluster/ssl_certs/root-ca.pem:/etc/ssl/root-ca.pem
              - ./production_cluster/ssl_certs/filebeat.pem:/etc/ssl/filebeat.pem
              - ./production_cluster/ssl_certs/filebeat.key:/etc/ssl/filebeat.key
              - ./production_cluster/wazuh_cluster/wazuh_worker.conf:/wazuh-config-mount/etc/ossec.conf

          elasticsearch:
            image: amazon/opendistro-for-elasticsearch:1.13.2
            hostname: elasticsearch
            restart: always
            ports:
              - "9200:9200"
            environment:
              - "ES_JAVA_OPTS=-Xms4g -Xmx4g"
            ulimits:
              memlock:
                soft: -1
                hard: -1
              nofile:
                soft: 65536
                hard: 65536
            volumes:
              - elastic-data-1-new:/usr/share/elasticsearch/data
              - ./production_cluster/ssl_certs/root-ca.pem:/usr/share/elasticsearch/config/root-ca.pem
              - ./production_cluster/ssl_certs/node1.key:/usr/share/elasticsearch/config/node1.key
              - ./production_cluster/ssl_certs/node1.pem:/usr/share/elasticsearch/config/node1.pem
              - ./production_cluster/elastic_opendistro/elasticsearch-node1.yml:/usr/share/elasticsearch/config/elasticsearch.yml
              - ./production_cluster/elastic_opendistro/internal_users.yml:/usr/share/elasticsearch/plugins/opendistro_security/securityconfig/internal_users.yml

          elasticsearch-2:
            image: amazon/opendistro-for-elasticsearch:1.13.2
            hostname: elasticsearch-2
            restart: always
            environment:
              - DISABLE_INSTALL_DEMO_CONFIG=true
              - "ES_JAVA_OPTS=-Xms2g -Xmx2g"
            ulimits:
              memlock:
                soft: -1
                hard: -1
              nofile:
                soft: 65536
                hard: 65536
            volumes:
              - elastic-data-2-new:/usr/share/elasticsearch/data
              - ./production_cluster/ssl_certs/root-ca.pem:/usr/share/elasticsearch/config/root-ca.pem
              - ./production_cluster/ssl_certs/node2.key:/usr/share/elasticsearch/config/node2.key
              - ./production_cluster/ssl_certs/node2.pem:/usr/share/elasticsearch/config/node2.pem
              - ./production_cluster/elastic_opendistro/elasticsearch-node2.yml:/usr/share/elasticsearch/config/elasticsearch.yml
              - ./production_cluster/elastic_opendistro/internal_users.yml:/usr/share/elasticsearch/plugins/opendistro_security/securityconfig/internal_users.yml

          elasticsearch-3:
            image: amazon/opendistro-for-elasticsearch:1.13.2
            hostname: elasticsearch-3
            restart: always
            environment:
              - DISABLE_INSTALL_DEMO_CONFIG=true
              - "ES_JAVA_OPTS=-Xms2g -Xmx2g"
            ulimits:
              memlock:
                soft: -1
                hard: -1
              nofile:
                soft: 65536
                hard: 65536
            volumes:
              - elastic-data-3-new:/usr/share/elasticsearch/data
              - ./production_cluster/ssl_certs/root-ca.pem:/usr/share/elasticsearch/config/root-ca.pem
              - ./production_cluster/ssl_certs/node3.key:/usr/share/elasticsearch/config/node3.key
              - ./production_cluster/ssl_certs/node3.pem:/usr/share/elasticsearch/config/node3.pem
              - ./production_cluster/elastic_opendistro/elasticsearch-node3.yml:/usr/share/elasticsearch/config/elasticsearch.yml
              - ./production_cluster/elastic_opendistro/internal_users.yml:/usr/share/elasticsearch/plugins/opendistro_security/securityconfig/internal_users.yml

          kibana:
            image: wazuh/wazuh-kibana-odfe:4.1.5
            hostname: kibana
            restart: always
            ports:
              - 5601:5601
            environment:
              - DISABLE_INSTALL_DEMO_CONFIG=true
              - ELASTICSEARCH_USERNAME=edradmin
              - ELASTICSEARCH_PASSWORD=Superm@n123
              - SERVER_SSL_ENABLED=true
              - SERVER_SSL_CERTIFICATE=/usr/share/kibana/config/kibana.pem
              - SERVER_SSL_KEY=/usr/share/kibana/config/kibana.key
              - WAZUH_API_URL="https://wazuh-master"
              - API_USERNAME=acme-user
              - API_PASSWORD=MyS3cr37P450r.*-
            volumes:
              - ./production_cluster/ssl_certs/root-ca.pem:/usr/share/kibana/config/root-ca.pem
              - ./production_cluster/ssl_certs/kibana.key:/usr/share/kibana/config/kibana.key
              - ./production_cluster/ssl_certs/kibana.pem:/usr/share/kibana/config/kibana.pem

            depends_on:
              - elasticsearch
            links:
              - elasticsearch:elasticsearch
              - wazuh-master:wazuh-master

          nginx:
            image: nginx:stable
            hostname: nginx
            restart: always
            ports:
              - "805:80"
              - "4435:443"
              - "1514:1514"
            depends_on:
              - wazuh-master
              - wazuh-worker
              - kibana
            links:
              - wazuh-master:wazuh-master
              - wazuh-worker:wazuh-worker
              - kibana:kibana
            volumes:
              - ./production_cluster/nginx/nginx.conf:/etc/nginx/nginx.conf:ro
              - ./production_cluster/ssl_certs/root-ca.pem:/etc/nginx/ssl/root-ca.pem
              - ./production_cluster/ssl_certs/nginx.key:/etc/nginx/ssl/nginx.key
              - ./production_cluster/ssl_certs/nginx.pem:/etc/nginx/ssl/nginx.pem      

        volumes:
          ossec-api-configuration-new:
          ossec-etc-new:
          ossec-logs-new:
          ossec-queue-new:
          ossec-var-multigroups-new:
          ossec-integrations-new:
          ossec-active-response-new:
          ossec-agentless-new:
          ossec-wodles-new:
          filebeat-etc-new:
          filebeat-var-new:
          worker-ossec-api-configuration-new:
          worker-ossec-etc-new:
          worker-ossec-logs-new:
          worker-ossec-queue-new:
          worker-ossec-var-multigroups-new:
          worker-ossec-integrations-new:
          worker-ossec-active-response-new:
          worker-ossec-agentless-new:
          worker-ossec-wodles-new:
          worker-filebeat-etc-new:
          worker-filebeat-var-new:
          elastic-data-1-new:
          elastic-data-2-new:
          elastic-data-3-new:
			

5. Next step is change default **certs.yml** config
      
     	 cd /production_cluster/ssl_certs

6. than,

       nano certs.yml

7. Now, you can change default **certs.yml** config with the configuration below

        ca:
           root:
              dn: CN=root-ca,OU=CA,O=Example\, Inc.,DC=example,DC=com
              pkPassword: none
              keysize: 2048
              file: root-ca.pem 
           intermediate:
              dn: CN=intermediate,OU=CA,O=Example\, Inc.,DC=example,DC=com
              keysize: 2048
              validityDays: 3650  
              pkPassword: intermediate-ca-password
              file: intermediate-ca.pem

        nodes:
          - name: node1
            dn: CN=node1,OU=Ops,O=Example\, Inc.,DC=example,DC=com
            dns: 
              - elasticsearch
          - name: node2
            dn: CN=node2,OU=Ops,O=Example\, Inc.,DC=example,DC=com
            dns: 
              - elasticsearch-2
          - name: node3
            dn: CN=node3,OU=Ops,O=Example\, Inc.,DC=example,DC=com
            dns: 
              - elasticsearch-3
          - name: filebeat
            dn: CN=filebeat,OU=Ops,O=Example\, Inc.,DC=example,DC=com
            dns: 
              - wazuh 
          - name: kibana
            dn: CN=kibana,OU=Ops,O=Example\, Inc.,DC=example,DC=com
            dns:
              - kibana
          - name: nginx
            dn: CN=nginx,OU=Ops,O=Example\, Inc.,DC=example,DC=com
            dns:
              - nginx

8. Back to the **production_cluster** directory and move to nginx directory
 
        cd ../nginx
        
9. Change default nginx.conf config

        user  nginx;
        worker_processes  1;

        error_log  /var/log/nginx/error.log warn;
        pid        /var/run/nginx.pid;


        events {
            worker_connections  1024;
        }


        http {
            include       /etc/nginx/mime.types;
            default_type  application/octet-stream;

            log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                              '$status $body_bytes_sent "$http_referer" '
                              '"$http_user_agent" "$http_x_forwarded_for"';

            access_log  /var/log/nginx/access.log  main;

            sendfile        on;
            tcp_nopush     on;

            keepalive_timeout  65;

            server_tokens off;
            gzip  on;

            # kibana UI
            server {
                listen 80;
                listen [::]:80;
                return 301 https://$host:443$request_uri;
            }

            server {
                listen 443 default_server ssl http2;
                listen [::]:443 ssl http2;
                ssl_certificate /etc/nginx/ssl/nginx.pem;
                ssl_certificate_key /etc/nginx/ssl/nginx.key;
                location / {
                    proxy_pass https://kibana:5601/;
                    proxy_ssl_verify off;
                    proxy_buffer_size          128k;
                    proxy_buffers              4 256k;
                    proxy_busy_buffers_size    256k;
                }
            }

        }



        # load balancer for Wazuh cluster
        stream {
            upstream mycluster {
                hash $remote_addr consistent;
                server wazuh-master:1514;
                server wazuh-worker:1514;
            }
            server {
                listen 1514;
                proxy_pass mycluster;
            }
        }

10. Back to wazuh-docker directory and now you can generate certificate for (**Elasticsearch, Kibana, Wazuh and Nginx**). Previously we have made the configuration in **certs.yml**. This configuration will generate a certificate. (You have to on wazuh-docker directory !!!). Than, run command below

        docker-compose -f generate-opendistro-certs.yml run --rm generator

11. Now you can see that the certificate has been generated. Check in **cd /production_cluster/ssl_certs/**

12. Generate hash from Elasticsearch

        docker run --rm -ti amazon/opendistro-for-elasticsearch:1.13.2 bash
        
        cd /usr/share/elasticsearch/plugins/opendistro_security/tools/
        
        chmod +x hash.sh
        
        ./hash.sh

13. Copy generated hash

14. Paste the generated hash to internal_users.yml. For example like this.

        admin:
          hash: "$2y$12$C5p6S9ZGjjKLl6ot1BBB9e3BdNW3TsJ0hxpLh9jhdshbjkducCrlTyoy"
          reserved: true
          backend_roles:
          - "admin"
          description: "Demo admin user"

15. Yupsssss. Now you can run compose production-cluster.yml

          docker-compose -f production-cluster.yml up
          
16. Wait a moment until pull & run docker container is finished. After that you can access you service with https://youripaddress

17. The next step is to add wazuh and filebeat agents to send logs. 
19. You can see this documentation from wazuh docs https://documentation.wazuh.com/current/installation-guide/wazuh-agent/wazuh-agent-package-linux.html
