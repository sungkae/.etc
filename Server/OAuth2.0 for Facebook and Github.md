# **내 웹서버에서 Facebook과 GitHub 로그인으로 로그인하기**

https://spring.io/guides/tutorials/spring-boot-oauth2/ 에 기반하여 작성한다.
  
이 샘플은 8080 포트를 사용하며 jav -jar target/*.jar로 서버를 build할 수 있다.  
` $ java -jar target/*.jar`
- - -
## **Single Sign on With Facebook (페이스북 통합 인증)**
**통합인증**은 한 번의 인증 과정으로 여러 컴퓨터 상의 자원을 이용 가능하게 하는 인증 기능이다. **싱글 사인온**, **단일 계정 로그인, 단일 인증** 이라고 한다. (*wikipedia*)  

내가 핸드폰으로 카카오톡을 로그인 한 후에, 카카오톡으로 로그인할 수 있는 다른 앱들은 비밀번호를 입력하지 않아도 통합인증을 도입한 환경으로 알아서 로그인할 수 있다. Google, Facebook, Github, GameCenter, Twitter, etc. ...많다.

### **새 프로젝트 생성**
 먼저 새 프로젝트를 생성하자!!  
 <div>
 > ![image](https://user-images.githubusercontent.com/38012662/51439003-92fc0300-1cf7-11e9-8cce-1d907b383134.png)  
 Intellij에서 새 프로젝트를 생성하면 Spring Initializer로 생성할 수 있다.

> ![image](https://user-images.githubusercontent.com/38012662/51439042-ff770200-1cf7-11e9-96c6-cbf2ef8e1e0e.png)  
그 이후에 간단히 Dependency를 추가한다. Security는 추가해야할 것 같아서.. Lombok과 Web은 일단 잘 써서 추가!

>![image](https://user-images.githubusercontent.com/38012662/51439060-41a04380-1cf8-11e9-9262-4b8f7a10d2b0.png)  
Project name은 적절하게.. 난 Git에 올릴 것이기 때문에 gitrepos라는 폴더에 생성한다. Finish!

>![image](https://user-images.githubusercontent.com/38012662/51439076-77ddc300-1cf8-11e9-97e2-b80dc240306a.png)  
프로젝트가 시작하면서 이 화면이 출력되는데, 그냥 OK하십셔 별거없습니다~ 하지만 Gradle JVM의 JDK가 내가 기억하는 JDK 장소와 맞는지정도는 확인하시길.. 8버전을 추천함.

>![image](https://user-images.githubusercontent.com/38012662/51439104-c1c6a900-1cf8-11e9-938b-3b0debfc8353.png)  
생성 끝.
</div>

### **홈페이지 생성**
이제 내가 앞을 볼 수 있는 웹페이지를 생성할 것이다.  

<summary>index.html </summary>

```html
<!doctype html>
<html lang="en">
<head>
    <meta charset="utf-8"/>
    <meta http-equiv="X-UA-Compatible" content="IE=edge"/>
    <title>Demo</title>
    <meta name="description" content=""/>
    <meta name="viewport" content="width=device-width"/>
    <base href="/"/>
    <link rel="stylesheet" type="text/css" href="/webjars/bootstrap/css/bootstrap.min.css"/>
    <script type="text/javascript" src="/webjars/jquery/jquery.min.js"></script>
    <script type="text/javascript" src="/webjars/bootstrap/js/bootstrap.min.js"></script>
</head>
<body>
	<h1>Demo</h1>
	<div class="container"></div>
</body>
</html>
```
다른건 넘긴다고 쳐도 head쪽에 bootstrap.min.css, jquery.min.js를 기억하자.  
  
별건 아니지만 bootstrap에서 제공하는 어여쁜 디자인들을 사용하는 코드와, 클라이언트와 서버간에 통신을 도와주는 jquery를 사용하는 코드를 이용한다는 뜻이다.
min이 들어간 이유는 코드의 용량을 줄이기 위해 띄어쓰기, 주석 이런것들을 모두 제거한 한 줄의 코드이다..  
(한 줄이라고 해도 한 줄이 아니더라..)
그리고 그것을 webjars에서 가져오는데, 가져오기 위해서는 dependency에 등록해야한다.

<summary>build.gradle </summary>

>```java
>compile group: 'org.webjars', name: 'jquery', version: '3.3.1-1'
>compile group: 'org.webjars', name: 'bootstrap', version: '4.1.3'
>compile group: 'org.webjars', name: 'webjars-locator-core', version: '0.35'
>compile group: 'org.webjars', name: 'js-cookie', version: '2.1.4'
>```
> oauth2test/build.gradle에 dependencies안에 요 네개를 추가한다.

### **응용 프로그램 보안하기**
Spring Security OAuth2 dependency부터 추가해야한다.  
프로젝트를 생성할 때 추가했던 Security는 spring-boot-starter-security를 추가시켜줘서 나는 안했지만,  
>`
compile group: 'org.springframework.security.oauth.boot', name: 'spring-security-oauth2-autoconfigure', version: '2.1.1.RELEASE'
`
>`    implementation 'org.springframework.boot:spring-boot-starter-web'
`  

요거 두개.. 하면 될걸..?  
<br>
이제 메인 클래스를 생성하자. 근데 생성할 필요없다.  
이미 생성이 되어있는 **Application.java에 적어 넣을 것이다.  
<br>
맨 처음엔 그냥 @EnableOAuth2Sso 어노테이션을 추가한다!  
<br>
<summary>Oauth2testApplication.java </summary>

```JAVA
package com.oauthtest.oauth2test;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.autoconfigure.security.oauth2.client.EnableOAuth2Sso;

@SpringBootApplication
@EnableOAuth2Sso
public class OAuth2testApplication {

    ...
        
}
 ```
<br>
그리고 application.yml에 몇가지 configuration을 추가하는데, 보통 생성하면 application.properties 파일이 src/resources에 추가된다.  
properties확장자를 yml로 쿨하게 고치면 된다. ㅎ  
<br>  
<br>
<summary>application.yml </summary>

```yml
security:
  oauth2:
    client:
      clientId: Dev.Facebook의 ClientId 키
      clientSecret: Dev.Facebook의 ClientSecret 키
      accessTokenUri: https://graph.facebook.com/oauth/access_token
      userAuthorizationUri: https://www.facebook.com/dialog/oauth
      tokenName: oauth_token
      authenticationScheme: query
      clientAuthenticationScheme: form
    resource:
      userInfoUri: https://graph.facebook.com/me
...
```
  
그리고 대망의 페이스북 개발자 등록.. 생략..  
>[Mark Lee] Facebook API 사용등록  https://ericnjennifer.github.io/python_crawling/2018/01/05/PythonCrawling_Chapt2.html  

참고할 곳은 많아요.  

이제 서버를 실행시키고  http://localhost:8080 접속  

**주의할 것이 있는데,** sso는 한 번 로그인하면 그 다음부터는 자동 로그인 될 때가 많다.그래서 나는 Chrome을 사용하는데, Ctrl + Shift + N 으로 실행하면 쿠키를 바로바로 삭제할 수 있기 때문에 나처럼 사용하지 않는다면 계속해서 쿠키를 삭제해주어야 한다.  

>![image](https://user-images.githubusercontent.com/38012662/51439569-cb9fda80-1cff-11e9-8fd1-043f64bf3891.png)  
성공하면 이와 같이 페이스북 로그인 페이지가 우리의 index.html 대신에 나온다.

그리고 내 아이디를 입력하면... (개발 단계에서는 개발자 등록한 아이디만 가능하다!)
>![image](https://user-images.githubusercontent.com/38012662/51439596-25a0a000-1d00-11e9-926a-4e6c75c9a1ec.png)  
요런식으로 내 index.html이 보이게 된다!!

신기하게도, 서버 로그에는 첫 index.html을 불러오고, 몇가지 보안 비밀번호를 인식하고나서 아무것도 안한다.  

### **무슨 일이.. 일어난거지..?**
방금 내가 작성한 코드는 __authorization code grant__ 를 사용하여 페이스북에 access token을 얻어오는 Client Application이다.  

그러면 access token은 Facebook에 사용자의 어느 정도의 정보를 요구할 수 있는 ticket이 된 것이고, 여기에는 login ID와 이름 정도겠다.  
Facebook은 이 token을 복호화하고 유효한지를 판단하여 내 App에게 Permission을 주게 된다.  
그러고나면 내 App은 이제 그 유저 정보를  Spring Security context에 삽입하고 내 앱은 사용자에 대한 로그인 권한을 얻은 것이다.  

>![image](https://user-images.githubusercontent.com/38012662/51439925-2dfada00-1d04-11e9-9ccc-6b567435d296.png)  
localhost -> /login -> /dialog/oauth?client_id=... -> login.php (어떠한 처리)->facebook 로그인화면


Chrome 에서 F12를 누르고 네트워크 상황을 보면 조금 더 알 수 있는데 여기서 맨처음 요청인 localhost를 눌러보면 

>![image](https://user-images.githubusercontent.com/38012662/51439997-ef195400-1d04-11e9-96de-983d0e6010e6.png)  
왠지 가려야 할 것 같아..!

이렇게 쿠키를 설정하는 모습을 볼 수가 있다.  

자 이렇게 만들었으니, 로그인 버튼을 누르면 페이스북 로그인이 나오도록 설정해보자!!
- - -
## **Welcome Page 추가하기**

### Conditional COntent in Home Page
조건적으로 컨텐츠를 보여주기위해 우리는 자바스크립트를 추가할 것이다. AngularJS!  

일단 index.html의 __div class="container"__ 내부에 간단하게 추가한다
<summary> index.html </summary>

```html
<div class="container unauthenticated">
        With Facebook: <a href="/login">click here</a>
    </div>
    <div class="container authenticated" style="display:none">
        Logged in as: <span id="user"></span>
    </div>
</div>
```
* ### **출력화면** 
>![image](https://user-images.githubusercontent.com/38012662/51440146-27ba2d00-1d07-11e9-9efc-c9fd182a8bff.png)


로그인을 하지 않았으면 __`container unauthenticated`__ 클래스의 div태그가 보일 것이고,   
그렇지 않으면 __`container authenticated`__ 클래스의 div가 보일 것이다~! 이말이다.

그리고 자바스크립트는 바로 고 밑에
<summary>index.html </summary>

```html
<script type="text/javascript">
    $.get("/user", function(data) {
        $("#user").html(data.userAuthentication.details.name);
        $(".unauthenticated").hide()
        $(".authenticated").show()
    });
</script>
```

user라는 id를 가진 태그에게 내리는 명령이다. `data.userAuthentication.details.name` 요게 뭔갈 가져오나본데...  
암튼 가져왔다면 어떤태그는 숨기고 (`hide()`)어떤 태그는 보여줘라 (`show()`)라는 의미이다.

### **서버쪽도 바꿔야지**
<summary>Oauth2testApplication.java </summary>

```java
@SpringBootApplication
@EnableOAuth2Sso
@RestController
public class Oauth2testApplication {

    @RequestMapping("/user")
    public Principal user(Principal principal) {
        return principal;
    }

    public static void main(String[] args) { ... }

}
```

principal 자체를 return 하는 것은 보안에 좋지 않다. 빠르게 구현하려고 했을 뿐이고..    
필요한 것만 return하는 과정은 나중에 수정할 것이다.

이제 로그인을 하면 화면에 `Logged in as: NAME` 으로 잘 나올 것이다. 하지만...  
click here 조차도 보이지 않고 바로 페이스북 로그인 화면으로 넘어간다.  
<br>

그래서 __`WebSecurityConfigurer`__ 를 상속하여 configure 메소드를 Override할 것이다.  
<br>
  
Override할 때 Ctrl + o 는 오버라이딩할 메소드들을 볼 수 있다. 바로 configure를 타이핑하면 
>![image](https://user-images.githubusercontent.com/38012662/51440375-c8115100-1d09-11e9-9593-098b1ba05985.png)

요 화면이 나오는데, httpsecurity를 인자로 받는 메소드를 바꾸면 된다.

 <summary>Oauth2testApplication.java </summary>

```java
@SpringBootApplication
@EnableOAuth2Sso
@RestController
public class Oauth2testApplication extends WebSecurityConfigurerAdapter {

    @RequestMapping("/user")
    public Principal user(Principal principal) {
        return principal;
    }

    public static void main(String[] args) {
        SpringApplication.run(Oauth2testApplication.class, args);
    }
    // 추가되었다.
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.antMatcher("/**")
                .authorizeRequests()
                    .antMatchers("/", "/login", "/webjars/**", "/error**")
                    .permitAll()
                .anyRequest()
                    .authenticated();
    }
}
```

스프링 부트는 `@EnableOAuth2Sso` 포함하는 __`WebSecurityConfigurer`__ 클래스에 특별한 임무를 주었다.  
이 클래스는 OAuth2 인증 프로세서를 처리하는 security filter chain을 구성한다.

그래서 우리는 그냥 우리 welcome page 보여주고 싶어서 / 다음에 무슨 글자를 갖고 있든 간에 보안을 갖고 인증을 받아야만 넘어가도록 authorizeRequests() 메소드를 사용하면 된다.  

/error**은 왜냐면 사용자가 인증되이 않은 경우에도 응용프로그램에 문제가 있으면 Spring Boot가 오류를 렌더링 할 수 있기를 원하기 때문에 일단 열어놓음

*여기부터 좀 헷갈리지만 한마디로 버튼 클릭하면 페이스북 로그인으로 넘어가는 보안이 해제된다는 소리다.*  

>![image](https://user-images.githubusercontent.com/38012662/51440522-7964b680-1d0b-11e9-9d67-df979052cd5d.png)  
어.. 됐어.

>![image](https://user-images.githubusercontent.com/38012662/51440603-22abac80-1d0c-11e9-8952-2e5f22cf4868.png)  
로그인도 됨. 당연하지만.  
- - -
## 로그아웃 버튼 추가하기

응. 추가하면 됨.

### 클라이언트

 <summary>index.html </summary>

```html
<div class="container authenticated">
  Logged in as: <span id="user"></span>
  <div>
    <button onClick="logout()" class="btn btn-primary">Logout</button>
  </div>
</div>
```

그리고 `logout()` 함수도 script태그에 집어 넣는다.
<summary>index.html </summary>

```javascript
var logout = function() {
    $.post("/logout", function() {
        $("#user").html('');
        $(".unauthenticated").show();
        $(".authenticated").hide();
    })
    return true;
}
```
잘 보면 login할 때와 정 반대다.
### 로그아웃 Endpoint 추가
```java
@Override
protected void configure(HttpSecurity http) throws Exception {
  http.antMatcher("/**")
    ... // existing code here
    .and().logout().logoutSuccessUrl("/").permitAll();
}
```
이다음부터는 추후에... 올리는걸로. 힘들다.
