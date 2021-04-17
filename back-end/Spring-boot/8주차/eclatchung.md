# 머스테치로 화면 구성하기


## 템플릿 엔진
지정된 템플릿 양식과 데이터가 합쳐져 HTML문서를 출력 
1. 서버 템플릿 엔진
	- JSP, Freemarker
	- 서버에서 Java코드로 문자열을 만들고 HTML로 변환하여 브라우저에 전달
	- 서버 위에서 작동
2. 클라이언트 템플릿 엔진
	- React, vue
	- 브라우저 위에서 작동 
	- SPA에 적합
### IndexController.java

```JAVA
public class IndexController {

    private final PostsService postsService;

    @GetMapping("/")
    public String index(Model model) {
        model.addAttribute("posts", postsService.findAllDesc());
        return "index";
    }

    @GetMapping("/posts/save")
    public String postsSave(){
        return "posts-save";
    }

    @GetMapping("/posts/update/{id}")
    public String postsUpdate(@PathVariable Long id, Model model) {
        PostsResponseDto dto = postsService.findById(id);
        model.addAttribute("post", dto);

        return "posts-update";
    }

}

```

- 해당 클래스를 통해서 해당 패스에 알맞은 HTML문서를 전달해줌.


## 에러 
1. **[ERROR] SpringBoot - net::ERR_ABORTED 404**
- 소스 파일을 찾을 수가 없었다.
- 폴더 생성을 바로 `/static/js/app`으로 설정을 햇는데 -> 각각 하나씩 생성하고 실행하니깐 되었다.

2. **최종수정일 등록이 안되는 에러**
- Application.java에서 `@EnableJpaAuditing`을 입력안했었다.






