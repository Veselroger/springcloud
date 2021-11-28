# <a id="home"></a> Spring Cloud Workbook

Данный репозиторий посвящён конспекту учебного курса **"[JVA-043 Spring Cloud](https://www.luxoft-training.ru/kurs/spring_cloud_dlya_java-razrabotchikov.html?sphrase_id=1131434)"**.

## Table of Contents:
- Preparation
    - [Maven Project](#maven)
    - [Dependencies](#dependencies)
    - [Executable](#executable)
    - [Heroku](#heroku)
- Modules
    - [Eureka](./cloud-eureka/eureka.md)

----

## [↑](#home) <a id="maven"></a> Maven Project
Для сборки проектов мы будем использовать систему сборки **"[Maven](https://maven.apache.org/download.cgi)"**.\
Maven проект описывается при помощи Project Object Model, она же POM. Описание хранится в одноимённом XML файле **pom.xml**.

Для создания минимального maven проекта нам понадобится написать свой **"[Minimal POM](https://maven.apache.org/guides/introduction/introduction-to-the-pom.html#Minimal_POM)"**.\
**Minimal POM** описывает только версию синтаксиса pom файла, а так же **"[Maven Coordinates](https://maven.apache.org/pom.html#Maven_Coordinates)"** проекта.

Пример **pom.xml** файла:
```xml
<project>
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.github.veselroger</groupId>
  <artifactId>cloud-workbook</artifactId>
  <version>1</version>
  <packaging>pom</packaging>
</project>
```

Наше Cloud решение будет состоять из разных аспектов, каждый из которых - это отдельный Maven модуль.\
Проект, который их агрегируют называется **Aggregator project** и его тип **packaging** будет **pom**.\
Подробнее про это описано в документации Maven: **"[Aggregation (Multi-Module Project)](https://maven.apache.org/pom.html#aggregation-or-multi-module)"**.

Кроме того, хорошим тоном является использование **"[Maven Properties](https://maven.apache.org/pom.html#Properties)"**. Можно добавить настройки из [традиционного примера](https://maven.apache.org/plugins/maven-compiler-plugin/examples/set-compiler-source-and-target.html):
```xml
<properties>
  <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  <maven.compiler.source>1.8</maven.compiler.source>
  <maven.compiler.target>1.8</maven.compiler.target>
</properties>
```

## [↑](#home) <a id="dependencies"></a> Dependencies
Наш проект будет использовать некоторые проекты Spring.\
На официальном сайте **spring.io** можно найти [список проектов](https://spring.io/projects).\
Для каждого проекта в разделе **Learn** доступна документация - **Reference Documentation**.

В основе всего будет **"[Spring Boot](https://spring.io/projects/spring-boot)"**.\
На странице проекта Spring Boot откроем **"[Spring Boot Reference Documentation](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/)"** для текущей версии.\
При помощи CTRL+F найдём упоминание **Build systems** и **Maven**, перейдём в **"[Spring Boot Maven Plugin](https://docs.spring.io/spring-boot/docs/current/maven-plugin/reference/htmlsingle/)"**. 

Настроим **[Dependency Management](https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html)"** нашего проекта.\
Spring Boot можно подключить с использованием Parent POM и без.\
Нас интересует **"[Using Spring Boot without the Parent POM](https://docs.spring.io/spring-boot/docs/current/maven-plugin/reference/htmlsingle/#using.import)"**. 

Добавим в блоке properties новую запись, например:
```xml
<properties>
  <spring.boot-version>2.6.0</spring.boot-version>
```

После этого добавим в pom.xml блок **"dependencyManagement"** согласно документации:
```xml
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-dependencies</artifactId>
      <version>${spring.boot-version}</version>
      <type>pom</type>
      <scope>import</scope>
    </dependency>
  </dependencies>
</dependencyManagement>
```

Также наш проект будет использовать **[Spring Cloud](https://spring.io/projects/spring-cloud)**. Поэтому на странице описания проекта проскролим немного вниз и найдём указание того, как нужно настроить **dependencyManagement** для Spring Cloud.

Добавим настройку для версии Spring Cloud:
```xml
<properties>
  <spring.boot-version>2.6.0</spring.boot-version>
  <spring.cloud-version>Hoxton.SR8</spring.cloud-version>
```

И далее в блок **dependencyManagement** добавим зависимость с описанием версий Spring Cloud проектов:
```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-dependencies</artifactId>
  <version>${spring.cloud-version}</version>
  <type>pom</type>
  <scope>import</scope>
</dependency>
```

Теперь стоит присмотреться к блоку properties в pom.xml и проверить, совместимы ли версии Spring Boot и Spring Cloud. Воспользуемся таблицей **"[Table 1. Release train Spring Boot compatibility](https://spring.io/projects/spring-cloud)"**.

Hа момент написания этого конспекта из таблицы следует, что самая последняя из поддерживаемых версий Spring Boot - 2.5.x. На странице проекта Spring Boot в разделе разделе [Learn](https://spring.io/projects/spring-boot#learn) видно, что ``spring.boot-version`` должна быть ``2.5.7``. Из таблицы следует, что для ``spring.cloud-version`` следует выбрать ``2020.0.3``. Таким образом: 
```xml
<properties>
  <spring.boot-version>2.5.7</spring.boot-version>
  <spring.cloud-version>2020.0.3</spring.cloud-version>
```

## [↑](#home) <a id="executable"></a> Executable
Разворачивание приложений из IDE и в реальности отличаются.\
Чтобы приложения развернуть удалённо они должны быть Executable.

Выполним действия, указанные в документации Spring Boot в разделе **"[Packaging Executable Archives](https://docs.spring.io/spring-boot/docs/current/maven-plugin/reference/htmlsingle/#packaging)"**:
```xml
<build>
  <plugins>
    <plugin>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-maven-plugin</artifactId>
      <version>${spring.boot-version}</version>
      <executions>
        <execution>
          <goals>
            <goal>repackage</goal>
          </goals>
        </execution>
      </executions>
    </plugin>
  </plugins>
</build>
```

## [↑](#home) <a id="heroku"></a> Heroku
Чтобы в учебных целях протестировать проект воспользуемся **[Heroku](https://dashboard.heroku.com/)**.\
Для работы с Heroku нам потребуется создать [учётную запись на Heroku](http://dashboard.heroku.com)
и установить **"[Heroku CLI](https://devcenter.heroku.com/articles/heroku-cli)"**. Удобнее всего использовать "**[Tarballs](https://devcenter.heroku.com/articles/heroku-cli#tarballs)**". Распаковываем tarball в каталог и добавляем подкаталог **bin** в переменную окружения PATH.

По правилам Heroku для разворачивания приложения в корне проекта требуется специальный файл **"[Procfile](https://devcenter.heroku.com/articles/procfile)"**. Как сказано в документации:
> The Procfile must live in your app’s root directory. It does not function if placed anywhere else.

В данном файле добавим строчку запуска приложений. Так как у каждого приложения свой собственный запуск, то создадим Procfile "обезличенным":
```
web: java -Dserver.port=$PORT -jar $JAR_PATH
```
