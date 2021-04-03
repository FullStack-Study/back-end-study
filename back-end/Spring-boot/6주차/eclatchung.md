# 스프링 부트에서 테스트 코드를 작성하자



## 단위 테스트와 TDD
- `TDD` : 테스트가 주도하는 개발, 테스트 코드를 먼저 작성
- `단위 테스트` : 기능 단위의 테스트 코드를 짜는것
- `TDD` ≠ `단위테스트`

### 단위 테스트의 이점
1. 빠른 피드백
2. 자동 검증
3. 기능 보호
#### 테스트 코드 ⇒ 기존 기능이 잘 작동되는 것을 보장
### `JUnit `
- 단위 테스트 도구
- javascript 는 `Mocha`, `Karama`, `Jest` 등 이있음

## 롬복 
- `Getter`, `Setter`등을 자동 생성해둠.

## HelloControllerTest.java
```java
package com.eclat.study.web;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;
import static org.hamcrest.Matchers.is;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.jsonPath;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@RunWith(SpringRunner.class)
@WebMvcTest()
public class HelloControllerTest {
    @Autowired
    private MockMvc mvc;

    @Test
    public void hello가_리턴된다() throws Exception{
        String hello = "hello";

        mvc.perform(get("/hello"))
                .andExpect(status().isOk())
                .andExpect(content().string(hello));
    }

    @Test
    public void helloDtod가_리턴된다() throws Exception {
        String name = "hello";
        int amount = 1000;

        mvc.perform(
                get("/hello/dto")
                    .param("name",name)
                    .param("amount", String.valueOf(amount))) // param은 String 값만 허
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.name",is(name)))
                .andExpect(jsonPath("$.amount",is(amount)));


    }
}

```

### param()
- `string`값만 허용 ⇒ 숫자나 날짜는 string으로 변환 필요
- `param("amount", String.valueOf(amount)))`


## 나온 에러
### 프로젝트 생성시 에러 (gradle에러)
```shell
Caused by: org.codehaus.groovy.control.MultipleCompilationErrorsException: startup failed:
```
- 예제는 JDK 11인데 나는 JDK 16을 사용하였음
[프로젝트 생성 에러 메시지 - 인프런 | 질문 & 답변](https://www.inflearn.com/questions/77532)


### package 미일치 에러 ( 오타 )

``` shell
java.lang.IllegalStateException: Unable to find a @SpringBootConfiguration, you need to use @ContextConfiguration or @SpringBootTest(classes=...) with your test
```
- 패키지 일치 하지 않아 생긴 에러 
`com.elcat.study`으로 컨트롤러가 설정되어있었고 테스트가 `com.eclat.study`로 되어있어서 패키지 일치하지 않아서 찾을 수 가 없어서 에러 생김

[java.lang.IllegalStateException: Unable to find a @SpringBootConfiguration, you need to use @ContextConfiguration or @SpringBootTest(classes=…) with your test :: 박철우의 블로그](https://parkcheolu.tistory.com/125)