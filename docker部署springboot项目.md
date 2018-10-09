# docker 部署springboot项目:
* 1 将springboot项目打成jar包。
* 2 创建dockerfile文件，内容如下:
```$xslt
FROM openjdk:8
MAINTAINER JIANG LU
LABEL app="server" version="0.0.1" by="jianglu"
COPY ./server.jar server.jar
CMD java -jar server.jar
```
* 3 执行docker build命令如下:(注意最后的.)
```
docker build -t server .
```
* 4 执行run:
```$xslt
docker run --name server -p 8077:8077 -t server
```
