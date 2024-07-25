---
title: 네이버 광고 웹 전환 추적 신 스크립트(trans) 전환가이드
layout: post
lesson: 5
---
------


# 1. 개요
## 1.1. 본 문서의 대상 서비스 및 목적
본 문서는 기존에 네이버 검색광고(SA)의 프리미엄로그분석, 네이버 성과형디스플레이광고(GFA)의 전환추적 서비스를 이용하시고 있는 광고주분들이, 사이트에 설치된 스크립트를, 기존 구 스크립트(cnv버전) 에서 신 스크립트(trans버전)로 변경하시려는 경우에 대한 가이드를 제공하는 것을 목적으로 합니다.

본 가이드는 사이트 종류로는 사이트의 소스코드를 광고주가 마음대로 변경할 수 있는 `독립몰` 을 대상으로 하고 있습니다.

`임대몰(카페24, 메이크샵 등)`의 경우는 아래 2개 가이드를 참고해주시기 바랍니다.<br>
 . [네이버 광고 웹 전환 추적을 위한 임대몰 설정 가이드](https://naver-conversiontracking.github.io/conversion-tracking/pages/02_ecom_platform_guide/)<br>
 . [네이버 광고 웹 전환 추적을 위한 임대몰 신 스크립트(trans버전) 수동설치 가이드](https://naver-conversiontracking.github.io/conversion-tracking/pages/03_ecom_platform_trans_guide/)<br>

## 1.2. 설치 및 테스트시 주의사항

> ##### WARNING
> 스크립트 설치테스트시 아래 중복전환필터링 로직을 반드시 숙지하시고 테스트 하시기 바랍니다.
신 스크립트(trans)에 의한 전환이 발생하는 경우, 그 전환과 동일한 유형의 전환으로 취급되는 구 스크립트(cnv)전환은 영구히 필터링 되므로 설치테스트시 주의가 필요하며 아래 내용을 반드시 숙지해주시기 바랍니다.
{: .block-warning }

현재 네이버 광고플랫폼(검색광고, 성과형디스플레이광고)에는 전환중복로그 발생으로, 광고 보고서에 중복으로 전환값이 잡히는 것을 방지하고자, 중복전환필터링 로직이 적용되어 있습니다.<br>
중복전환 필터링 로직은 다음과 같습니다.<br>
 . 특정 사이트식별자(네이버공통키, na_account_id) 의 동일한 것으로 취급되는 전환유형에서 신 스크립트에 의한 전환이 발생한 경우, 그 구 스크립트(cnv)에 의한 전환은 영구적으로 필터링 함<br>

예를 들어<br>
 . 네이버공통키 s_123 인 사이트에서, 구 스크립트(cnv)버전에서의 전환유형 `3번` 장바구니 를 사용하고 있었는데<br>
 . 해당 사이트에서 신 스크립트(trans)버전에서의 장바구니인 `add_to_cart` 로그가 한번이라도 발생하면, 그 뒤 부터는 구 스크립트(cnv)버전의 전환유형 `3번` 장바구니는 영구히 필터링이 됩니다. (구 스크립트(cnv)버전의 로그를 다시 보고서에 반영할 수 있도록 돌리는 방법은 없으니 주의해주시기 바랍니다.)<br>

구 스크립트(cnv) 버전에서의 전환유형 번호 와 신 스크립트(trans) 버전에서의 전환유형간에는 다음과 같이 mapping됩니다.

![05_01_02-01]({{"/assets/img/05_01_02-01.png"| relative_url}})

위 장바구니의 예시와 마찬가지로 신 스크립트(trans)버전에서 `lead` 인 전환이 발생하면, 구 스크립트(cnv)버전에서의 `4`번 전환은 영구히 필터링 됩니다.

만약, 신 스크립트(trans)에 대한  설치를 테스트하고 있으시다면, 위와 같은 필터링 로직에 해당되는 경우, 구 스크립트(cnv)에 의해 발생하는 전환이 필터링될 수 있으므로<br>
 . 신 스크립트(trans)버전의 전환유형 string을 넣으실 때(예:  `_conv.type = "OOOOOO";` 에서 `OOOOOO`부분), 사용하고자 하시는 전환유형 string의 앞에 `test_`를 넣으시는 것을 추천드립니다 (예: `add_to_cart` 를 테스트 하신다면, `test_add_to_cart`로 설정) 이렇게 하시면 `add_to_cart` 를 테스트 하시다가, 기존 구 스크립트(cnv)에 의해 발생하는 3번(장바구니)전환이 영구히 필터링 되어버리는 불상사를 방지할 수 있을 것입니다.<br>
 테스트하시고 문제 없음이 모두 확인되었을 때 원래 스펙에 있는 전환유형 string을 넣으시는 것을 권장드립니다.<br>
 
※ 참고<br>
  . 신 스크립트(trans)버전을 이용하여 테스트를 하실 때, 전환유형 string을 넣으실 때(예:  `_conv.type = "OOOOOO";` 에서 `OOOOOO`부분), 구 스크립트(cnv)버전과 신 스크립트(trans)버전의 mapping이 존재하는 `purchase`, `sign_up`, `add_to_cart`, `lead`, `custom001` 외 다른 string을 사용하시거나<br>
 . 또는 공식문서에 존재하는 string이 아닌 다른 string (예: `test_conversion`)을 사용하셔서 테스트 하시는 것도 가능합니다. <br>
 . 다만, 혹시 모를 실수를 방지하기 위해, 그리고 편의를 위해, 사용하고자 하시는 전환유형 string의 앞에 `test_OOOOOO`와 같이 사용하시는 것이 보다 편리하실 것으로 보여 이 방법을 권장드립니다.<br>


# 2. 신 스크립트(trans버전) 전환방법 가이드
## 2.0. 코드블록 정의
설명과 이해를 돕기 위해, 코드블록별 명칭을 정의합니다. 

### 2.0.1. 구 스크립트(cnv버전)의 코드블록

#### (1) 코드블록 `C1` : 전환값 설정 블록
```html
<script type="text/javascript" src="//wcs.naver.net/wcslog.js"></script>

// 전환Script를 이용하여 전환값 설정
<script type="text/javascript">
    var _nasa={};
    if (window.wcs) _nasa["cnv"] = wcs.cnv("1","결제 금액 변수(variable)");
</script>
```

#### (2) 코드블록 `C2` : 전환추적용 cookie설정 및 PV발생 블록 (전환값 설정이 된 경우, PV로그 1개에 전환값을 함께 전송)
```html
// 위 Script를 통해 설정된 전환값과 본 페이지의 로그를 서버에 전송

<script type="text/javascript" src="//wcs.naver.net/wcslog.js"></script>

<script type="text/javascript">
    if (!wcs_add) 
        var wcs_add = {};
    wcs_add["wa"] = "AccountId";    // 사이트 식별자 (=네이버공통키, na_account_id) 사이트에 따라 적절하게 변경 필요

    if (!_nasa)
        var _nasa = {};
    if (window.wcs) {
        wcs.inflow("site-domain"); // 전환추적 cookie  domain.
        wcs_do(_nasa); //서버로 로그 전송
    }
</script>
```

### 2.0.2. 신 스크립트(trans버전)의 코드블록

#### (1) 코드블록 `T1` : 전환추적용 cookie설정 및 PV발생 블록  

```html
<script type="text/javascript" src="//wcs.naver.net/wcslog.js"></script>
 
<script type="text/javascript">
 
if (window.wcs) {
    if(!wcs_add) var wcs_add = {};

    // (2) 각 사이트별 식별자 설정
    wcs_add["wa"] = "AccountId";     // 사이트 식별자 (=네이버공통키, na_account_id)
  
    // (3) 광고 전환추적을 위한 cookie domain설정
    wcs.inflow("site-domain");

    // (4) PV 이벤트 전송
    wcs_do(); // PV 이벤트 전송
 }
 
</script>
```

#### (2) 코드블록 `T2-1` : (전환) items 없는 단순 전환
```html
<script type="text/javascript" src="//wcs.naver.net/wcslog.js"></script>

<script type="text/javascript">
 
 if (window.wcs) {
     if(!wcs_add) var wcs_add = {};
     
    // (2) 각 사이트별 식별자 설정
    wcs_add["wa"] = "AccountId";     // 사이트 식별자 (=네이버공통키, na_account_id)
 
    // (5) 결제완료 전환 이벤트 전송
    var _conv = {}; // 이벤트 정보 담을 object 생성
   
    _conv.type = 'sign_up';  // object에 회원가입(sign_up) 이벤트 세팅  

    wcs.trans(_conv); // 전환 이벤트 정보를 담은 object를 서버에 전송
  }
 
</script>
```


#### (3) 코드블록 `T2-2` : (전환) 구매완료(purchase) 전환

```html
<script type="text/javascript" src="//wcs.naver.net/wcslog.js"></script>

<script type="text/javascript">
 
 if (window.wcs) {
     if(!wcs_add) var wcs_add = {};
     
    // (2) 각 사이트별 식별자 설정
    wcs_add["wa"] = "AccountId";     // 사이트 식별자 (=네이버공통키, na_account_id)
 
    // (5) 결제완료 전환 이벤트 전송
    var _conv = {}; // 이벤트 정보 담을 object 생성
   
    _conv.type = 'purchase';  // object에 구매(purchase) 이벤트 세팅  
    
    _conv.items = [       // 전환이벤트 행동의 내용 및 대상에 대한 정보 기술
          { // #1번 item
            id: '8321',                   //string 상품 id (필수), NDA광고를 이용중인 경우, 네이버쇼핑 상품EP로 전달하는 상품ID와 동일해야 함
            name: '네이버운동화83',      //string 상품 명 (필수)
            quantity: 1,                     //number 결제 수량 (필수)
            payAmount: 30000,                //number 결제 금액 (필수) (주의: 단가가 아닌 해당 상품의 전체 금액)
          },              
          { // #2번 item
            id: '9456'                   //string 상품 id (필수), NDA광고를 이용중인 경우, 네이버쇼핑 상품EP로 전달하는 상품ID와 동일해야 함
            name: '네이버티셔츠XL',      //string 상품 명 (필수)
            quantity: 2,                     //number 결제 수량 (필수)
            payAmount: 60000,                //number 결제 금액 (필수) (주의: 단가가 아닌 해당 상품의 전체 금액)
          }
    ];      
    
    _conv.value = '90000';     // item별 결제금액(payAmount)의 합       
	
    wcs.trans(_conv); // 전환 이벤트 정보를 담은 object를 서버에 전송
  }
 
</script>
```

## 2.1. 스크립트 교체 방법

### STEP 01. 코드블록 단위로 스크립트 교체 하기

![05_02_01-01]({{"/assets/img/05_02_01-01.png"| relative_url}})

전환 페이지에서는 일반적으로

구 스크립트(cnv버전)의 경우<br>
 . 코드블록 `C1` 이 실행된 뒤<br>
 . 코드블록 `C2` 가 실행되도록<br>
 되어 있을 것입니다. 

신 스크립트(trans버전)으로 변경을 하실 경우<br>
 . 코드블록 `C1`은 삭제 (혹은 주석 처리하여 동작하지 않도록 함)<br>
 . 코드블록 `C2`는 코드블록 `T1`으로 교체하고<br>
 . 그 아래에 코드블록 `T2-1`(혹은 `T2-2`)와 같은 전환 스크립트를 삽입하시면 됩니다. <br>
 (혹은 위 순서대로 Java Script가 실행 되도록 구현해주시기 바랍니다.)<br>

혹시 사이트의 특정 버튼이 클릭될 때 전환이벤트가 발생되도록 구현하고 싶으시다면
버튼이 위치한 페이지가 로딩될 때 코드블록 `T1`을 실행하고, 
버튼 클릭이벤트가 발생될 때 코드블록 `T2-1`(혹은 코드블록 `T2-2`) 이 실행되도록 구현하시면 됩니다.

※ 주의
 . T2가 먼저 실행되고, T1이 실행되는 경우 전환추적에 문제가 발생하니, 반드시 T1 뒤에 T2가 실행되도록 구현을 해주시기 바랍니다.

### STEP 02. 코드 블록에 각 사이트에 따라 별도로 설정해야 하는 부분 설정

코드 블록 교체시 코드블록 `T1`, `T2` 에서 아래 `(a) 각 사이트별 식별자` 설정, `(b) 광고 전환추적을 위한 cookie domain`설정 이 필요하며
기존에 사용하시던 값에 문제가 없으시다면, 그대로 동일하게 복사하여 사용하시면 됩니다.
```js
// (a) 각 사이트별 식별자 설정
    wcs_add["wa"] = "AccountId";     // 사이트 식별자 (=네이버공통키, na_account_id)

// (b) 광고 전환추적을 위한 cookie domain설정
  wcs.inflow("site-domain");
```

### STEP 03. 테스트 하기
실제 사이트를 방문하여 전환행동을 해보고, PV와 전환로그가 발생하는지 확인해봅니다. 
테스트방법은 https://naver.github.io/conversion-tracking/pages/04_trans_script_test_guide/ 가이드를 참고해주세요.

`1.2. 설치 및 테스트시 주의사항` 을 반드시 살펴보시기 바라며, 테스트할 때의 전환유형 string으로는 사용하고자 하는 전환유형 앞에 `test_` prefix 를 넣으시고 테스트를 하시다가, 최종적으로 적용하실 때 `test_` prefix 를 제거하시는 것을 추천드립니다.

### STEP 04. NHN Data에 검수요청 하기
(독립몰) 사이트에 신 스크립트(trans버전) 설치를 완료 하셨다면, 올바르게 전환이 설치되었는지를 확인하기 위해, 네이버 웹 전환추적 스크립트 설치 공식 아웃소싱 업체인 NHN Data에 연락을 하셔서 재검수 요청을 하시기를 권장드립니다. 재검수 요청을 하시면 전환스크립트가 정상적으로 설치되었는지 확인을 해보고, 결과를 알려드립니다. 만약 잘못된 부분이 있다면, 알려드립니다.

NHN Data 연락처<br>
. 고객센터(전화): 1877-7035<br>
. 채널 톡(채팅) 상담 : https://navercts.channel.io/home<br>
. 공용 Email : navercts@nhndata.com<br>

※ 참고<br>
 . 임대몰(카페24, 메이크샵 등)에서의 시스템적으로 자동설치되는 전환에 대해서는 별도의 전환검수가 필요없습니다. 설치가 정상적으로 되어 있는지가 궁금하시다면, [네이버 광고 웹 전환 추적 Script 테스트 가이드](https://naver-conversiontracking.github.io/conversion-tracking/pages/04_trans_script_test_guide/) 를 참고하셔서 테스트를 진행해보시기 바랍니다.<br>

Version: 20240725_01
