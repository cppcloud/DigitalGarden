---
{"dg-publish":true,"tags":["tools"],"permalink":"/Tools/Docker搭建NewApi/","dgPassFrontmatter":true}
---

使用 docker-compose 和 mysql 5.7 搭建

```bash
services:
  mysql:
    image: mysql:5.7
    container_name: mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: 123456
      MYSQL_DATABASE: mydatabase
    ports:
      - "3306:3306"
    volumes:
      - mysql_data:/var/lib/mysql
      - ./my.cnf:/etc/mysql/conf.d/custom.cnf
    command: --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 3
    networks:
      - app-network

  new-api:
    image: calciumion/new-api:latest
    container_name: new-api
    restart: always
    ports:
      - "3000:3000"
    environment:
      - SQL_DSN=root:123456@tcp(mysql:3306)/mydatabase?charset=utf8mb4&collation=utf8mb4_unicode_ci&parseTime=True&loc=Local
      - TZ=Asia/Shanghai
      # 添加其他环境变量，如有需要
    volumes:
      - /www/wwwroot/new-api:/data
    depends_on:
      mysql:
        condition: service_healthy
    networks:
      - app-network

volumes:
  mysql_data:

networks:
  app-network:
    driver: bridge

```