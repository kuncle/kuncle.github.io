---
layout: post
title:  "Gradle Avro Plugin Guide"
date:   2019-07-18 13:25:00
categories: Other
tags: Other
---
Add below script into your build.gradle file.
``` java
apply plugin: "com.commercehub.gradle.plugin.avro-base"  
  
buildscript {  
    repositories {  
        maven {  
            url "xxx"  
            credentials{  
                username=xxx  
                password=xxx  
            }  
        }  
        maven {  
            url "xxx"  
        }  
    }  
    dependencies {  
        classpath 'com.commercehub.gradle.plugin:gradle-avro-plugin:0.16.0'  
    }  
}  
  
dependencies {  
    compile group: 'org.codehaus.jackson', name: 'jackson-mapper-asl', version: '1.9.13'  
}  
  
task generateAvro(type: com.commercehub.gradle.plugin.avro.GenerateAvroJavaTask) {  
    source("src/main/avro")  
    outputDir = file("src/main/java")  
}  
```
If you are using IntelliJ IDEA, you can enter the Gradle view and click the Tasks -> other -> generateAvro button to generate a Java bean from the avsc file you provided.
