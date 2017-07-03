FROM tomcat:9.0.0.M6-jre8

## Extract ivory into the opt directory.
ADD ivoryserver4j.5.1.1.20160427.tar /opt

## Add license file for ivory.
COPY key.lic /opt/gtsoftware/ivoryserver4j/license/key.lic

## Deploy the ivory war into tomcat.
COPY ivory.war /usr/local/tomcat/webapps/ivory.war
