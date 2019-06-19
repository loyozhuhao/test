```
version: '3'  
services:  

  elasticsearch:
    image: 'registry.ukuaiqi.com/library/elasticsearch:x-pack'
    command: [ elasticsearch, -E, network.host=0.0.0.0, -E, discovery.zen.ping.unicast.hosts=elasticsearch, -E, discovery.zen.minimum_master_nodes=1, -E, node.max_local_storage_nodes=3, -E, xpack.security.enabled=false ]
    volumes:
      - /mnt/es/data:/usr/share/elasticsearch/data
    deploy:
      mode: 'global'
      placement:
        constraints: [node.labels.app_role == elasticsearch]

  nginx:
    image: 'registry.ukuaiqi.com/library/nginx'
    ports:
        - '9200:9200'
    command: |
      /bin/bash -c "echo '
      server {
        listen 9200;
        client_max_body_size 50m;
        add_header X-Frame-Options "SAMEORIGIN";
        location / {
            proxy_pass http://elasticsearch:9200;
            proxy_http_version 1.1;
            proxy_set_header Connection keep-alive;
            proxy_set_header Upgrade $$http_upgrade;
            proxy_set_header Host $$host;
            proxy_set_header X-Real-IP $$remote_addr;
            proxy_cache_bypass $$http_upgrade;
        }
      }' | tee /etc/nginx/conf.d/default.conf && nginx -g 'daemon off;'"
    deploy:
      mode: 'global'
```
