# IntelliJ

# 1. 설치

[toolbox 설치](https://www.jetbrains.com/toolbox-app/)

1. toolbox를 설치
2. IntelliJ IDEA Community (무료버전) 설치
3. Setting에서 JVM 옵션을 설정할 수 있음
    - 메모리가 8G라서 Maximum Heap Size를 2048MB로 설정했다.
4. 설치 후 설정할 것은 딱히 없었음

# 2. 프로젝트 생성

메인 화면의 new project를 클릭하여 프로젝트를 생성할 수 있다.

-   Gradle 프로젝트를 선택
-   Project SDK를 설정해야하는데 java를 설치하지 않아서 설정창에서 1.8 버전으로 다운로드함
    -   사용자>.jdks 디렉토리에 설치됨
-   GroupId, ArtifactId를 설정해야하는데 그냥 프로젝트명과 경로를 선택하도록 되어있었음
    -   프로젝트명만 설정하고 defalt로 진행했다.
    -   사용자>IdeaProjects 디렉토리에 프로젝트 파일이 생성된다.
-   [Eclipse의 Workspace와 IntelliJ의 Project](https://jojoldu.tistory.com/334#ref=facebook)

# 3. 그레이들 프로젝트를 스프링 부트 프로젝트로 변경하기

프로젝트 내의 build.gradle 파일을 다음과 같이 수정

```bash
buildscript {
    ext{
        springBootVersion = '2.1.7.RELEASE'
    }
    repositories{
        mavenCentral()
        jcenter()
    }
    dependencies{
        classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
    }
}

apply plugin: 'java'
apply plugin: 'eclipse'
apply plugin: 'org.springframework.boot'
apply plugin: 'io.spring.dependency-management'

group 'org.example'
version '1.0-SNAPSHOT'
sourceCompatibility = 1.8

repositories {
    mavenCentral()
    jcenter()
}

dependencies {
    compile('org.springframework.boot:spring-boot-starter-web')
    testCompile('org.springframework.boot:spring-boot-starter-test')
}
```

-   buildscript

    -   dependencies : 이 프로젝트의 플러그인 의존성 관리를 위한 설정
    -   ext : build.gradle에서 사용하는 전역변수 설정
        -   springBootVersion 전역변수를 생성하고 그 값을 '2.1.7.RELEASE'로 하겠다는 의미

-   apply plugin

    자바와 스프링부트를 사용하기 위한 필수 플러그인

    -   'java'-
    -   'eclipse'
    -   'org.springframework.boot'
    -   'io.spring.dependency-management'
        -   스프링부트의 의존성을 관리해주는 플러그인

-   repositories

    각 의존성(라이브러리)들을 어떤 원격 저장소에서 받을지 결정함

    -   mavenCentral()
        -   기본 라이브러리 저장소, 업로드가 어렵기 때문에 잘 사용하지 않는 문제가 발생
    -   jcenter()
        -   라이브러리 업로드가 간단한 저장소, mavenCentral에도 업로드할 수 있도록 자동화 할 수도 있음

-   dependencies

    프로젝트 개발에 필요한 의존성 선언

# 4. 인텔리제이에서 깃과 깃허브 사용하기

-   .gitignore

```bash
.gradle
.idea
```

-   commit: `ctrl+K`
-   push: `ctrl+shift+K`
