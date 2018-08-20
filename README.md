# Tourism data collector

An Elasticsearch CSV file manager written in Java 1.8 using Maven for the build.

## Features

* Uses the [ElasticUtils](https://github.com/bytefish/ElasticUtils) library for working with Elasticsearch 5
* Files get processed by [Apache NiFi](https://nifi.apache.org)
* [SQLite database](https://www.sqlite.org) for user management

## Sample CSV File

```
"arrival","departure","country","adults","children","destination","category","booking","cancellation","submitted_on","cancelled_on"
"2015-01-01","2015-01-03","","2","0","21027","1","1","0","2015-01-01 01:59:00",""
"2015-07-03","2015-07-04","","2","0","21008","1","1","0","2015-01-01 11:27:00",""
"2015-07-05","2015-07-07","","2","0","21027","1","0","0","2015-01-01 13:33:00",""
"2015-08-01","2015-08-05","","3","0","21027","1","0","0","2015-04-01 12:00:00",""
"2015-08-01","2015-08-05","","3","0","21027","1","0","0","2015-04-01 12:00:00","2015-04-01 14:00:00"
```

## What you will need

* About 10 minutes
* A favorite text editor or IDE
* JDK 8 or later

## Setting up the application

Download and unzip the latest [Tourism data collector](https://github.com/idm-suedtirol/big-data-for-tourism/archive/master.zip) release.

Install [Maven](http://maven.apache.org/download.cgi) and add the _bin_ folder to your path.

To test the Maven installation, run `mvn -v` from the command line:

If all goes well, you should be presented with some information about the Maven installation.

Now you are ready to configure the application by opening the [application.properties](src/main/resources/application.properties) file:

```
spring.http.multipart.max-file-size=1024MB
spring.http.multipart.max-request-size=1024MB

# ===============================
# = HTTP AUTHENTICATION
# ===============================

# Username and password

security.username=username
security.password=password

# ===============================
# = ELASTICSEARCH
# ===============================

# Set here configurations for the elasticsearch connection

es.endpoint=search-domainname-domainid.us-east-1.es.amazonaws.com
es.port=1234
es.ssl=true
es.index=index
es.type=index-type
es.cluster=cluster
es.user=username:password
es.userdetails=index-userdetails

# ===============================
# = DB SETTINGS
# ===============================

spring.datasource.driverClassName=org.sqlite.JDBC
spring.datasource.url=jdbc:sqlite:tourism-collector.db
spring.datasource.username=
spring.datasource.password=

spring.profiles.active=@activatedProperties@
```

Last but not least, setup your smtp mail server [simplejavamail.properties](src/main/resources/simplejavamail.properties):

```
# ===============================
# = MAIL SERVER
# ===============================

# Set here configurations for the smtp mail server

simplejavamail.transportstrategy=SMTP_PLAIN
simplejavamail.smtp.host=smtp.host
simplejavamail.smtp.port=25
simplejavamail.smtp.username=username
simplejavamail.smtp.password=password
simplejavamail.defaults.subject=Congratulations! Your data has been stored
simplejavamail.defaults.from.name=Tourism Data Collector
simplejavamail.defaults.from.address=from@default.com
```

Congratulations. You are done!

## Running the application

### Using the Maven plugin

Running `mvn spring-boot:run` compiles and runs the application. You should be able to access `http://localhost:8080`.

### Deploy it to an existing Tomcat installation

Making a WAR file is straight forward enough and can be accomplished by executing the following command

`mvn package`

You can now take a look in the ${basedir}/target directory and you will see the generated WAR file.

## Elasticsearch Indices

```json
PUT /index
{
  "mappings": {
    "index-type": {
      "_all": {
        "enabled": false
      },
      "properties": {
        "arrival": {
          "type": "date",
          "format": "epoch_millis||date"
        },
        "departure": {
          "type": "date",
          "format": "epoch_millis||date"
        },
        "country": {
          "properties": {
            "code": {
              "type": "keyword"
            },
            "name": {
              "type": "keyword"
            },
            "latlon": {
              "type": "geo_point"
            }
          }
        },
        "adults": {
          "type": "byte"
        },
        "children": {
          "type": "byte"
        },
        "destination": {
          "properties": {
            "code": {
              "type": "short"
            },
            "name": {
              "type": "keyword"
            },
            "latlon": {
              "type": "geo_point"
            }
          }
        },
        "category": {
          "properties": {
            "code": {
              "type": "byte"
            },
            "name": {
              "type": "keyword"
            }
          }
        },
        "booking": {
          "type": "boolean"
        },
        "cancellation": {
          "type": "boolean"
        },
        "submitted_on": {
          "type": "date",
          "format": "epoch_millis||date_hour_minute_second"
        },
        "cancelled_on": {
          "type": "date",
          "format": "epoch_millis||date_hour_minute_second"
        },
        "length_of_stay": {
          "type": "short"
        },
        "user": {
          "type": "keyword"
        },
        "days_before_arrival": {
          "type": "short"
        },
        "uploaded_on": {
          "type": "date",
          "format": "epoch_millis||date_hour_minute_second"
        }
      }
    }
  }
}
```
