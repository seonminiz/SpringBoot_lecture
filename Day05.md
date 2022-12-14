# 1. Spring Security
1. Spring Security란?
- 스프링 시큐리티는 스프링기반 어플리케이션의 보안(인증(로그인)과 권한(사용자의 권한에 대하 처리), 인가(권한에 따른 사용분리) 등)을 담당하는 스프링의 하위 프레임워크
- 주로 서블릿 필터와 필터체인으로 위임모델을 구성한다.(HttpSecurity 객체를 사용하여 필터체인을 구성)
- 보안관 관련 체계적인 옵션을 많이 제공해주므로 개발자가 직접 보안관련 로직을 작성하지 않아도 된다. 

2. Spring Security의 보안용어
- 접근 주체(Principal): 보호된 리소스(어플리케이션의 기능들)에 접근하는 대상 
- 인증(Authentication): 보호된 리소스에 접근하는 대상(Principal)안에 이 유저가 누구인지, 어플리케이션의 기능을 사용할 수 있는 권한이 있는 주체인지를 확인하는 과정(로그인->하면서 인증이만들어짐)
- 인증토큰(Authentication Token): 인증과정을 통해 발급되는 토큰. 토큰 안에는 username(아이디), password(유저아이디에 대한 패스워드), 권한이 셋팅된다.  
- 인가(Authorize): 해당 리소스에 대한 접근이 허용된 권한인지 확인하는 과정
- 권한
: 특정리소스에 대한 접근 제한, 스프링 시큐리티 사용 시 모든 리소스는 접근 제한이 걸려있다. 
: 인가과정을 통해서 해당 리소스에 대한 권한을 가지고 있는 지 검사하고 권한이 있으면 해당리소스를 사용할 수 있게 권한이 없으면 해당 리소스에 접근 못하도록 설정
// dmin은 adminpage를 볼수 있고, 일반 user는 adminpage를 볼수 없는 것. 

3. Spring Security의 로그인 인증과정 (Form 기반의 로그인)
1) 사용자가 로그인 정보를 입력하고 인증 요청을 보낸다. 
2) AuthenticationFilter(구현체: UsernamePasswordAuthenticationFileter)가 사용자가 보낸 인증 요청을 인터셉한다.
사용자가 입력한 아이디와 비밀번호에 대한 유효성 검사를 진행(null값이 들어왔는지).
// 만들어 놓은 로그인 유저를 타는 것이 아니라 인터셉트해서 얘가 처리함 .
유효성 검사가 끝난 후에는 AuthenticatiionManager(구현체: ProviderManager)에게 인증용 객체(UsernamePasswordAuthenticationToken)을 만들어서 전달.
                                                            //거쳐가기만 함 
3) 인증용 객체를 전달받은 ProviderManager는 실제 인증 과정이 발생할 AuthenticationProvider에게 인증용 객체를 전달.
4) AuthenticationProvider에서 전달받은 인증용 객체로 실제 인증 과정을 처리.
5) 실제 인증 과정1 - DB에서 사용자인증정보를 가져올 UserDetailService 객체에게 아이디를 넘겨주고 DB에서 사용할 사용자 정보(아이디,패스워드,권한 등)를 UserDetails객체를 AuthenticationProvider 객체로 리턴.  
UserDetails findbyUsername();  // UserDetailService 상속받은 클래스를 만들어 UserDetails findbyUsername()메소드를 만들어 사용할 예정이다. 
6) 실제 인증 과정2 - AuthenticationProvider 다시 전달 받은 UserDetails 객체를 가지고 사용자가 입력한 정보와 비교, 인증 처리를 시도. 
(//로그인 정보 id, pw 동일할 경우) 인증이 완료되면 사용자 정보를 가지고 있는 Authentication 객체를 SecurityContextHolder에 등록. 등록 후에는 AuthenticationSucessHandler를 호출한다.
인증에 실패(로그인 정보가 틀리거나 SecurityContextHolder에 등록하다가 에러가 발생 등)하면 AuthenticationFailureHandler 호출한다.
<p style="text-align: center;"><img src="images/Security 인증 처리 과정.PNG.5"></p>

// 처음에 요청을 받는 녀석이 AuthenticationFilter. 여기에 사용자의 인증요청이 옴. 

AuthenticationFilter에서 UsernamePasswordAuthentication Token 발급. 그것을 AuthenticationManager(구현체: ProviderManger)로 전달 받고 토큰을 다시 AuthenticationProvider에 전달.
AuthenticationProvider에서 실제 인증처리가 일어남. 

UserDetailService라는 것을 통해서 findByUsername() 사용자의 정보를 조회하게 됨. UserDetails(사용자 정보, 아이디, 패스워드, 권한 등)에 전달 

UserDetails의 내용과 사용자가 입력한 아이디, 패스워드 비교후 인증 (AuthenticationProvider에서 일어남)

Authentication 객체 생성 후 AuthenticationManager전달.
AuthenticationManager가 AuthenticationFilter 전달
AuthenticationFilterd에서 받아온 정보를 SecurityContextHolder에 전달 받은 Authenication 객체를 등록.

4. Spring Security의 Fileter들
1) SecurityContextPersistenceFilter: SecurityContextRepository에서 SecurityContext를 가져오거나 저장하는 필터
2) LogoutFilter: 설정된 로그아웃 URL로 오는 요청을 감시하며, 요청이 왔을 때 해당 유저의 로그아웃 처리.
3) AuthenticationFilter
: 설정된 로그인, URL로 오는 요청을 감시하며, 요청이 왔을 때 해당 유저의 인증처리
: Authenticationmanager에게 인증용 객체(UsernamePasswordAuthenticationToken)을 넘겨준다.
: 인증 성공 시 SecurityContext에 Authentication객체를 등록 후 AuthenticationSuccessHandler 호출.
: 인증 실패 시 AuthenticationFailureHandler 호출.
4) DefaultLoginPageCeneratingFilter: 인증을 위한 로그인폼 페이지 URL을 감시. 요청이 왔을 때 로그인폼 페이지를 리턴.
5) AnonymousAuthenticationFilter: 사용자 정보가 인증되지 않았을 때 익명 사용자 정보로 인증토큰을 만들어주는 필터. username => anynomous
6) ExceptionTranslationFilter: 보호된 요청을 처리하는 과정에서 발생하는 예외에 대한 처리를 해주거나 전달해주는 역할.
7) FilterSecurityInterceptor: AccessDisicionMangaer로 권한부여 처리를 위임하므로써 접근 제어 결정을 쉽게 해준다.

5. Authentication Interface
- 인증 완료 후 SecurityContext에 등록되어 사용될 인증 완료 객체
- Prinicipal, Serializable을 상속받음.
- Collection<? extends GrantedAuthority> getAuthorities(); //Authentication에 등록된 사용자의 권한 목록
  Object getCredentials(); //주로 비밀번호 정보
  Object getDetail(); //사용자 상세 정보
  Object getPrincipal(); //주로 아이디
  boolean isAuthenticated(); //인증 여부

