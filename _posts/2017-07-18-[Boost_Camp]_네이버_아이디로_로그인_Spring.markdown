---
layout: post
title:  "[Boost_Camp] 네이버 아이디로 로그인 Spring"
subtitle: "3주차"
date:   2017-07-20 20:40:00
categories: [study]
---

# 1 내 어플리케이션 등록하기 #

우선 어플리케이션을 등록해야 네이버 API를 사용할 수 있다. 네이버 아이디로 로그인 API를 사용한다.
 
[네이버 아이디로 로그인](https://developer.naver.com/openapi/register/login.nhn)

어플리케이션을 생성했다면,

[관리메뉴](https://developer.naver.com/openapi/register.nhn)에서 ClientID와 ClientSecret을 확인해야 합니다.

ClientSecret을 누르면 됩니다.
또한 개발이 완료되어 서비스를 사용하고 싶다면 API 상태를 서비스 적용으로 바꾸시면 됩니다.

 
 

# 2 세션 유지 및 위조 방지용 토큰 생성하기 #

가장 먼저 어플리케이션과 사용자간의 상태를 저장하는 세션 토큰을 생성해야 합니다. 이 토큰은 인증 과정의 결과로 나오는 세션 토큰과 일치하는지를 검증하여 사용자가 정상적인 요청을 했는지를 검증 하게 됩니다. 이 값은 로그인이 유지되는 동안에 저장되는 값입니다.

앞으로 몇가지 유틸들이 필요하게 될 것이기에 이런 유틸 메소드들을 모은 Utils.java 라는 클래스를 생성하겠습니다.
그리고 여기에 토큰 생성용 메소드를 만듭니다. 

```
public class Utils {
 
 public static String generateState() {
  SecureRandom random = new SecureRandom();
  return new BigInteger(130, random).toString(32);
 }
}
```

# 3 인증 요청하기 #

이제 생성된 토큰과 ClinetID를 조합하여 네이버에 인증을 요청합니다. 이때 필요한 정보는 아래과 같습니다.

 

위의 조건에 맞춰서 만들어진 URL은 아래와 같은 식으로 나타납니다.

```
https://nid.naver.com/oauth2.0/authorize?client_id=YOURCLIENTID&response_type=code&redirect_uri= 
https%3A%2F%2Fyourdomain.com%2Fcallback&state=YOURSTATETOKEN
```
 

이때 redirect_uri의 값은 인코딩된 형태로 입력해야 하는데, 이는 크롬의 개발자 모드(f12)를 켠 후 console 창에서 아래처럼 입력하면 쉽게 구할 수 있습니다.

```
 encodeURIComponent('http://localhost:8080/oauth/callback')
```


코딩을 한다면 아래처럼 나타낼 수 있습니다. 우선 변하지 않는 고정 값들과 주소는 static으로 선언해 놓았습니다.
그리고 naverLogin으로 접속하면 토큰을 생성하고 세션에 저장한 후 네이버에 인증을 요청합니다.
```
private static final String mydomain = "http%3A%2F%2Flocalhost%3A8080%2Foauth%2Fcallback";
private static final String clientId = "dCMWFJGFpqXmRgAAAA";
private static final String clientSecret = "dnHAAAAA";
private static final String requestUrl = "https://nid.naver.com/oauth2.0/authorize?client_id=" + clientId + "&response_type=code&redirect_uri="+ mydomain + "&state="; 


@RequestMapping(value = "/naverLogin")
 public String naverLogin(HttpSession session) {
  String state = Utils.generateState();     //토큰을 생성합니다.
  session.setAttribute("state", state);      //세션에 토큰을 저장합니다.
  return "redirect:" + requestUrl + state;   //만들어진 URL로 인증을 요청합니다.
 
 }
``` 

무사히 네이버에 인증을 요청하게 되면 아래와 같은 화면이 뜨면서 인증을 할 수 있습니다.




# 4 토큰 검증하기 #

 로그인 창에서 아이디와 비밀번호를 입력하여 인증 요청이 무사히 진행되었다면, 위에서 설정한 redirect_uri의 주소로 토큰이 돌아오게 됩니다. 돌아오는 URL의 형식은 다음과 같습니다.

``` 
https://yourdomain.com/callback?state=YOURSTATETOKEN&code=NAVER_OAUTH_AUTHORIZATION_CODE
```
 

이제 이 돌아온 값을 가지고 토큰을 검증하고 AccessToken을 요청해야 합니다.

```
 @RequestMapping("/callback")
 public String callback(@RequestParam String state, @RequestParam String code, HttpServletRequest request) throws UnsupportedEncodingException {
  String storedState = (String) request.getSession().getAttribute("state");  //세션에 저장된 토큰을 받아옵니다.
  if (!state.equals(storedState)) {             //세션에 저장된 토큰과 인증을 요청해서 받은 토큰이 일치하는지 검증합니다.
   System.out.println("401 unauthorized");   //인증이 실패했을 때의 처리 부분입니다.
   return "redirect:/";
  }
  
         //AccessToken 요청 및 파싱할 부분
  return "redirect:/";
 }
```


# 5 AccessToken 요청하기 #

AccessToken을 요청하면 이제서야 그 값을 가지고 네이버의 사용자 정보를 가져다 쓸 수 있습니다. 이 요청 과정의 결과 값은 네이버에서 xml과 json으로 만들어 놓았기에 xml과 json을 파싱할 수 있는 메소드를 Utils.java에 추가합니다. 
덤으로 저는 파싱한 데이터를 Map에 넣고 사용하고 싶기 때문에 Map으로 변환하는 메소드도 넣었습니다.

JSON과 XML을 다루기 위해서는 몇가지 라이브러리가 필요하므로 Maven Dependency를 추가합니다.

```
 <!-- JSON -->
  <dependency>
   <groupId>org.json</groupId>
   <artifactId>json</artifactId>
   <version>20140107</version>
  </dependency>
    
  <dependency>
   <groupId>com.fasterxml.jackson.core</groupId>
   <artifactId>jackson-core</artifactId>
   <version>${org.jackson-version}</version>
  </dependency>
  <dependency>
   <groupId>com.fasterxml.jackson.core</groupId>
   <artifactId>jackson-databind</artifactId>
   <version>${org.jackson-version}</version>
  </dependency>
```

그리고 Utils.java에 메소드를 추가합니다. 
getHtml은 요청한 주소의 html을 가져오는데 사용하는데, accessToken 요청시 헤더에 Authorization을 넣어서 요청해야 하기 때문에 그에 대한 파라미터를 받게 했습니다.

```
 public static Map<String, String> JSONStringToMap(String str){
  Map<String,String> map = new HashMap<String,String>();
  ObjectMapper mapper = new ObjectMapper();
  
  try {
   map = mapper.readValue(str, new TypeReference<HashMap<String,String>>() {});
  } catch (JsonParseException e) {
   e.printStackTrace();
  } catch (JsonMappingException e) {
   e.printStackTrace();
  } catch (IOException e) {
   e.printStackTrace();
  }
  return map;
 }
 
 public static String getHtml(String url, String authorization) {
  HttpURLConnection httpRequest = null;
  String resultValue = null;
  try {
   URL u = new URL(url);
   httpRequest = (HttpURLConnection) u.openConnection();
   httpRequest.setRequestProperty("Content-type", "text/xml; charset=UTF-8");
   
   if(authorization != null){
    httpRequest.setRequestProperty("Authorization", authorization);
   }
   httpRequest.connect();
   BufferedReader in = new BufferedReader(new InputStreamReader(httpRequest.getInputStream(), "UTF-8"));
   
   StringBuffer sb = new StringBuffer();
   String line = null;
   while((line = in.readLine()) != null){
    sb.append(line);
    sb.append("\n");
   }
   resultValue = sb.toString();
  } catch (IOException e) {

  } finally {
   if (httpRequest != null) {
    httpRequest.disconnect();
   }
  }
  return resultValue;
 }
``` 

이제 AccessToken을 요청합니다. 요청에 필요한 데이터는 아래와 같습니다.

 

위의 조건에 맞춰서 만들어지는 URL의 예는 아래와 같습니다.

```
https://nid.naver.com/oauth2.0/token?client_id=YOURCLIENTID&client_secret=YOURCLIENTSECRET&grant_type=authorization_code&state=YOURSTATETOKEN&code=AUTHORIZATIONCODE
``` 

저는 아래와 같은 식으로 4에서 받아온 state(YOURSTATETOKEN)와 code(AUTHORIZATIONCODE)를 넣어 주소를 만드는 메소드를 생성해서 사용하였습니다. 

```
 private String getAccessUrl(String state, String code) {
  String accessUrl = "https://nid.naver.com/oauth2.0/token?client_id=" + clientId + "&client_secret=" + clientSecret
    + "&grant_type=authorization_code" + "&state=" + state + "&code=" + code;
  return accessUrl;
 }
``` 

AccessToken을 요청합니다. 네이버에서는 결과 값을 json 형태로 뿌려주기 때문에 getHtml을 이용하여 받아온 후 Utils에서 만들었던 JSONStringToMap을 사용하여 json을 파싱합니다.
그리고 Map의 형태로 저장합니다.

```
 String data = Utils.getHtml(getAccessUrl(state, code), null);           //AccessToken을 요청하고 그 값을 가져옵니다.
 Map<String,String> map = Utils.JSONStringToMap(data);               //JSON의 형태로 받아온 값을 Map으로 저장합니다.
 String accessToken = map.get("access_token");
 String tokenType = map.get("token_type"); 
```


# 6 네이버 사용자 정보 가져오기 #

5에서 가져온 AcessToken으로 사용자 정보를 가져와 보겠습니다. 필요한 데이터는 아래와 같습니다.
 
 

요청이 성공한다면 아래와 같이 XML의 형태로 값을 가져오게 됩니다.

```
<?xml version="1.0" encoding="UTF-8" ?>
<data>
  <result>
     <resultcode>00</resultcode>
     <message>success</message>
  </result>
   <response>
     <email><![CDATA[platinasnow@naver.com]]></email>
     <nickname><![CDATA[심해펭귄]]></nickname>
     <enc_id><![CDATA[bdb91d2e6e40b2bb748e16007bf09711c06b437686f8f7]]></enc_id>
     <profile_image><![CDATA[https://phinf.pstatic.net/contactthumb.phinf/profile/blog/30/4/platinasnow.jpg?type=s80]]></profile_image>
      <age><![CDATA[20-29]]></age>
      <birthday><![CDATA[04-01]]></birthday>
      <gender>M</gender>
      <id><![CDATA[43711119]]></id>
  </response>
</data>
```

프로필을 요청할 주소를 선언하고, accessToken을 구해왔던 부분 밑에 사용자 정보 요청 메소드를 추가합니다.


```
 private static final String userProfileUrl = "https://apis.naver.com/nidlogin/nid/getUserProfile.xml";

 
 String profileDataXml = Utils.getHtml(userProfileUrl, tokenType +" "+ accessToken); 
// tokentype 와 accessToken을 조합한 값을 해더의 Authorization에 넣어 전송합니다. 결과 값은 xml로 출력됩니다. 
  
JSONObject jsonObject =  XML.toJSONObject(profileDataXml);     //xml을 json으로 파싱합니다.
JSONObject responseData = jsonObject.getJSONObject("data");   
//json의 구조가 data 아래에 자식이 둘인 형태여서 map으로 파싱이 안됩니다. 따라서 자식 노드로 접근합니다.
Map<String,String> userMap = Utils.JSONStringToMap(responseData.get("response").toString()); 
//사용자 정보 값은 자식노드 중에 response에 저장되어 있습니다. response로 접근하여 그 값들은 map으로 파싱합니다.
```

이제 사용자 정보를 map에 접근하여 사용 할 수 있습니다. map에 접근하는 key는 위의 응답 파라미터의 값들입니다.