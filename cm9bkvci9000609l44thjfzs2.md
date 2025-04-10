---
title: "Set Up SonarQube Community Build using Docker Compose"
datePublished: Thu Apr 10 2025 16:33:53 GMT+0000 (Coordinated Universal Time)
cuid: cm9bkvci9000609l44thjfzs2
slug: set-up-sonarqube-community-build-using-docker-compose
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/JKUTrJ4vK00/upload/b2e78f1c7a749253043dd4efea1b1403.jpeg
tags: docker, devops, docker-compose, codequality

---

[SonarQube](https://sonarqube.com) is a code quality tool that helps ensure your team adheres to best practices and can catch bugs before merging into production. It allows you to define rules checked anytime SonarQube is run on your code base. This helps reduce friction during code reviews because the rules for what defines 'good code' in your team are predefined and not subject to one person's opinion. Although Sonarqube is a paid tool, it provides a free community version you can install on your server.

SonarQube can be installed in multiple ways. I prefer Docker Compose because it allows you to run multiple containers easily. It uses Docker, so there is no need to install and maintain versions of packages/libraries. Below is a sample `docker-compose.yaml`

```yaml
services:
  sonarqube:
    image: sonarqube:community
    read_only: true
    volumes:
      - sonarqube_data:/opt/sonarqube/data
      - sonarqube_extensions:/opt/sonarqube/extensions
      - sonarqube_logs:/opt/sonarqube/logs
      - sonarqube_temp:/opt/sonarqube/temp
    ports:
      - "9000:9000"
    networks:
      - sonar-network
    environment:
      SONAR_JDBC_URL: jdbc:{database-url}  
      SONAR_JDBC_USERNAME: XXXXXXXXXX
      SONAR_JDBC_PASSWORD: "XXXXXXXX"
      SONAR_WEB_PORT: 9000
      SONAR_AUTH_JWTBASE64HS256SECRET: "XXXXXXXXXXXXXXXXXXXXX"
      VIRTUAL_HOST: sonarqube.yourdomain.com
      VIRTUAL_PORT: 9000
      LETSENCRYPT_HOST: sonarqube.yourdomain.com

  proxy:
    image: nginxproxy/nginx-proxy:1.6
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - /var/run/docker.sock:/tmp/docker.sock:ro
      - certs:/etc/nginx/certs:ro
      - html:/usr/share/nginx/html
      - conf:/etc/nginx/conf.d
    environment:
      VIRTUAL_HOST: sonarqube.yourdomain.com
      VIRTUAL_PORT: 9000
    networks:
      - sonar-network
      - sonar-public

  acme-companion:
    image: nginxproxy/acme-companion
    container_name: nginx-proxy-acme
    environment:
      - DEFAULT_EMAIL=dev@yourdomain.co
    volumes_from:
      - proxy
    volumes:
      - certs:/etc/nginx/certs:rw
      - acme:/etc/acme.sh
      - /var/run/docker.sock:/var/run/docker.sock:ro
    networks:
    - sonar-public

volumes:
  sonarqube_data:
  sonarqube_extensions:
  sonarqube_logs:
  sonarqube_temp:
  certs:
  html:
  conf:
  acme:

networks:
  sonar-network:
    ipam:
      driver: default
      config:
        - subnet: 172.28.2.0/24
  sonar-public:
    driver: bridge
```

This configuration uses an external database (such as GCP Cloud SQL or AWS RDS) and allows you to access SonarQube using HTTPS. The `acme-companion` service handles auto-renewal of the SSL certificates using Let's Encrypt.