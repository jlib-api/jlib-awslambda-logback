# jlib AWS Lambda SLF4J/Logback Appender

[![Build Status](https://travis-ci.org/jlib-framework/jlib-awslambda-logback.svg?branch=master)](https://travis-ci.org/jlib-framework/jlib-awslambda-logback)
[![Maven Central](https://maven-badges.herokuapp.com/maven-central/org.jlib/jlib-awslambda-logback/badge.svg)](https://maven-badges.herokuapp.com/maven-central/org.jlib/jlib-awslambda-logback)
[![Javadoc](https://www.javadoc.io/badge/org.jlib/jlib-awslambda-logback.svg)](http://www.javadoc.io/doc/org.jlib/jlib-awslambda-logback)

The _jlib AWS Lambda Logback appender_ library allows to log through [SLF4J](https://www.slf4j.org/)/[Logback](https://logback.qos.ch/) 
to [CloudWatch Logs](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/WhatIsCloudWatchLogs.html) 
from [AWS Lambda](https://aws.amazon.com/de/lambda) code.

#### Usage
##### Dependency
###### Gradle (build.gradle)
    dependencies {
        implementation 'org.slf4j:slf4j-api:1.8.0-beta2'
        runtimeOnly 'org.jlib:jlib-awslambda-logback:1.0.0'
    }
    
###### Maven (pom.xml)
    <dependencies>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.8.0-beta2</version>
        </dependency>
        <dependency>
            <groupId>org.jlib/groupId>
            <artifactId>jlib-awslambda-logback/artifactId>
            <version>1.0.0</version>
            <scope>runtime</scope>
        </dependency>
    </dependencies>

##### Configuration
###### XML (src/main/resources/logback.xml)
    <configuration>
    
        <appender name="awslambda" class="org.jlib.cloud.aws.lambda.logback.AwsLambdaAppender">
            <encoder type="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
                <pattern>[%d{yyyy-MM-dd HH:mm:ss.SSS}] &lt;%-36X{AWSRequestId:-request-id-not-set-by-lambda-runtime}&gt; %-5level %logger{10} - %msg%n</pattern>
            </encoder>
        </appender>
    
        <root level="INFO">
            <appender-ref ref="awslambda" />
        </root>
    
    </configuration>
    
##### Code
To log information from your Lambda application, just get the logger for your class and output the message:

    import org.slf4j.Logger;
    import org.slf4j.LoggerFactory;
    ...
    
    public class MyHandler {
    
        private static final Logger log = LoggerFactory.getLogger(MyHandler.class);
         
        public void handle(Context context) {
            log.info("My lambda is called '{}'.", context.getFunctionName());
            ...
        }
    }

#### Features
##### Multi-line logging to CloudWatch Logs
The library handles stacktraces and other messages spanning across multiple lines. Each multi-line message is treated as a single CloudWatch Log event.

If you were instead writing a multi-line text to the standard output, 
CloudWatch Logs would register every line of this text as a separate event.

To ensure a single event per multi-line message, this library does not write to the standard output. 
It uses the `LambdaLogger` provided by the AWS Lambda SDK,
and CloudWatch Logs treats the whole multi-line message as a single event.
Consequently, the developer does not need to handle newline characters, 
e.g. by replacing them by carriage return characters.

##### Tracing of requests through AWS request id in every log message
When dealing with multiple requests, it can be hard to identify to which request a specific response belongs.
This library helps to trace requests by allowing to include a unique request id to every single log message.
The `AWSRequestId` is provided by the AWS Lambda runtime.
Simply include an MDC reference to this id into the encoder pattern, and the id will be added to every single log message. 
The encoder pattern can be specified in the Logback configuration, e.g. `logback.xml`.
Please refer to the Logback documentation for details on how to use the [MDC](https://logback.qos.ch/manual/mdc.html). 

##### Faster deployment and cold starts
One goal when building Lambda applications should be to keep the application archive as small as possible.
This allows for a faster deployment of the application when uploading its archive to AWS.
It also speeds up the initial loading of the application, also known as _cold start_.

_Up to 700kB_ can be saved when using this library.
When depending on `logback-classic` in Maven or Gradle, 
the dependency tree includes a transitive dependency to `com.sun.mail:javax.mail`.
When building an archive for the Lambda application,
this transitive dependency is included in the archive.
It raises the archive size by around 700kB.
This happens whether the application is packaged as an uber-jar or as a zip archive.

This library excludes this transitive dependency 
in order to minimize the archive size of the Lambda application.

Alternative approaches using other Logging implementations for SLF4J produce archives at least
about 100kB larger than the archive including this library.

##### No extra build information for uber jar
Some logging implementations for SLF4J require additional handling during the build process when creating an uber-jar.
For instance, log4j2 requires the `maven-shade-plugin.log4j2-cachefile-transformer` to be executed while producing the archive.

This library does not require further configuration. Just add the dependency.

#### Disclaimer
“Amazon Web Services", "AWS", "Lambda" and "CloudWatch" are trademarks of Amazon.com, Inc. or its affiliates in the United States and/or other countries.

#### License
Copyright 2018 Igor Akkerman

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

   http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
