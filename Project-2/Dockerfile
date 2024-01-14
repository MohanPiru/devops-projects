FROM tomcat:8.0-alpine
LABEL maintainer="satheeskumar.pmp@gmail.com"

ADD target/*.war /usr/local/tomcat/webapps/

EXPOSE 8000
CMD ["catalina.sh", "run"]
