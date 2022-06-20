#  NSU 과팅 APP 💌




- ### 프로젝트 개요

  - ⚡ [NSU 번개](https://www.nsu.ac.kr/)

  - `NSU 과팅 APP`은 2년동안의 코로나 시국에서의 인간관계의 결여를 탈피하기 위해 NSU 학생들만 사용할 수 있는 WEB기반의 APP서비스이다.

- ### 주요 기능 

  - **SSO 로그인 서비스**

    > 1) 우리학교인 남서울대학교 **학생포탈에 로그인이 가능한 사람**만 이용할 수 있도록 학생포탈의 토큰을 이용하여 APP에 로그인 서비스 구현.
    >
    > 2)  회원가입의 불편함을 덜기 위해, 학생포탈에서 사용하는 학번과 비번을 그대로 우리 APP에서 사용가능
    >
    > ​	- 사용자는 우리학교 학생들로만 이뤄진 집단이기때문에, 우리 APP에 대한 신뢰도가 증가함.
   
* **[기술 구현] :** 학생포탈로그인서비스에 로그인할 때 아이디 비번 값을 입력하는데,
                  id, password param값을 request해서 나온 응답코드 10000을 이용하여 
                  10000이 나올 경우 다음 페이지로 넘어가고 아닐 경우 리다이렉트 되도록 구현하였다.
```
@Controller
public class Certification {
   
   @Inject
   LoginService service;
   
   @Autowired
   private LoginService loginService;
   
   // 회원가입시 SSO 로그인 구현
   @RequestMapping(value = "/NSUOK", method = { RequestMethod.GET, RequestMethod.POST })
    public String paramTest(
          HttpServletRequest request,
           @RequestParam("id") String id,
            @RequestParam("password") String password,
            Model model,
            RedirectAttributes rttr,
            MemberInfo member
            )
            throws Exception {
        // 타임아웃 설정을 위해 HttpComponentsClientHttpRequestFactory 사용
        HttpComponentsClientHttpRequestFactory httpRequestFactory = new HttpComponentsClientHttpRequestFactory();
        httpRequestFactory.setConnectTimeout(3000);
        httpRequestFactory.setReadTimeout(5000);
        
        
        // 세션값(id) 유지 후 넘김. 
        HttpSession session = request.getSession();
        String name = id;
        session.setAttribute("id", name);
        ModelAndView mv = new ModelAndView();
        mv.setViewName(name);
        
        //인증확인
        
        String Class = request.getParameter("id");
        MemberInfo idCheck = loginService.idCheck(Class);
        
        int result1 = 0;
        
        if(idCheck != null) {
           result1 = 1;
        }

        HttpClient httpClient = HttpClientBuilder.create()
                .setMaxConnTotal(200)
                .setMaxConnPerRoute(20)
                .build();
        httpRequestFactory.setHttpClient(httpClient);
        // RestTemplate
        RestTemplate restTemplate = new RestTemplate(httpRequestFactory);
        // 파라미터 설정
        MultiValueMap<String, String> parameters = new LinkedMultiValueMap<>();
        parameters.add("id", id);
        parameters.add("password", password);
        // POST 호출
        String url = "https://sso.nsu.ac.kr/api/login";
        ResponseEntity<String> responseEntity = restTemplate.postForEntity(url, parameters, String.class);
        String body = responseEntity.getBody();
        ObjectMapper objectMapper = new ObjectMapper();
        Map result = objectMapper.readValue(body, Map.class);
        String resultCode = String.valueOf(result.get("code"));
        // 응답 코드 10000 - 성공
        if (resultCode.equals("10000") && result1 == 0) {
            System.out.println("인증 성공");
            service.potal(member);
            System.out.println(id);
            
            return "user/join";                       
        }if(resultCode.equals("10000") && result1 == 1) {
           System.out.println(result1);
           return "user/login";
           
        }else {
            System.out.println("인증 실패");
            rttr.addFlashAttribute("msg", false);
            return "redirect:/NSU";
        }

    }
   
   
   }

   ```
  
   

  - **게시판 등록 및 신청** 

    > 1)  사용자들은 자신의 소소한 개인정보(나이, MBTI, 주량)등이 포함된 게시글을 작성한다. 
    >
    > 2)  마음에 드는 게시글에 들어가서 친구신청 버튼을 클릭한다. 
    >
    
    
    
    

  - **친구 수락 및 등록**

    > 1) 친구수락 신청이 오면, 사용자는 수락이나 거절버튼을 활용할 수 있다.
    >
    > 2) 수락을 눌러 서로 친구상태가 되면, 친구 목록에 추가된다.
    >
    > 3) 서로 친구인 상태에서는 서로의 KAKAOTALK ID를 자동으로 교환할 수 있다.
    >

