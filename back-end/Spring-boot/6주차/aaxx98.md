# TDD

### 테스트 주도 개발은 테스트 코드 작성하기 부터 시작하는 개발 방식이다.

기존의 개발 방식은 다음과 같은데, 이런 경우 코드 수정시 2~5를 매번 반복하게 된다.

1. 코드를 작성하고
2. 프로그램(Tomcat)을 실행한 뒤
3. API 테스트 도구로 HTTP를 요청하고
4. 요청 결과를 눈으로 검증한다(`System.out.print()`)
5. 결과가 다르면 다시 프로그램을 중)지하고 코드를 수정한다.

직접 수정된 기능을 확인하려면 톰캣을 내렸다가 다시 실행하는 일을 반복하게 된다.

테스트 코드를 작성하면 **자동 검증**이 가능하다.

또, 기능 단위로 테스트 코드를 작성하므로 문제가 발생한 기능이 무엇인지 구별하기 쉽다.

### 테스트 코드 작성을 도와주는 프레임워크

-   JUnit (Java)
-   DBUnit (DB)
-   CppUnit (C++)
-   NUnit (.net)

## 1. 테스트 코드 작성

### src/main/java/com/jojoldu/book/springboot/Application.java

```java
package com.jojoldu.book.springboot;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

-   `@SpringBootApplication`: 프로젝트 상단에 선언해야한다. (스프링부트 자동설정, 스프링 Bean 읽기, 생성을 자동 설정함)
-   `SpringApplication.run()`: 내장 WAS을 실행함, 내부에서 WAS를 실행하므로 서버에 톰캣 설치할 필요가 없다.

    -   언제 어디서나 같은 환경에서 스프링 부트를 배포할 수 있기 때문에 스프링 부트에서는 내장 WAS를 권장한다.

### src/main/java/com/jojoldu/book/springboot/web/HelloController.java

```java
package com.jojoldu.book.springboot.web;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {
    @GetMapping("/hello")
    public String hello() {
        return "hello";
    }
}
```

-   `@RestController`: 컨트롤러를 JSON을 반환하는 컨트롤러로 만들어준다.
-   `@GetMapping`: HTTP Method인 Get 요청을 받을 수 있는 API를 만들어준다.
    -   `@GetMapping("/hello")`: `/hello`로 요청이 오면, `hello()` 실행 결과를 반환한다.

### src/test/java/com/jojoldu/book/springboot/web/HelloControllerTest.java

```java
package com.jojoldu.book.springboot.web;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.ResultActions;

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@RunWith(SpringRunner.class)
@WebMvcTest(controllers = HelloController.class)
public class HelloControllerTest {
    @Autowired
    private MockMvc mvc;

    @Test
    public void hello가_리턴된다() throws Exception {
        String hello = "hello";

        mvc.perform(get("/hello"))
                .andExpect(status().isOk())
                .andExpect(content().string(hello));
    }
}

```

-   `@RunWith(SpringRunner.class)`

    -   테스트 진생시 JUnit에 내장된 실행자 외에 다른 실행자를 실행시킨다.
    -   여기서 SpringRunner라는 스프링 실행자를 사용한다.
    -   스프링 부트 테스트와 JUnit 사이에 연결자 역할을 한다.

-   `@WebMvcTest`
    -   선언할 경우 `@Controller`, `@ControllerAdvice` 등을 사용할 수 있다.
    -   단, `@Service`,`@Component`,`@Repository` 등은 사용할 수 없다.
-   `@Authowired`
    -   스프링이 관리하는 Bean을 주입 받는다.
-   `private MockMvc mvc`
    -   웹 API를 테스트할 때 사용한다.
    -   스프링 MVC 테스트의 시작점이다.
    -   이 클래스를 통해 HTTP GET, HTTP POST 등에 대한 API 테스트를 할 수 있다.
-   `mvc.perform(get("/hello"))`
    -   MockMvc를 통해 /hello 주소로 HTTP GET 요청을 한다.
    -   체이닝이 지원된다. (여러 검증 기능을 이어서 선언 가능)
-   `.andExpect(status().isOk())`

    -   mvc.perform의 결과를 검증한다.
    -   HTTP Header의 Status를 검증한다.(**200 OK**인지 확인한다.)

-   `.andExpect(content().string(hello))`

    -   mvc.perform의 결과를 검증한다.
    -   응답 본문 내용을 검증한다.
    -   Controller에서 `"hello"`를 리턴하기 때문에 이 값이 맞는지 검증한다.

    ![테스트결과](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/b4de3b21-1e1a-446e-8902-9e403f5763c2/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210402%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210402T204642Z&X-Amz-Expires=86400&X-Amz-Signature=e754b4b673ed2578670a68f218b0c7c477a76934808c3d942e1151ab538b1ca8&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22)

## 2. HelloController 코드를 롬복(Lombok)으로 전환하기

롬복은 Getter, Setter, 기본 생성자, toString을 **어노테이션**으로 자동 생성시키는 라이브러리이다.

### src/main/java/com/jojoldu/book/springboot/web/dto/HelloResponseDto.java

```java
package com.jojoldu.book.springboot.web.dto;

