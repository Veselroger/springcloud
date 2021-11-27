# [←](../README.md) <a id="home"></a> Eureka

## Table of Contents:
- [Eureka](#eureka)
- [Create Module](#module)
- [Eureka Configuration](#configuration)
- [Deploy](#deploy)

--------

## [↑](#home) <a id="eureka"></a> Eureka
Микросервисная архитектура использует множество различных шаблонов проектирования.\
**[Service Discovery](https://microservices.io/patterns/service-registry.html)** - шаблон проектирования, который предполагает наличие реестра, в котором будут регистрироваться сервисы. Если какой-то из сервисов кому-то нужен, этот сервис можно найти через этот реестр.

Одной из реализаций Service Discovery является **Eureka**.


## [↑](#home) <a id="module"></a> Create Module
Для создания нашего экземпляра Eureka Server нам нужно создать новый **[Maven Module](https://books.sonatype.com/mvnex-book/reference/multimodule-sect-intro.html)**.

Откроем корневой каталог нашего проекта при помощи любимой IDE. В примерах будет использована IntelliJ Idea.\
В главном меню при помощи **"File → New → Module"** создадим новый Maven модуль, назовём его cloud-eureka.

После создания нового модуля нужно указать его как модуль в родительском pom.xml:
```xml
<modules>
  <module>simple-weather</module>
</modules>
```

В pom.xml нашего нового Maven модуля добавим ссылку на родителя:
```xml
<parent>
  <groupId>com.github.veselroger</groupId>
  <artifactId>cloud-workbook</artifactId>
  <version>1</version>
</parent>
```
Это позволит унаследовать настройки из нашего родителя. Например, версию Java и описание версий зависимостей.

Теперь мы можем подключить зависимости для Eureka. Откроем Reference Doc для модуля **"[Spring Cloud Netflix](https://spring.io/projects/spring-cloud-netflix)"**. Нас интересуют Eureka Client и Eureka Server. Поэтому нам нужно подключить их при помощи Spring Boot Starter'ов. В секциях **"How to Include"** указано, какие стартеры нужно подключать.

Таким образом, наши подключённые зависимости могут выглядеть так:
В качестве зависимостей подключим стартер сервера и клиента Эврики:
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
    </dependency>
</dependencies>
```

## [↑](#home) <a id="configuration"></a> Eureka Configuration
Теперь нам нужно настроить Эврику. В каталоге ``src/main/resources`` создадим файл настроек.\
Настройки эврики можно хранить в файле **application.yml**, как указано в примерах в документации. Кроме этого, Spring Cloud вводит понятие **"[Bootstrap Application Context](https://docs.spring.io/spring-cloud/docs/current/reference/htmlsingle/#spring-cloud-context-application-context-services)"**, позволяющий настройки хранить в **bootstrap.yml**.

Добавим настройки Эврики:
```yaml
spring:
  application:
  name: Eureka
server:
  port: 8761
eureka:
  client:
  fetchRegistry: false
  registerWithEureka: false
```

Остаётся только сделать из нашего приложения Spring Boot Application.\
Для этого в каталоге ``src/main/java`` создадим новый Java класс Eureka. Если в IntelliJ Idea такой опции нет, то через контекстное меню каталога можно выбрать **"Mark Directory as → Source Root"**.

По умолчанию наш Maven Archetype создал нам Java класс App. Сделаем из него Spring Boot Application:
```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@SpringBootApplication
@EnableEurekaServer
public class Eureka {
    public static void main(String[] args) {
        SpringApplication.run(Eureka.class, args);
    }
}
```
Благодаря аннотации ``@EnableEurekaServer`` вместе со Spring Boot будет поднят реестр сервисов по настройкам, указанным в конфигурационном yaml файле.

--------

## [↑](#home) <a id="deploy"></a> Deploy
Чтобы завершить настройку Эврики, нам нужно сервер Эврики где-то развернуть.\
Существует большое количество способов, но мы в целях обучения воспользуемся бесплатным **[Heroku](https://blog.heroku.com/managing_your_microservices_on_heroku_with_netflix_s_eureka)**.

Ранее мы подготовили Heroku CLI. Можем открыть терминал (например, в IntelliJ Idea при помощи ATL+F12). Выполним **heroku --version**, чтобы убедиться, что Heroku CLI работает. Если нет - необходимо перепроверить, что переменная среды окружения PATH настроена верно и перезапустить терминал (т.к. переменные среды окружения считываются только один раз при запуске терминала).

Выполним вход на Heroku через Heroku CLI при помощи команды **heroku login**. В случае ошибки вида "[unable to get local issuer certificate](https://devcenter.heroku.com/articles/using-the-cli)" убедитесь, что не использует ПО вроде ZScaler.

Нам необходимо создать новое приложение на Heroku:
> heroku create --app cloud-eureka

Данная команда создаёт приложение на стороне Heroku, а так же git репозиторий на стороне Heroku и добавляет его в качестве remote для текущего каталога, который будет локальным git репозиторием. Например:
```
C:\github\cloud>heroku create --app cloud-eureka
Creating ⬢ cloud-eureka... done
https://cloud-eureka.herokuapp.com/ | https://git.heroku.com/cloud-eureka.git
```

Теперь создадим remote для cloud-eureka:
> heroku git:remote --remote cloud-eureka --app cloud-eureka

Остаётся теперь сделать коммит и отправить его в наш remote:
> git add .
> git commit -m "Add Eureka module"
> git subtree push --prefix cloud-eureka cloud-eureka master




