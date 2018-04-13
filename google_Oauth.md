# Google OAuth 2.0 로그인 연동

1. google API console 등 - 구글API service 에서 등록하여 아이디 비밀번호 받음

2. pom.xml dependency 추가
~~~
<dependency>
 <groupId>org.springframework.social</groupId>
 <artifactId>spring-social-google</artifactId>
 <version>1.0.0.RELEASE</version>
</dependency>
~~~

3. login.jsp
~~~
<%
  String user_name = ""; 
	if(session.getAttribute("oAuthUser") != null) {
		OAuthUser oAuthUser = (OAuthUser)session.getAttribute("oAuthUser");
		user_name = oAuthUser.getName();
%>
		<div>
			<ul class="menu">
				<li><%=user_name %></li>
				<li><a href="/oAuthLogout.do" class="button">google logout</a></li>
			</ul>
		</div>
<%} else {%>
   <a href="<c:url value="/googleLogin.do" />" class="success button">google로 로그인</a>
<%} %>
~~~

4. LoginController.java
> goGoogleSignUrl : 구글로그인 url로 요청보냄<br/> 
getGoogleSignResponse : 요청받은 정보 redirection<br/>
oAuthLogout : 로그아웃
~~~
@RequestMapping(value = "/googleLogin.do", method = RequestMethod.GET)
	public String goGoogleSignUrl(HttpServletResponse response) {
		OAuth2Operations oauthOperations = googleConnectionFactory.getOAuthOperations();
		String url = oauthOperations.buildAuthorizeUrl(GrantType.AUTHORIZATION_CODE, googleOAuth2Parameters);
		//System.out.println("google login url : " + url);
		
		return "redirect:" + url;
	}
	
	@RequestMapping(value = "/oAuthCallback.do", method = RequestMethod.GET)
	public String getGoogleSignResponse(HttpServletRequest request, HttpServletResponse response) {
		String code = request.getParameter("code");
		
		OAuth2Operations oauthOperations = googleConnectionFactory.getOAuthOperations();
		AccessGrant accessGrant = oauthOperations.exchangeForAccess(code , googleOAuth2Parameters.getRedirectUri(), null);
		
		String accessToken = accessGrant.getAccessToken();
		Long expireTime = accessGrant.getExpireTime();
		if (expireTime != null && expireTime < System.currentTimeMillis()) {
			accessToken = accessGrant.getRefreshToken();
			System.out.printf("accessToken is expired. refresh token = {}", accessToken);
		}
		Connection<Google> connection = googleConnectionFactory.createConnection(accessGrant);
		Google google = connection == null ? new GoogleTemplate(accessToken) : connection.getApi();
		
		PlusOperations plusOperations = google.plusOperations();
		Person person = plusOperations.getGoogleProfile();
		
		OAuthUser oAuthUser = new OAuthUser();
		oAuthUser.setEmail(person.getAccountEmail());
		oAuthUser.setName(person.getDisplayName());
		oAuthUser.setGender(person.getGender());
		
		HttpSession session = request.getSession();
		session.setAttribute("oAuthUser", oAuthUser );
		
		return "index";
	}
	
	@RequestMapping(value = "/oAuthLogout.do", method = RequestMethod.GET)
	public String oAuthLogout(HttpServletRequest request, HttpSession session) {
		session.invalidate();

		//String url = request.getHeader("referer");
		return "redirect:https://www.google.com/accounts/Logout?continue=https://appengine.google.com/_ah/logout?continue=http://localhost:8080/";
	}
~~~

5. LoginConfig.java
> GoogleConnectionFactory와 OAuth2Parameters 빈을 등록한다.
~~~
@Configuration
public class LoginConfig {
	
	@Bean
	public GoogleConnectionFactory googleConnectionFactory() {
		return new GoogleConnectionFactory("ID", "PASSWORD"); //생성된 아이디와 시크릿키를 넣는다
	}
	
	@Bean
	public OAuth2Parameters googleOAuth2Parameters() {
		OAuth2Parameters googleOAuth2Parameters = new OAuth2Parameters();
		googleOAuth2Parameters.setScope("https://www.googleapis.com/auth/plus.login");
		//googleOAuth2Parameters.setScope("https://www.googleapis.com/auth/plus.me");
		googleOAuth2Parameters.setRedirectUri("http://localhost:8080/oAuthCallback.do");
		return googleOAuth2Parameters;
	}
}
~~~

6. OAuthUser
> 세션에 저장 할 VO 객체
~~~
public class OAuthUser {
	private String name;
	private String email;
	private String gender;
	
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public String getEmail() {
		return email;
	}
	public void setEmail(String email) {
		this.email = email;
	}
	public String getGender() {
		return gender;
	}
	public void setGender(String gender) {
		this.gender = gender;
	}
}
~~~