import lombok.Getter;
import lombok.RequiredArgsConstructor;

@Getter
@RequiredArgsConstructor
public class HelloResponseDto {
    private final String name;
    private final int amount;
}
```

-   `@Getter`: 선언된 모든 필드의 get 메소드를 생성해준다.
    -   `getName()`, `getAmount()`가 생성된다.
-   `@RequiredArgsConstructor`: 선언된 모든 final 필드가 포함된 생성자를 생성해준다. final이 없는 필드는 생성자에 포함되지 않는다.

    ```java
    public HelloResponseDto(String name, int amount){
    this.name=name;
    this.amount=amount;
    }
    ```

    클래스 내에 위와 같은 생성자가 존재하는것과 같다.

### src/test/java/com/jojoldu/book/springboot/web/dto/HelloResponseDtoTest.java

```java
package com.jojoldu.book.springboot.web.dto;

import org.junit.Test;

import static org.assertj.core.api.Assertions.assertThat;

public class HelloResponseDtoTest {

    @Test
    public void 롬복_기능_테스트(){
        //given
        String name = "test";
        int amount = 1000;

        //when
        HelloResponseDto dto = new HelloResponseDto(name, amount);

        //then
        assertThat(dto.getName()).isEqualTo(name);
        assertThat(dto.getAmount()).isEqualTo(amount);
    }
}
```

-   `assertThat`
    -   assertj 라는 테스트 검증 라이브러리의 검증 메소드
    -   검증하고 싶은 대상을 메소드 인자로 받는다.
    -   메소드 체이닝이 지원된다.
-   `isEqualTo`
    -   assertj의 동등 비교 메소드
    -   assertThat에 있는 값과 isEqualTo의 값을 비교하여 같을 때만 테스트 통과

> 테스트 실행중 오류가 발생하면 [gradle 4버전으로 다운그레이드 해보자...](https://github.com/jojoldu/freelec-springboot2-webservice/issues/2)

### src/main/java/com/jojoldu/book/springboot/web/HelloController.java

```java
package com.jojoldu.book.springboot.web;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {
    @GetMapping("/hello")
    public String hello() {
        return "hello";
    }

    @GetMapping("/hello/dto")
    public HelloResponseDto helloDto(@RequestParam("name") String name,
                                     @RequestParam("amount") int amount) {
        return new HelloResponseDto(name, amount);
    }
}
```

-   `@RequestParam`: 외부에서 API로 넘긴 파라미터를 가져온다.
    -   `@RequestParam("name")`: 외부에서 name이란 이름으로 넘긴 파라미터를
    -   `String name`: String name에 저장한다.

### src/test/java/com/jojoldu/book/springboot/web/HelloControllerTest.java

```java
package com.jojoldu.book.springboot.web;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;

import static org.hamcrest.Matchers.is;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@RunWith(SpringRunner.class)
@WebMvcTest
public class HelloControllerTest {
    @Autowired
    private MockMvc mvc;

    @Test
    public void hello가_리턴된다() throws Exception {
        String hello = "hello";

        mvc.perform(get("/hello"))
                .andExpect(status().isOk())
                .andExpect(content().string(hello));
    }

    @Test
    public void helloDto가_리턴된다() throws Exception {
        String name = "hello";
        int amount = 1000;

        mvc.perform(
                get("/hello/dto")
                        .param("name", name)
                        .param("amount", String.valueOf(amount)))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.name", is(name)))
                .andExpect(jsonPath("$.amount", is(amount)));
    }
}

```

-   param

    -   API 테스트 시 사용될 요청 파라미터 설정
    -   값은 String만 허용한다.

-   jsonPath
    -   JSON 응답값을 필드별로 검증할 수 있는 메소드
    -   $를 기준으로 필드명을 명시한다.

### Application.java를 실행한 결과는 다음과 같다.

![결과1](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/5baccf57-e7a2-4ee8-bf71-77ed1f27c3de/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210402%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210402T231825Z&X-Amz-Expires=86400&X-Amz-Signature=3d18e39501efaf910fe4a5abf7c54286fe8192a0c2ed5a80fbaef734934d8695&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22)

![결과2](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/bbf1ce60-9f4d-4616-9136-6c661af6d648/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210402%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210402T231605Z&X-Amz-Expires=86400&X-Amz-Signature=886bfc20ba4276798fa4ac38f7da1d0832da7151837494974188973f47141d53&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22)
