# Spring Cloud Workbook

Данный репозиторий посвящён конспекту учебного курса **"[JVA-043 Spring Cloud](https://www.luxoft-training.ru/kurs/spring_cloud_dlya_java-razrabotchikov.html?sphrase_id=1131434)"**.

## Table of Contents:
- [Intro](#intro)
- [Eureka](./cloud-eureka/eureka.md)

----

## [↑](#home) <a id="intro"></a> Intro
Для сборки проектов мы будем использовать систему сборки **"[Maven](https://maven.apache.org/download.cgi)"**.\
Maven проект описывается при помощи Project Object Model, она же pom. Описание хранится в одноимённом XML файле **pom.xml**.

Для создания минимального maven проекта нам понадобится написать свой **"[Minimal POM](https://maven.apache.org/guides/introduction/introduction-to-the-pom.html#Minimal_POM)"**, который описывает только версию синтаксиса pom файла, а так же **"[Maven Coordinates](https://maven.apache.org/pom.html#Maven_Coordinates)"** проекта.\
Например:
```xml
<project>
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.github.veselroger</groupId>
  <artifactId>cloud-workbook</artifactId>
  <version>1</version>
  <packaging>pom</packaging>
</project>
```

Мы будем создавать многомодульный проект, где каждый аспект нашего Cloud решения будет реализован при помощи отдельного модуля. Наш корневой проект (группирующий) должен быть создан как packaging=pom. Подробнее про это можно прочитать в документации Maven: **"[Aggregation (Multi-Module Project)](https://maven.apache.org/pom.html#aggregation-or-multi-module)"**.

Наш проект будет использовать **"[Spring Boot](https://spring.io/projects/spring-boot)"**. В разделе **Learn** откроем **"[Spring Boot Reference Documentation](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/)"** для текущей версии. При помощи CTRL+F найдём упоминание **Maven** и перейдём в **"[Build Tool Plugins](https://docs.spring.io/spring-boot/docs/current/reference/html/build-tool-plugins.html#build-tool-plugins)"**.

Перейдём в **"[Spring Boot Maven Plugin Documentation](https://docs.spring.io/spring-boot/docs/current/maven-plugin/reference/htmlsingle/)"**. Нас интересует информация из раздела **"[Using Spring Boot without the Parent POM](https://docs.spring.io/spring-boot/docs/current/maven-plugin/reference/htmlsingle/#using.import)"**. Добавим в наш pom.xml блок **"dependencyManagement"** согласно документации.

Кроме того, наш проект будет использовать **[Spring Cloud](https://spring.io/projects/spring-cloud)**. Поэтому на странице описания проекта проскролим немного вниз и найдём указание того, что нужно подключить в нашу секцию **dependencyManagement** для Spring Cloud.

Кроме того, Spring Cloud рекомендует версии указывать через Maven Properties. Например:
```xml
<properties>
    <spring.boot-version>2.6.0</spring.boot-version>
    <spring.cloud-version>Hoxton.SR8</spring.cloud-version>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
</properties>
```

Проекты для теста мы будем разворачивать на бесплатном сервисе [Heroku](https://dashboard.heroku.com/), а значит нам нужно создать специальный файл **"[procfile](https://devcenter.heroku.com/articles/procfile)"**. Создадим его пустым, т.к. его заполнение будет происходить по мере работы над конспектом.

Кроме этого, для работы с Heroku нам потребуется создать [учётную запись на Heroku](http://dashboard.heroku.com)
и установить **"[Heroku CLI](https://devcenter.heroku.com/articles/heroku-cli)"**. Удобнее всего использовать "**[Tarballs](https://devcenter.heroku.com/articles/heroku-cli#tarballs)**". Распаковываем tarball в каталог и добавляем подкаталог bin в переменную PATH.