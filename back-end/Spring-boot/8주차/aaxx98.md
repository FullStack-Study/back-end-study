# 4. 머스테치로 화면 구성하기

## 4.1 서버 템플릿 엔진

-   웹 개발에서 템플릿 엔진

    -   지정된 템플릿 양식과 데이터가 함쳐져 HTML문서를 출력하는 소프트웨어
    -   JSP, Freemarker (서버 템플릿 엔진)
    -   React, Vue (클라이언트 템플릿 엔진)

**JSP 코드**
다음 코드는 if문과 관계 없이 무조건 콘솔에 test를 출력한다.

```java
<script type="text/javascript">

$(document).ready(function(){
    if(a=="1"){
    <%
        System.out.println("test");
    %>
    }
});
```

-   JSP는 서버 템플릿 엔진이고, 서버 템플릿 엔진은 서버에서 구동된다.
-   서버 템플릿 엔진에서 화면 생성을 할 때, 서버에서 Java 코드로 문자열을 만들어 HTML로 변환하여 브라우저로 전달한다.
-   위 코드는 HTML을 만드는 과정에서 자바스크립트 코드와는 별개로 `System.out.println("test");`가 실행된다.
-   자바스크립트 코드는 브라우저 위에서 작동한다.

## 4.2 머스테치

머스테치는 수많은 언어를 지원하는 가장 심플한 템플릿 엔진이다.

-   장점
    -   문법이 다른 템플릿 엔진보다 심플하다.
    -   로직 코드를 사용할 수 없어 View의 역할과 서버의 역할을 명확하게 분리한다.
    -   Nustache.js와 Mustache.java 두가지가 모두 있기 때문에 하나의 문법으로 클라이언트/서버 템플릿을 사용할 수 있다.

**index.mustache**

```html
<!DOCTYPE html>
<html>
    <head>
        <title>스프링 부트 웹서비스</title>
        <meta http-equiv="Content-Type" content="text/html;charset=UTF-8" />
    </head>
    <body>
        <h1>스프링 부트로 시작하는 웹 서비스</h1>
    </body>
</html>
```

**IndexController.java**

```java
package com.jojoldu.book.springboot.web;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class IndexController {
    @GetMapping("/")
    public String index(){
        return "index";
    }
}
```

-   머스테치 스타터 덕분에 컨트롤러가 반환하는 문자열 `index`는 `src/main/resources/templates/index.mustache`로 변환되어 View Resolver가 처리하게된다.

## 4.3 등록/조회/수정/삭제 기능

**src/main/java/.../web/IndexController**

```java
package com.jojoldu.book.springboot.web;

import com.jojoldu.book.springboot.service.PostsService;
import com.jojoldu.book.springboot.web.dto.PostsResponseDto;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;

@RequiredArgsConstructor
@Controller
public class IndexController {

    private final PostsService postsService;

    @GetMapping("/")
    public String index(Model model){
        model.addAttribute("posts", postsService.findAllDesc());
        return "index";
    }

    @GetMapping("/posts/save")
    public String postsSave(){
        return "posts-save";
    }

    @GetMapping("/posts/update/{id}")
    public String postsUpdate(@PathVariable Long id, Model model){
        PostsResponseDto dto = postsService.findById(id);
        model.addAttribute("post",dto);

        return "posts-update";
    }
}
```

-   `@GetMapping(path)`: path로 접근하면 머스태치 파일을 호출한다.
-   `model.addAttribute("posts", postsService.findAllDesc());`
    -   `postsService.findAllDesc()`로 가져온 내용을`posts`로 머스태치를 호출하면서 같이 전달한다.

**src/main/java/resources/index.mustache**

```html
{{>layout/header}}
<h1>스프링 부트로 시작하는 웹 서비스 Ver.2</h1>
<div class="col-md-12">
    <div class="row">
        <div class="col-md-6">
            <a href="/posts/save" role="button" class="btn btn-primary"
                >글 등록</a
            >
        </div>
    </div>
    <br />
    <!-- 목록 출력 영역 -->
    <table class="table table-horizontal table-bordered">
        <thead class="thead-strong">
            <tr>
                <th>게시글 번호</th>
                <th>제목</th>
                <th>작성자</th>
                <th>최종 수정일</th>
            </tr>
        </thead>
        <tbody id="tbody">
            {{#posts}}
            <tr>
                <td>{{id}}</td>
                <td><a href="/posts/update/{{id}}">{{title}}</a></td>
                <td>{{author}}</td>
                <td>{{modifiedDate}}</td>
            </tr>
            {{/posts}}
        </tbody>
    </table>
</div>
{{>layout/footer}}
```

-   `for each`문 처럼 `{{#posts}}`~`{{/posts}}` 사이의 내용을 전달받은 `posts` 요소 개수에 따라 반복한다.

**src/main/java/resources/posts-save.mustache**