- ### 향후 계획


  - **사용자 프로필** : 현재 우리 프로젝트에서 보여지고 있는 단순한 프로필이 아닌 사진이나 여러 소스들이 포함된 다채로운 프로필
  - **채팅 기능** : 우리 APP에서 KAKAOTALK ID를 교환해주는 방식이 아닌, APP자체의 채팅서비스로 사용자들로 하여금 편의를 제공
  - **데이터베이스 암호화** : 사용자들의 정보가 데이터베이스에 담길 때, BCyptPasswordEncoder을 활용한 암호화 작업
  - **APP Design** : 우리 APP만의 정체성을 지닌 총체적 디자인 개편 




## 📌 목차

[Run With Me ? 🏃](#triangular_flag_on_post-run-with-me--%EF%B8%8F) 

* [사용된 도구](#hammer_and_wrench-사용된-도구)

* [시스템 아키텍쳐](#desktop_computer-시스템-아키텍쳐)

* [서비스 소개](#-서비스-소개)

* [일정](#calendar-일정)

* [저자](#-저자)

* [라이센스](#page_with_curl-라이센스)

  


## :hammer_and_wrench: 사용된 도구



![TechStack](https://user-images.githubusercontent.com/19357410/100544132-062d1380-3297-11eb-832e-9e1dd8f8da13.png)




**[ TEAM Cooperation ]**

- **GitHub** : GitHub을 활용하여 프로젝트를 관리.
  - Git Flow 에 따른 브랜치 전략 수립.
  
- **Slack** : 협업을 위한 공용 문서 및 산출물들을 공유할 수 있도록 활용.
  - 동시 문서 작성
  - 대용량 파일 첨부

## 🎞 서비스 소개

### 1. 로그인 화면

#### 1-1. 로그인 화면

<img src="https://user-images.githubusercontent.com/19357410/100543558-2dceac80-3294-11eb-9d4a-51a0c0e7757b.jpg" width="30%">

---

### 2. 메인 화면

#### 2-1. 메인 화면

<img src="https://user-images.githubusercontent.com/19357410/100543566-332bf700-3294-11eb-98ee-fb4c9274adf7.jpg" width="30%">  <img src="https://user-images.githubusercontent.com/19357410/100543569-3cb55f00-3294-11eb-8038-701758f0d13d.jpg" width="30%">

- **[메인 화면] :** 상단에는 현재 진행중인 챌린지, 중앙에는 랭킹, 하단에는 팔로우한 유저의 최신 러닝 기록을 보여준다.

---

#### 2-2. 메인 화면에서 랭커 클릭

<img src="https://user-images.githubusercontent.com/19357410/100543571-3de68c00-3294-11eb-8dce-13ead347632a.jpg" width="30%">  <img src="https://user-images.githubusercontent.com/19357410/100543578-46d75d80-3294-11eb-9855-00c81a701ecc.jpg" width="30%">

- **[메인 화면에서 랭커 클릭 시] :** 선택한 유저의 간단한 프로필, 팔로우 여부, 러닝 기록을 보여준다.
- **[랭커 프로필에서 러닝 피드 클릭 시] :** 유저의 해당 러닝 상세 기록을 보여준다.

---

#### 2-3. 메인 화면에서 친구 피드 클릭

<img src="https://user-images.githubusercontent.com/19357410/100543582-4b037b00-3294-11eb-9ef7-f924754da14e.jpg" width="30%">

* **[메인 화면에서 팔로워 피드 클릭 시] :** 팔로우한 유저의 러닝 상세 기록을 보여준다.

---

### 3. 러닝 페이지

#### 3-1. 러닝 페이지

<img src="https://user-images.githubusercontent.com/19357410/100543583-4e970200-3294-11eb-9b69-67f6f57f2af5.jpg" width="30%">  <img src="https://user-images.githubusercontent.com/19357410/100543584-4f2f9880-3294-11eb-8812-6b609c48aa0e.jpg" width="30%">  <img src="https://user-images.githubusercontent.com/19357410/100543586-4fc82f00-3294-11eb-8a76-a75ae0e5fba9.png" width="30%">

* **[러닝 페이지에서 시작 버튼 클릭 시] :** 러닝 기록을 시작한다.
* **[러닝 페이지에서 정지 버튼 클릭 시] :** 러닝을 끝내고 기록을 저장한다.
* **[러닝 페이지에서 페이지 스왑 시] :** 현재 러닝의 중간 기록을 1km 단위로 확인한다.

---

#### 3-2. 러닝 결과 페이지

<img src="https://user-images.githubusercontent.com/19357410/100543588-50f95c00-3294-11eb-93bb-d9bd055ad608.jpg" width="30%">  <img src="https://user-images.githubusercontent.com/19357410/100543589-50f95c00-3294-11eb-904b-55cd97774134.jpg" width="30%">

* **[러닝 결과] :** 현재 러닝의 부분 기록 및 전체 기록을 확인한다.

---

#### 3-3. 러닝 분석 페이지

<img src="https://user-images.githubusercontent.com/19357410/100543590-522a8900-3294-11eb-8f17-c664e27a60d0.jpg" width="30%">  <img src="https://user-images.githubusercontent.com/19357410/100543592-535bb600-3294-11eb-9525-df0f8aafde69.jpg" width="30%">

* **[러닝 기록] :** Github의 contributions을 표현하는 잔디 포멧을 가져와 개발자 감성을 살려 한달 단위로 하루에 뛴 거리를 보여주며, 이미지로 저장할 수 있다.
* **[러닝 분석 그래프] :** 이전 기록, 최근 기록과 유저들의 평균 기록을 보여주며, 이미지로 저장할 수 있다.

---

### 4. 주변 러너 추천

#### 4-1. 주변 러너 추천

<img src="https://user-images.githubusercontent.com/19357410/100543958-1a244580-3296-11eb-8501-d2a6ebc7ab19.jpg" width="30%">  <img src="https://user-images.githubusercontent.com/19357410/100543960-1b557280-3296-11eb-97bb-f78a235a0e5d.jpg" width="30%">

* **[주변 러너 추천] :** 같은 동네의 주변 러너들을 추천해주고, 클릭 시 유저의 프로필과 기록을 보여준다.

---

### 5. 일대일 채팅 및 매칭

#### 5-1. 일대일 채팅

<img src="https://user-images.githubusercontent.com/19357410/100543962-1c869f80-3296-11eb-8d9b-342b48349992.jpg" width="30%">  <img src="https://user-images.githubusercontent.com/19357410/100543963-1d1f3600-3296-11eb-9a7c-782bb19479fd.jpg" width="30%">

* **[일대일 채팅] :** 팔로우한 유저의 온라인 접속 상태가 보이며 온라인인 유저와 실시간으로 채팅한다.

---

#### 5-2. 매칭

<img src="https://user-images.githubusercontent.com/19357410/100543929-0d9fed00-3296-11eb-8ca0-1e0df7388ee6.jpg" width="30%">  <img src="https://user-images.githubusercontent.com/19357410/100543930-0f69b080-3296-11eb-929b-1c1e3bc5fb1b.jpg" width="30%">  <img src="https://user-images.githubusercontent.com/19357410/100543932-0f69b080-3296-11eb-83d0-83adcb3e1959.jpg" width="30%">  <img src="https://user-images.githubusercontent.com/19357410/100543933-10024700-3296-11eb-934f-9b131c543022.jpg" width="30%">

* **[일대일 채팅 밑 매칭 클릭 시] :** 일대일 채팅의 온라인 및 오프라인 유저의 목록 하단 매칭 버튼 클릭 시 경험치에 따른 유저를 매칭해준다.
* **[매칭 시스템] :** 성별을 선택 후 원하는 러너를 선택해 팔로우한 다음 일대일 채팅을 시작한다.

---

### 6. 지역 채팅

#### 6-1. 지역 채팅

<img src="https://user-images.githubusercontent.com/19357410/100543934-11337400-3296-11eb-93df-4bb018129209.jpg" width="30%">

* **[지역 채팅] :** 원하는 지역을 선택 시 해당 지역에서 여러 유저와 실시간으로 채팅한다.

---

### 7. 챌린지 페이지

#### 7-1. 챌린지 페이지

<img src="https://user-images.githubusercontent.com/19357410/100543935-1264a100-3296-11eb-95db-bd848289a472.jpg" width="30%">

* **[챌린지 페이지] :** 상단의 보유 마일리지가 표시되고, 현재 진행중, 진행 예정, 종료된 챌린지를 확인한다.

---

#### 7-2. 챌린지 상세

<img src="https://user-images.githubusercontent.com/19357410/100543936-12fd3780-3296-11eb-9392-7269ed244161.jpg" width="30%">  <img src="https://user-images.githubusercontent.com/19357410/100543937-12fd3780-3296-11eb-8ea9-7c279b4bfeff.jpg" width="30%">  <img src="https://user-images.githubusercontent.com/33771279/100744077-964a9480-3420-11eb-995d-97c2903c193d.PNG" width="30%"> <img src="https://user-images.githubusercontent.com/19357410/100543938-1395ce00-3296-11eb-8479-a79389bf8ff3.jpg" width="30%">  <img src="https://user-images.githubusercontent.com/19357410/100543939-142e6480-3296-11eb-9db2-4ed86ddb56f1.jpg" width="30%">  <img src="https://user-images.githubusercontent.com/19357410/100543941-142e6480-3296-11eb-907b-b69543697523.jpg" width="30%">

* **[챌린지 클릭 시] :** 챌린지 클릭 시 해당 챌린지의 상세 정보를 확인하고, 신청 버튼을 통해 챌린지에 참여한다.
* **[챌린지 신청 버튼 클릭 시] :** 챌린지 신청 버튼 클릭 시 챌린지 참여 페이지로 넘어가며, 기부 금액을 설정해 참여한다. 이 때, 기부 금액은 0원을 설정해도 참여가 가능하며, 해당 금액은 미리 충전된 마일리지에서 차감된다.
* **[충전하기 버튼 클릭 시] :** 충전하기 버튼을 클릭 시, [마이페이지]-[마일리지 충전] 탭으로 전환되며 카카오페이를 통해 충전이 가능하다.

---

### 8. 챌린지 제안

#### 8.1 챌린지 제안

<img src="https://user-images.githubusercontent.com/33771279/100743407-98f8ba00-341f-11eb-9757-517c487ae721.PNG" width="30%">  <img src="https://user-images.githubusercontent.com/33771279/100743427-9e560480-341f-11eb-99bc-94b88b1b6654.PNG" width="30%"> <img src="https://user-images.githubusercontent.com/33771279/100743445-a2822200-341f-11eb-8396-e8c922af8ecc.PNG" width="30%"> <img src="https://user-images.githubusercontent.com/33771279/100743436-a0b85e80-341f-11eb-809b-63bbd62e7a9a.PNG" width="30%"> <img src="https://user-images.githubusercontent.com/33771279/100743434-9f873180-341f-11eb-8684-ec48e036a39f.PNG" width="30%"> <img src="https://user-images.githubusercontent.com/33771279/100743430-9e560480-341f-11eb-875b-d814ff91a8aa.PNG" width="30%">

* **[챌린지 제안] :** 유저가 관리자에게 챌린지를 제안한다.

---

#### 8.2 챌린지 관리 페이지

<img src="https://user-images.githubusercontent.com/19357410/100543945-15f82800-3296-11eb-89fe-b7faf669e5e0.JPG" width="30%">  <img src="https://user-images.githubusercontent.com/19357410/100543947-1690be80-3296-11eb-9e8a-89af452253e0.JPG" width="30%"> <img src="https://user-images.githubusercontent.com/33771279/100744077-964a9480-3420-11eb-995d-97c2903c193d.PNG" width="30%">

* **[챌린지 관리 페이지] :** 챌린지 관리 페이지는 관리자 등급만 확인 가능하며, 챌린지 생성, 삭제, 수정이 가능하다.

---

### 9. 러닝 기록 조회

#### 9-1. 러닝 기록 조회

<img src="https://user-images.githubusercontent.com/19357410/100543948-17295500-3296-11eb-8d14-165996f4ae60.JPG" width="30%">  <img src="https://user-images.githubusercontent.com/19357410/100543949-17295500-3296-11eb-8a84-4eb34aac749e.JPG" width="30%">

* **[러닝 기록 조회] :** 본인의 모든 러닝 기록을 조회한다. 하단에서 러닝 기록을 클릭 시, 지도에 러닝 경로가 표시된다.
* **[활동 지역 러닝 기록 조회] :** 본인이 설정한 활동 지역 러닝 기록을 조회한다. 하단에서 러닝 기록을 클릭 시, 지도에 러닝 경로가 표시된다.
* **[러닝 기록 클릭 시] :** 러닝 상세 페이지로 이동한다.

---

### 10. 팔로우 목록

#### 10-1. 팔로우 목록 조회

<img src="https://user-images.githubusercontent.com/19357410/100543950-17c1eb80-3296-11eb-99b0-9f415388e0dd.JPG" width="30%">

* **[팔로우 목록 조회] :** 팔로워의 간단한 정보와, 말풍선 아이콘을 클릭 시 일대일 채팅으로 이동하며, 엑스 아이콘을 클릭 시 팔로우를 취소한다.

---

### 11. 유저 정보 수정

#### 11-1. 유저 정보 수정

<img src="https://user-images.githubusercontent.com/19357410/100543951-185a8200-3296-11eb-9472-75905c604228.JPG" width="30%">  <img src="https://user-images.githubusercontent.com/19357410/100543952-18f31880-3296-11eb-94de-1cb7ac9e91c6.JPG" width="30%">  <img src="https://user-images.githubusercontent.com/19357410/100543954-198baf00-3296-11eb-8438-a602935e4f9e.JPG" width="30%">  <img src="https://user-images.githubusercontent.com/19357410/100543957-1a244580-3296-11eb-9d21-354d7693f06c.JPG" width="30%">

* **[유저 정보 수정] :** 유저 정보를 수정하거나 탈퇴한다.
* **[유저 프로필 수정] :** 유저 프로필을 수정한다.

---

## :calendar: 일정

![일정](https://user-images.githubusercontent.com/19357410/100542772-7d5ea980-328f-11eb-806c-4bd76138aa1e.png)

## 👤 저자

* 이건 - Lee Gun - LeeGun@naver.com - @[imdaeyong](https://github.com/imdaeyong) [Back/PL]
* 전민우 - Jeon Min Woo - JeonMinWoo@gmail.com - @[kkmwkk](https://github.com/kkmwkk) [Back]
* 안형관 - An Hyeong Kwan - AnHyeongKwan@gmail.com - @[hyungtaik](https://github.com/hyungtaik) [Back]
* 정슬필 - Jeong Seung Pil - JeongSeungPil@gmail.com - @[LEESUNSOO](https://github.com/LEESUNSOO) [Front]


## :page_with_curl: 라이센스

```
Copyright (c) 2015 Juns Alen

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

     http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```
