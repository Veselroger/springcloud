# [←](../README.md) <a id="home"></a> Eureka Service Discovery

## Table of Contents:
- [Service Discovery](#discovery)
- [Eureka Server Module](#module)
- [Eureka Server Configuration](#configuration)
- [Eureka Server Instance](#instance)

- [Deploy](#deploy)
- [Security](#security)
--------

## [↑](#home) <a id="discovery"></a> Service Discovery
Микросервисная архитектура использует множество различных шаблонов проектирования.\
**[Service Discovery](https://microservices.io/patterns/service-registry.html)** - шаблон проектирования, который предполагает наличие реестра, в котором будут регистрироваться сервисы. Если какой-то из сервисов кому-то нужен, этот сервис можно найти через этот реестр.

Одной из реализаций Service Discovery является **Netflix Eureka**.


## [↑](#home) <a id="module"></a> Eureka Server Module
Для создания нашего экземпляра Netflix Eureka Server нам нужно создать новый **[Maven Module](https://books.sonatype.com/mvnex-book/reference/multimodule-sect-intro.html)**.

Откроем корневой каталог нашего проекта при помощи IDE и создадим новый Maven модуль.\
Например, в IntelliJ Idea это можно сделать при помощи **"File → New → Module"**.

После создания нового Maven модуля нужно указать его как модуль в родительском **pom.xml**:
```xml
<modules>
  <module>cloud-eureka</module>
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

Теперь мы можем подключить зависимости для Eureka Server.\
Откроем Reference Doc для модуля **"[Spring Cloud Netflix](https://spring.io/projects/spring-cloud-netflix)"** и найдём раздел, который описывает **"[Service Discovery: Eureka Server](https://docs.spring.io/spring-cloud-netflix/docs/current/reference/html/#spring-cloud-eureka-server)"**.

В секции **"How to Include"** указано, какие стартеры нужно подключать.\
Таким образом, добавим блок **dependencies** в наш pom.xml:
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
    </dependency>
</dependencies>
```

## [↑](#home) <a id="configuration"></a> Eureka Server Configuration
Теперь нам нужно описать настройки Эврики в файле настроек в каталоге ``src/main/resources``.

С одной стороны, Eureka Server использует инфраструктуру Spring Boot, а следовательно может использовать **application.yml** файл для хранения своей конфигурации. Подробне см. **"[Spring Boot: External Application Properties](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config.files)"**.

С другой стороны, Spring Cloud вводит понятие **"[Bootstrap Application Context](https://docs.spring.io/spring-cloud/docs/current/reference/htmlsingle/#spring-cloud-context-application-context-services)"**, что позволяет настройки именно для Spring Cloud загружать раньше всего. Такие настройки должны быть в файле **bootstrap.yml**. Стоит понимать, что другие Spring проекты (такие как Spring Security) в общем случае базируются на Spring Boot, а следовательно умеют читать свою конфигурацию ТОЛЬКО из application.yml.

Добавим настройки Эврики в файл **src/main/resources/bootstrap.yml**:
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

## [↑](#home) <a id="instance"></a> Eureka Server Instance
Чтобы создать свой экземпляр Eureka Server нужно создать Spring Boot приложение.

Используя Spring Boot стоит избегать использования **"default package"**, о чём сказано в разделе документации **"[Using the “default” Package](https://docs.spring.io/spring-boot/docs/current/reference/html/using.html#using.structuring-your-code.using-the-default-package)"**. Поэтому для начала создадим свой пакет.

Далее создадим новый Java класс:
```java
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
Чтобы завершить настройку Эврики, нужно "задеплоить" сервер Эврики.

В целях обучения воспользуемся Heroku и за основу возьмём пример: **"[Managing your Microservices on Heroku with Netflix's Eureka](https://blog.heroku.com/managing_your_microservices_on_heroku_with_netflix_s_eureka)"**.

Ранее мы установили Heroku CLI. Откроем терминал (например, в IntelliJ Idea при помощи ATL+F12).\
Выполним **heroku --version**, чтобы убедиться, что Heroku CLI работает. Если нет - необходимо перепроверить, что переменная среды окружения PATH настроена верно и перезапустить терминал (т.к. переменные среды окружения считываются только один раз при запуске терминала).

Выполним вход на Heroku через Heroku CLI при помощи команды **heroku login**. В случае ошибки вида "[unable to get local issuer certificate](https://devcenter.heroku.com/articles/using-the-cli)" убедитесь, что не использует ПО вроде ZScaler, которое блокирует логин.

Для начала, нам необходимо создать новое приложение на стороне Heroku:
> heroku apps:create cloud-eureka --remote cloud-eureka

Каждое приложение на Heroku имеет свой собственный репозиторий. Каталогу с git репозиторием, из которого выполняется данная команда, добавляется remote, указанный командой --remote.

Далее необходимо указать переменную окружения, которая расскажет Heroku, какой файл нужно запускать. Именно для этого мы делали обезличенные Procfile:
> heroku config:set --app cloud-eureka JAR_PATH=cloud-eureka/target/cloud-eureka-1.jar

Теперь наши коммиты с исходным кодом в репозиторий Heroku: 
> git push --force cloud-eureka main

--------

## [↑](#home) <a id="security"></a> Security
Сервер эврики после старта отображает свою собственную страничку, на которой видно всю информацию по зарегистрированным сервисам. Необходимо скрыть это информацию от посторонних. Воспользуемся документацией по Spring Eureka Server: **"[Securing The Eureka Server](https://docs.spring.io/spring-cloud-netflix/docs/3.0.3/reference/html/#securing-the-eureka-server)"**.

Добавим в проект стартер для подключения **Spring Security**:
```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

Добавим конфигурационный файл application.yml и заполним его для Spring Security:
```yaml
spring:
  security:
    user:
      name: admin
      password: ${USER_PASSWORD}
```

В соответствии с документацией добавим новый класс **WebSecurityConfig**:
```java
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf().ignoringAntMatchers("/eureka/**");
        super.configure(http);
    }
}
```
Данный код отключает требование наличие валидного CSRF токена для страницы Eureka Server.

Добавим в Heroku для нашего приложения пароль: 
> heroku config:set --app cloud-eureka USER_PASSWORD=developer

Сделаем commit изменений и отправим в репозиторий Heroku: 
> git push --force cloud-eureka main

В случае необходимости, логи можно посмотреть так:
> heroku logs --tail