```html
{{>layout/header}}
<h1>게시글 등록</h1>
<div class="col-md-12">
    <div class="col-md-4">
        <form>
            <div class="form-group">
                <label for="title">제목</label>
                <input
                    type="text"
                    class="form-control"
                    id="title"
                    placeholder="제목을 입력하세요"
                />
            </div>
            <div class="form-group">
                <label for="author">작성자</label>
                <input
                    type="text"
                    class="form-control"
                    id="author"
                    placeholder="작성자를 입력하세요"
                />
            </div>
            <div class="form-group">
                <label for="content">내용</label>
                <textarea
                    class="form-control"
                    id="content"
                    placeholder="내용을 입력하세요"
                ></textarea>
            </div>
        </form>
        <a href="/" role="button" class="btn btn-secondary">취소</a>
        <button type="button" class="btn btn-primary" id="btn-save">
            등록
        </button>
    </div>
</div>
{{>layout/footer}}
```

**src/main/java/resources/posts-update.mustache**

```html
{{>layout/header}}
<h1>게시글 수정</h1>
<div class="col-md-12">
    <div class="col-md-4">
        <form>
            <div class="form-group">
                <label for="id">글 번호</label>
                <input
                    type="text"
                    class="form-control"
                    id="id"
                    value="{{post.id}}"
                    readonly
                />
            </div>
            <div class="form-group">
                <label for="title">제목</label>
                <input
                    type="text"
                    class="form-control"
                    id="title"
                    value="{{post.title}}"
                />
            </div>
            <div class="form-group">
                <label for="author">작성자</label>
                <input
                    type="text"
                    class="form-control"
                    id="author"
                    value="{{post.author}}"
                    readonly
                />
            </div>
            <div class="form-group">
                <label for="content">내용</label>
                <textarea class="form-control" id="content">
{{post.content}}</textarea
                >
            </div>
        </form>
        <a href="/" role="button" class="btn btn-secondary">취소</a>
        <button type="button" class="btn btn-primary" id="btn-update">
            수정 완료
        </button>
        <button type="button" class="btn btn-danger" id="btn-delete">
            삭제
        </button>
    </div>
</div>
{{>layout/footer}}
```

**src/resources/static/js/app/index.js**

```js
var main = {
    init: function () {
        var _this = this;
        $("#btn-save").on("click", function () {
            _this.save();
        });

        $("#btn-update").on("click", function () {
            _this.update();
        });

        $("#btn-delete").on("click", function () {
            _this.delete();
        });
    },
    save: function () {
        var data = {
            title: $("#title").val(),
            author: $("#author").val(),
            content: $("#content").val(),
        };
        $.ajax({
            type: "POST",
            url: "/api/v1/posts",
            dataType: "json",
            contentType: "application/json; charset-utf-8",
            data: JSON.stringify(data),
        })
            .done(function () {
                alert("글이 등록되었습니다.");
                window.location.href = "/";
            })
            .fail(function (error) {
                alert(JSON.stringify(error));
            });
    },
    update: function () {
        var data = {
            title: $("#title").val(),
            content: $("#content").val(),
        };

        var id = $("#id").val();

        $.ajax({
            type: "PUT",
            url: "/api/v1/posts/" + id,
            dataType: "json",
            contentType: "application/json; charset-utf-8",
            data: JSON.stringify(data),
        })
            .done(function () {
                alert("글이 수정되었습니다.");
                window.location.href = "/";
            })
            .fail(function (error) {
                alert(JSON.stringify(error));
            });
    },
    delete: function () {
        var id = $("#id").val();

        $.ajax({
            type: "DELETE",
            url: "/api/v1/posts/" + id,
            dataType: "json",
            contentType: "application/json; charset-utf-8",
        })
            .done(function () {
                alert("글이 삭제되었습니다.");
                window.location.href = "/";
            })
            .fail(function (error) {
                alert(JSON.stringify(error));
            });
    },
};

main.init();
```

-   각 페이지의 등록/수정/삭제 버튼에 기능을 연결한다.
-   `main` 객체를 만들어 내부에 함수를 선언하여 다른 JS와 스코프가 겹치지 않도록 함수를 관리한다.

**src/main/java/.../web/PostsApiController**

```java
package com.jojoldu.book.springboot.web;

import com.jojoldu.book.springboot.domain.posts.PostsRepository;
import com.jojoldu.book.springboot.service.PostsService;
import com.jojoldu.book.springboot.web.dto.PostsResponseDto;
import com.jojoldu.book.springboot.web.dto.PostsSaveRequestDto;
import com.jojoldu.book.springboot.web.dto.PostsUpdateRequestDto;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.*;

@RequiredArgsConstructor
@RestController
public class PostsApiController {
    private final PostsService postsService;

    @PostMapping("/api/v1/posts")
    public Long save(@RequestBody PostsSaveRequestDto requestDto){
        return postsService.save(requestDto);
    }

    @PutMapping("/api/v1/posts/{id}")
    public Long update(@PathVariable Long id, @RequestBody PostsUpdateRequestDto requestDto) {
        return postsService.update(id, requestDto);
    }

    @GetMapping("/api/v1/posts/{id}")
    public PostsResponseDto findById (@PathVariable Long id){
        return postsService.findById(id);
    }

    @DeleteMapping("/api/v1/posts/{id}")
    public Long delete(@PathVariable Long id){
        postsService.delete(id);
        return id;
    }
}
```

-   버튼에 매핑된 기능을 구현한다.
