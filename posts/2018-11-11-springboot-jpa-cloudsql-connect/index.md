# appengine + springboot + kotlin + jpa + cloudsql 연동하기


회사에서 새로 시작하는 API 서버 개발에 아래와 같은 환경을 적용하기로 했다.

* 구동환경 : [Google App Engine Standard Environment](https://cloud.google.com/appengine/docs/standard/java/)
* 언어 : [kotlin 1.3](https://kotlinlang.org/)
* framework : [Spring Boot 2.0.2.RELEASE](https://spring.io/projects/spring-boot)
* DB : [cloud sql(MySQL 5.7)](https://cloud.google.com/sql/docs/mysql/)
* ORM : [hibernate/JPA](https://opentutorials.org/module/1281)
* IDE: [IntelliJ Ultimate](https://www.jetbrains.com/idea/)

각각 놓고보면 다들 유명하고 널리 쓰이는 기술들이지만 얘들을 한꺼번에 적용한 가이드라인이 없어서 내가 직접 가이드를 작성해볼까 한다. A to Z 형식의 가이드는 아니고 셋팅 중 삽질을 크게 했던 부분 위주로 작성해보도록 하겠다.

## build.gradle 작성

의존성 관리를 위한 build.gradle 파일이다. 아래의 코드 내용은 [spring initializer](https://start.spring.io/)를 통해서 생성된 프로젝트를 바탕으로 이것저것 수정한 버전이다.

``` gradle

buildscript {
	ext {
		kotlinVersion = '1.3.0'
		springBootVersion = '2.0.2.RELEASE'
	}
	repositories {
		jcenter()
		mavenCentral()
	}
	dependencies {
		classpath 'com.google.cloud.tools:appengine-gradle-plugin:1.+'    // latest App Engine Gradle tasks
		classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
		classpath("org.jetbrains.kotlin:kotlin-gradle-plugin:${kotlinVersion}")
		classpath "org.jetbrains.kotlin:kotlin-allopen:$kotlinVersion"
		classpath 'com.google.gms:google-services:4.1.0'
	}
}

plugins {
	id "org.jetbrains.kotlin.plugin.spring" version '1.3.0'
	id "org.jetbrains.kotlin.plugin.noarg" version '1.3.0'
	id "org.jetbrains.kotlin.plugin.allopen" version '1.3.0'
}

apply plugin: 'java'
apply plugin: 'kotlin'
apply plugin: 'org.springframework.boot'
apply plugin: 'org.jetbrains.kotlin.plugin.spring'
apply plugin: 'kotlin-jpa'
apply plugin: 'kotlin-noarg'

apply plugin: 'kotlin-kapt'
apply plugin: 'idea'

apply plugin: "kotlin-allopen"

apply plugin: 'war'                               // standard Web Archive plugin
apply plugin: 'com.google.cloud.tools.appengine'  // App Engine tasks

noArg {
	annotation("javax.persistence.Entity")
}

allOpen {
	annotation("javax.persistence.Entity")
}

group = 'com.companyname'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = 1.8
compileKotlin {
	kotlinOptions {
		freeCompilerArgs = ["-Xjsr305=strict"]
		jvmTarget = "1.8"
	}
}
compileTestKotlin {
	kotlinOptions {
		freeCompilerArgs = ["-Xjsr305=strict"]
		jvmTarget = "1.8"
	}
}

repositories {
	mavenCentral()
}

dependencies {
	implementation('com.google.appengine:appengine:+')
	implementation('com.google.appengine:appengine-api-1.0-sdk:+')
	implementation('com.google.appengine.tools:appengine-gcs-client:0.8')

	// kotlin
	compile "org.jetbrains.kotlin:kotlin-stdlib-jdk8:$kotlinVersion"
	compile "org.jetbrains.kotlin:kotlin-reflect:$kotlinVersion"

	// spring boot
	compile "org.springframework.boot:spring-boot-starter-web:${springBootVersion}"
	compile "org.springframework.boot:spring-boot-starter-data-jpa:${springBootVersion}"
	compile "org.springframework.boot:spring-boot-starter-jdbc:${springBootVersion}"

	// jackson
	compile "com.fasterxml.jackson.datatype:jackson-datatype-jdk8"
	compile "com.fasterxml.jackson.datatype:jackson-datatype-jsr310"
	compile "com.fasterxml.jackson.datatype:jackson-datatype-hibernate5:2.+"
	compile "com.fasterxml.jackson.module:jackson-module-kotlin:2.+"

	// db: hibernate
	compile "org.hibernate:hibernate-core:5.2.7.Final"
	compile "org.hibernate:hibernate-entitymanager:5.2.7.Final"
	compile "org.hibernate:hibernate-java8:5.2.7.Final"
	compile group: 'org.qlrm', name: 'qlrm', version: '2.0.2'

	//db : mysql
	implementation('mysql:mysql-connector-java:5.+')
	implementation('com.google.cloud.sql:mysql-socket-factory-connector-j-8:1.0.11')
	implementation('com.google.cloud.sql:mysql-socket-factory:1.0.11')
	
	compile 'javax.xml.bind:jaxb-api:2.3.0'

	testImplementation('org.springframework.boot:spring-boot-starter-test:2.0.2.RELEASE')
	testImplementation 'com.google.appengine:appengine-testing:1.+'
	testImplementation 'com.google.appengine:appengine-api-stubs:1.+'
	testImplementation 'com.google.appengine:appengine-tools-sdk:1.+'
}

appengine {  // App Engine tasks configuration
    deploy {   // deploy configuration

    }
}

```

## appengine-web.xml 파일 작성

보통 스프링의 각종 프로퍼티들은 `resources/application.properties`에 작성하는 경우가 많다(요즘에는 `yml` 파일로 많이들 작성한다고 카더라). 하지만 우리는 App Engine 환경에서 서버를 실행할 것이기 때문에 `webapp/WEB-INF/appengine-web.xml`에 각종 프로퍼티들을 선언해 줄 것이다.

``` xml

<appengine-web-app xmlns="http://appengine.google.com/ns/1.0">
    <version>1</version>
    <threadsafe>true</threadsafe>
    <runtime>java8</runtime>
    <env>standard</env>
    <use-google-connector-j>true</use-google-connector-j>

    <system-properties>
        <property name="spring.datasource.continue-on-error" value="true"/>
        <property name="spring.datasource.initialization-mode" value="always"/>

        <property name="spring.jackson.serialization.fail-on-empty-beans" value="false"/>
        <property name="spring.jpa.hibernate.ddl-auto" value="update"/>
        <property name="spring.jpa.properties.hibernate.hbm2ddl.auto" value="update"/>
        <property name="spring.jpa.properties.hibernate.dialect" value="org.hibernate.dialect.MySQL57Dialect"/>
        <property name="spring.jpa.properties.hibernate.id.new_generator_mappings" value="true"/>
        <property name="spring.jpa.properties.hibernate.format_sql" value="true"/>

        <property name="spring.datasource.url" value="jdbc:mysql://google/<DATABASE_NAME>?cloudSqlInstance=<INSTANCE_CONNECTION_NAME>&socketFactory=com.google.cloud.sql.mysql.SocketFactory&user=<MYSQL_USER_NAME>&password=<MYSQL_USER_PASSWORD>&useSSL=false"/>
    </system-properties>
</appengine-web-app>


```

위 내용들을 `application.properties`에 형식을 맞춰서 작성해도 동작이 되긴 하는듯 하다. 하지만 구글의 앱엔진 가이드에는 `appengine-web.xml`에 작성하는걸 기준으로 설명하고 있으니 일단은 위와 같이 작성해두도록 하겠다.



위 내용에서 내가 특히 헤맸던 설정이 있다. 

``` xml
<property name="spring.jpa.properties.hibernate.dialect" value="org.hibernate.dialect.MySQL57Dialect"/>
```

이 옵션인데 현재 내가 최초에 적용했던 값은 `org.hibernate.dialect.MySQL5Dialect`였다(`7`이 빠졌다). 하이버네이트를 MySQL5 버전에 연결하기 위한 옵션값인데 _로컬서버에 설치된 MYSQL5.7에 연결했을때는 아무 문제도 없었다._ 근데 앱엔진에 배포하고 테스트를 해보니 하이버네이트가 DB에 연결을 못하고 있었다. 그래서 열심히 구글링을 해본 결과 저 dialect 값이 문제였다. 이유는 모르겠다. 아마 앱엔진에서 CloudSQL에 접근할때 사용되는 프록시와 연관이 있을지도..? 아무튼 **`MySQL57Dialect`라고 정확히 명시를 해줘야 앱엔진 환경에서도 DB에 문제없이 연결이 된다.**


## main class 작성하기

``` kotlin

@SpringBootApplication
class Application : SpringBootServletInitializer(){
    @Bean
    fun hibernate5Module(): Module { //for jpa lazy loading
        return Hibernate5Module()
    }
}

fun main(args: Array<String>) {
    SpringApplication.run(Application::class.java, *args)
}

```

여기까지 작성한다면 API 개발을 위한 기본적인 셋팅이 완료된 것이다. 이제 `@Controller`, `@Service` 등등을 작성하면서 API를 완성해나가면 된다.

참고 : [Kotlin에서 JPA 사용할 때 주의할 점](https://blog.sapzil.org/2017/11/02/kotlin-jpa-pitfalls/)

