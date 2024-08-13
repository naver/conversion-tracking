---
title: 네이버 광고 웹 전환 추적 Script 설치 가이드 (wcs.trans버전)
layout: post
lesson: 1
---
------


# 1. 개요
## 1.1. 본 문서의 대상 서비스 및 목적

본 문서는 네이버 검색광고(SA)의 프리미엄로그분석, 네이버 성과형디스플레이광고(GFA)의 전환추적 서비스를 이용하기 위하여 광고주 사이트에 설치하는 Script에 대한 가이드를 제공하는 것을 목적으로 합니다. 

> ##### WARNING
> Script 설치시, 네이버 광고에서 제공하는 Script는 1개 종류만 설치가 되어야 합니다.  
본 문서에서 기술하는 신 스크립트(trans버전)와 구 스크립트(cnv버전)이 특정 전환 발생 시점에 같은 곳(페이지 혹은 클릭 이벤트를 받는 곳)에 있다면 중복으로 전환이벤트가 발생할 수 있으며, 이에 따라 광고 보고서에도 중복으로 전환이 집계될 수 있으므로 주의가 필요합니다.
{: .block-warning }


## 1.2. 설치 및 테스트시 주의사항

> ##### WARNING
> 스크립트 설치테스트시 아래 중복전환필터링 로직을 반드시 숙지하시고 테스트 하시기 바랍니다.
신 스크립트(trans)에 의한 전환이 발생하는 경우, 그 전환과 동일한 유형의 전환으로 취급되는 구 스크립트(cnv)전환은 영구히 필터링 되므로 설치테스트시 주의가 필요하며 아래 내용을 반드시 숙지해주시기 바랍니다.
{: .block-warning }

현재 네이버 광고플랫폼(검색광고, 성과형디스플레이광고)에는 전환중복로그 발생으로, 광고 보고서에 중복으로 전환값이 잡히는 것을 방지하고자, 중복전환필터링 로직이 적용되어 있습니다. 

중복전환 필터링 로직은 다음과 같습니다.<br>
 . 특정 사이트식별자(네이버공통키, na_account_id) 의 동일한 것으로 취급되는 전환유형에서 신 스크립트(trans버전)에 의한 전환이 발생한 경우, 그 구 스크립트(cnv버전)에 의한 전환은 영구적으로 필터링 함

예를 들어<br>
 . 네이버공통키 s_123 인 사이트에서, 구 스크립트(cnv)버전에서의 전환유형 `3번` 장바구니 를 사용하고 있었는데<br>
 . 해당 사이트에서 신 스크립트(trans)버전에서의 장바구니인 `add_to_cart` 로그가 한번이라도 발생하면, 그 뒤 부터는 구 스크립트(cnv)버전의 전환유형 `3번` 장바구니는 영구히 필터링이 됩니다. (구 스크립트(cnv)버전의 로그를 다시 보고서에 반영할 수 있도록 돌리는 방법은 없으니 주의해주시기 바랍니다.)<br>

구 스크립트(cnv) 버전에서의 전환유형 번호 와 신 스크립트(trans) 버전에서의 전환유형간에는 다음과 같이 mapping됩니다.

![04_tsg_01_02-01-01]({{"/assets/img/04_tsg_01_02-01-01.png"| relative_url}})

위 장바구니의 예시와 마찬가지로 신 스크립트(trans)버전에서 `lead` 인 전환이 발생하면, 구 스크립트(cnv)버전에서의 `4`번 전환은 영구히 필터링 됩니다.

만약, 신 스크립트(trans)에 대한  설치를 테스트하고 있으시다면, 위와 같은 필터링 로직에 해당되는 경우, 구 스크립트(cnv)에 의해 발생하는 전환이 필터링될 수 있으므로<br>
 . 신 스크립트(trans)버전의 전환유형 string을 넣으실 때(예:  `_conv.type = "OOOOOO";` 에서 `OOOOOO`부분), 사용하고자 하시는 전환유형 string의 앞에 `test_`를 넣으시는 것을 추천드립니다 (예: `add_to_cart` 를 테스트 하신다면, `test_add_to_cart`로 설정) 이렇게 하시면 `add_to_cart` 를 테스트 하시다가, 기존 구 스크립트(cnv)에 의해 발생하는 3번(장바구니)전환이 영구히 필터링 되어버리는 불상사를 방지할 수 있을 것입니다.<br>
 . 테스트하시고 문제 없음이 모두 확인되었을 때 원래 스펙에 있는 전환유형 string을 넣으시는 것을 권장드립니다.
 
※ 참고 1 <br>
  . 신 스크립트(trans)버전을 이용하여 테스트를 하실 때, 전환유형 string을 넣으실 때(예:  `_conv.type = "OOOOOO";` 에서 `OOOOOO`부분), 구 스크립트(cnv)버전과 신 스크립트(trans)버전의 mapping이 존재하는 `purchase`, `sign_up`, `add_to_cart`, `lead`, `custom001` 외 다른 string을 사용하시거나<br>
 . 또는 공식문서에 존재하는 string이 아닌 다른 string (예: `test_add_to_cart`)을 사용하셔서 테스트 하시는 것도 가능합니다. <br>
 . 다만, 혹시 모를 실수를 방지하기 위해, 그리고 편의를 위해, 사용하고자 하시는 전환유형 string의 앞에 `test_OOOOOO`와 같이 사용하시는 것이 보다 편리하실 것으로 보여 이 방법을 권장드립니다.<br>

 ※ 참고 2 <br>
위 내용 관련 궁금한 점이 있으시면, 광고 전환추적 스크립트 설치 공식 아웃소싱 업체인 NHN Data에 아래 방법으로 문의해주시면 친절하게 안내해드리겠습니다.<br>
 . 고객센터(전화): 1877-7035 <br>
 . 채널 톡(채팅) 상담 : [https://navercts.channel.io/home](https://navercts.channel.io/home) <br>
 . 공용 Email : navercts@nhndata.com <br>

------

# 2. Script 스펙
## 2.1. Script 전체 구조

Script의 개략의 구조는 다음과 같습니다.

```

(1) wcslog.js import
(2) 각 사이트별 식별자 설정 (=네이버공통키, na_account_id)
(3) 광고 전환추적을 위한 cookie domain설정
(4) PV(page view)이벤트 전송
(5) 전환이벤트 전송

```

(1), (2), (3)은 모든 페이지에 공통적으로 설정되어야 하는 부분 이며,
(4)번, (5번) 은 각 화면의 성격에 따라 필요시 들어가는 이벤트 들에 대한 Script 입니다. (PV이벤트는 기본적으로 모든 페이지에서 실행이 되어야 합니다.)

예를 들어, 결제완료 페이지가 별도로 있는 경우, 페이지에 대한 PV(page view)이벤트  및 결제완료이벤트 가 발생해야 하는데, 
이를 pseudo-code로 설명하면 다음과 같습니다. ( (4)는 PV이벤트 전송, (5)는 결제완료 이벤트 전송 )

```html
// (1) wcslog.js import
<script type="text/javascript" src="//wcs.naver.net/wcslog.js"></script>
 
<script type="text/javascript">
 
if (window.wcs) {
    if(!wcs_add) var wcs_add = {};
    // (2) 각 사이트별 식별자 설정
    wcs_add["wa"] = "AccountId";      // 사이트 식별자 (=네이버공통키, na_account_id)
 
    // (3) 광고 전환추적을 위한 cookie domain설정
    wcs.inflow("site-domain");
 
    // 아래 Script는 개별 이벤트 발생시 로그를 전송하기 위한 구역. 결제 완료 페이지이므로, PV 이벤트 와 결제완료 전환 이벤트를 전송함
 
    // (4) PV 이벤트 전송
    wcs_do(); // PV 이벤트 전송
 
    // (5) 결제완료 전환 이벤트 전송
    var _conv = {}; // 이벤트 정보를 담을 object 생성
 
    _conv.type = "purchase";  // object에 구매(purchase) 이벤트 세팅
 
    _conv.items = ... // 구매(purchase) 이벤트에 대한 세부 내용(Property) 세팅
 
    wcs.trans(_conv); // 이벤트 정보를 담은 object를 서버에 전송
 }
</script>
```


위 Script구조의 각 구역번호별 개략적인 설명은 다음과 같습니다.

| 번호 | 설명 |
| ---- | ---- |
| (1) | 이벤트 설정, 로그전송 등의 method를 가지고 있는 wcslog.js Script import |
| (2) | 각 사이트별 식별자 설정 (=네이버공통키, na_account_id) |
| (3) | 광고전환 추적을 위한 유입정보 저장 목적의 cookie domain 설정 및 cookie 저장 |
| (4) | PV이벤트 전송 |
| (5) | 전환이벤트 전송 (이벤트와 함께 전송되는 가능한 meta정보들에 대해서는 추가 설명) |

위 구조에서 사이트에 따라, 사이트 화면의 종류/수집하고자 하는 전환이벤트에 따라 달라지는 정보에 대한 상세 설명은 다음 챕터에서 설명합니다.

------
## 2.2. Script의 각 구역별 설명

위 Script 구조의 각 구역번호별로 Script 설치 방법에 대해 구체적으로 설명드립니다.

#### (1) wcslog.js import
wcslog.js는 각종 이벤트를 수집하고 서버로 전송하는 method를 가지고 있는 javascript library입니다. 이 library를 import합니다.
(이 library를 import하지 않는 경우 script는 정상동작하지 않습니다.)

#### (2) 각 사이트별 식별자(=네이버공통키, na_account_id) 설정
각 사이트별 식별자(=네이버공통키, na_account_id)입니다. 광고플랫폼에서 전환분석을 신청하면 사이트별 식별자가 지정됩니다. 이 값을  'AccountId'부분에  넣으시면 됩니다. (광고플랫폼의 전환분석 메뉴를 통해 신청 다음날 확인 가능하거나 혹은 전환분석 신청시 기재한 담당자에게 전달되는 메일에서 확인이 가능합니다.)

> ##### TIP
> 각 사이트별 식별자 (=네이버공통키, na_account_id) 확인 방법
> 로그 분석 서비스를 신청하고 (영업일 기준)1~2일 뒤에 네이버공통키(na_account_id)가 발급됩니다. 발급받은 네이버공통키는 다음과 같은 방법으로 확인할 수 있습니다.<br>
(i) 네이버 검색광고의 경우 <br>
 . 검색광고시스템의 [도구 > 프리미엄 로그 분석] 메뉴에서 [서비스 사용 현황 전체] 탭을 클릭해 [네이버공통키] 칼럼에 있는 값을 확인합니다.<br>
(ii) 네이버 성과형디스플레이광고의 경우<br>
 . 광고관리시스템의 [도구 > 전환 추적 관리] 메뉴에서 사이트를 선택하면 세부내용이 나오는 창에서 확인합니다.<br>
(iii) 공통 (메일을 통해 확인)<br>
 . 로그 분석 서비스를 신청한 다음날(영업일 기준)에 신청 시 기재한 메일 주소 혹은 광고주 메일 주소로 전달된 메일에서 확인합니다.<br>
{: .block-tip }

#### (3) 광고전환 추적을 위한 cookie domain설정
광고유입정보를 저장하기 위한 cookie에 대한 domain을 설정합니다. 일반적으로 최상위 domain 값을 넣으시면 됩니다. 
예를 들어, 사이트가 `www.abc-motors.com` 이라면, site-domain 부분에 `abc-motors.com`을 입려해주시면 됩니다.

> ##### WARNING
> 만약 다른 사이트와 내 사이트가 sub domain 혹은 sub path 로 구분이 된다면 sub domain 혹은 sub path 까지 포함된 host 값을 site-domain부분에 넣어주시기 바랍니다 
> 
예1) sub domain으로 내 사이트와 다른 사이트가 구분이 되는 경우. 내 사이트는 `aaa.abc.com` 인데, 타 사이트는 `bbb.abc.com` 일 때  
=> cookie domain: `aaa.abc.com` 을 넣습니다.
>
예2) sub path로 내 사이트와 다른 사이트가 구분이 되는 경우. 내 사이트는 `www.abc.com/aaa` 인데, 타 사이트는 `www.abc.com/bbb` 일 때
=> cookie domain: `www.abc.com/aaa` 를 넣습니다.
{: .block-warning }

#### (4) PV(page view) 이벤트 전송
PV이벤트를 서버로 전송합니다. 별다른 설정이 필요하지 않습니다.

#### (5) 전환이벤트 전송
전환이벤트를 전송하는 구역입니다. 
전환이벤트의 종류는 총 24종 + 사용자정의(custom) 10개 입니다. 
이에 대한 내역 및 사용방법은 \[2.4. 전환이벤트] 챕터에서 설명드립니다

-------
## 2.3. PV(page view) 이벤트
PV(page view)이벤트는 페이지가 열릴 때 해당 페이지의 URL, referrer등에 대한 정보를 수집서버로 전송하는 이벤트로, 로그 전송에 별다른 특별한 설정이 요구되지 않습니다. 
PV이벤트 전송을 위해서는, 기본 설정 ( (1), (2), (3) ) 후, wcs_do() method만 실행하면 됩니다.

```html
// (1) wcslog.js import
<script type="text/javascript" src="//wcs.naver.net/wcslog.js"></script>
 
<script type="text/javascript">
 
if (window.wcs) {
    if(!wcs_add) var wcs_add = {};
 
    // (2) 각 사이트별 식별자 설정
    wcs_add["wa"] = "AccountId";      // 사이트 식별자 (=네이버공통키, na_account_id)
 
    // (3) 광고 전환추적을 위한 cookie domain설정
    wcs.inflow("site-domain");
 
    // 아래 Script는 개별 이벤트 발생시 로그를 전송하기 위한 구역. 결제 완료 페이지이므로, PV 이벤트 와 결제완료 전환 이벤트를 전송함
 
    // (4) PV 이벤트 전송
    wcs_do(); // PV 이벤트 전송
 }
</script>
```

PV이벤트 와 전환이벤트를 함께 전송하는 경우(예: 구매완료 페이지)에 대해서는 다음 챕터에서 설명드리겠습니다.

------

&nbsp;
&nbsp;
&nbsp;
&nbsp;
## 2.4. 전환(conversion) 이벤트

### 2.4.1. 전환이벤트를 전송하는 Script의 모습
전환이벤트 1개를 수집서버로 전송하는 Script의 전체 모습을 예시를 통해 보면 다음과 같습니다.

```js
// (5) 전환 이벤트 전송, (예) 결제완료 이벤트 전송
 
var _conv = {}; // 전환 이벤트 정보 담을 object 생성
 
_conv.type = "purchase";  // 전환 이벤트의 종류 설정
 
_conv.id = "xxxnn"; // 전환 ID, 이용자가 한 행동에 대한 ID (예: 주문번호). 없을 경우 생성하지 않거나, null, '' 등 으로 설정
 
_conv.items = [       // 전환 이벤트 행동의 내용 및 대상에 대한 정보 기술
      {
            id: "c12354",                    //string 상품 id (필수)
            name: "naverMenShoes123",        //string 상품 명 (필수)
            category: "fashion/men/shoes",   //string 카테고리
            quantity: 1,                     //number 결제 수량 (필수)
            payAmount: 20000,                //number 결제 금액 (필수)
            option: "color:black, size:260" //string 상품 자체의 옵션 (예: 색상 )
        },
        {
            id: "c23456",                    //string 상품 id (필수)
            name: "naverWomenShoes234",      //string 상품 명 (필수)
            category: "fashion/women/shoes", //string 카테고리
            quantity: 1,                     //number 결제 수량 (필수)
            payAmount: 30000,                //number 결제 금액 (필수)
            option: "color:red, size:240"   //string 상품 자체의 옵션 (예: 색상 )
        }
]; 
 
_conv.value = "50000";
 
wcs.trans(_conv); // 전환 이벤트 정보를 담은 object를 서버에 전송
```

#### 최소(필수)스펙
일반적으로 전환이벤트를 전송하기 위한, 최소(필수) 스펙은 다음과 같습니다.
- 구매(purchase)의 경우 **전환이벤트코드** 및 **전환가치(\_conv.value)** 2가지가 필수스펙입니다.
- 그 외 전환이벤트 에서는, **전환이벤트코드**만 넣으시면 됩니다.

구매(purchase)의 경우, 최소필수스펙 예시

```js
// (5) 전환이벤트 전송, (예) 결제완료 이벤트 전송
 
var _conv = {}; // 전환이벤트 정보 담을 object 생성
 
_conv.type = "purchase";  // 전환이벤트의 종류 설정
 
_conv.value = "50000";  // 결제에 여러개 상품을 구매했다면 여기에 금액 합계
 
wcs.trans(_conv); // 전환이벤트 정보를 담은 object를 서버에 전송
```

구매(purchase) 외 다른 전환이벤트의 경우, 최소필수스펙 예시
```js
// (5) 전환이벤트 전송, (예) 신청완료 이벤트 전송
 
var _conv = {}; // 전환이벤트 정보 담을 object 생성
 
_conv.type = "lead";  // 전환이벤트의 종류 설정
 
wcs.trans(_conv); // 전환이벤트 정보를 담은 object를 서버에 전송
```

----
#### **※ 네이버 다이내믹광고(Naver Dynamic Ad)를 사용하시는 경우 최소(필수)스펙**

네이버 다이내믹광고(Naver Dynamic Ad) 상품을 이용하시는 경우, 사이트에 설치되어야 하는 Script 필수스펙이 위에서 기술된 내용보다 더 많습니다.

(a) 네이버 다이내믹광고(NDA)상품 이용시, 사이트에는 아래 5가지 전환이벤트가 모두 설치되어야 원활한 광고 노출이 가능합니다.
- 구매완료(purchase)
- 결제시작(begin_checkout)
- 장바구니담기(add_to_cart)
- 상품찜/저장(add_to_wishlist)
- 상품상세보기(view_product)

> ##### TIP
> 위 전환이벤트 용 Script를 사이트에 삽입할 때, 사이트에서 해당 행동의 시작 시점(예: 장바구니 버튼 클릭) 보다는, 해당 행동이 실제 완료되었을 때(예: 장바구니에 실제로 담기는 것이 완료되었을 때) Script가 동작하도록 구현하시는 것을 권장드립니다. 
> (행동이 실제 완료되었을 때 Script가 동작하도록 구현하는 것이 어려운 경우, 버튼 클릭과 같은 행동의 시작시점에 Script가 동작하도록 구현하시는 것도 가능합니다)
{: .block-tip }

(b) 위 전환이벤트를 설치하실 때, 각 전환이벤트에 대한 최소필수 property는 \[상품ID\](=item.id) 입니다.  
(상품 ID는 네이버 쇼핑 상품EP(Engine Page)로 전달하는 상품ID(id 컬럼으로 연동하는 값)와 동일해야 합니다.)

NDA광고상품을 사용하시는 경우, 구매완료(purchase)이벤트에 대한 최소 필수 스펙을 코드로 나타내면 다음과 같습니다.

```html
// (1) wcslog.js import
<script type="text/javascript" src="//wcs.naver.net/wcslog.js"></script>
 
<script type="text/javascript">
 
if (window.wcs) {
    if(!wcs_add) var wcs_add = {};
 
    // (2) 각 사이트별 식별자 설정
    wcs_add["wa"] = "AccountId";    // 사이트 식별자 (=네이버공통키, na_account_id)
 
    // (3) 광고 전환추적을 위한 cookie domain설정
    wcs.inflow("site-host");
 
    // 아래 Script는 개별 이벤트 발생시 로그를 전송하기 위한 구역. 결제 완료 페이지이므로, PV이벤트 와 결제완료 전환이벤트를 전송함
 
    // (4) PV이벤트 전송
    wcs_do(); // PV이벤트 전송
 
    // (5) 결제완료 전환이벤트 전송
 
    var _conv = {}; // 이벤트 정보 담을 object 생성
 
    _conv.type = "purchase";  // object에 구매(purchase) 이벤트 세팅
 
    _conv.items = [       // 전환이벤트 행동의 내용 및 대상에 대한 정보 기술
          { // #1번 item
                id: "7786"                  //string 상품 id (필수), 네이버쇼핑 상품 EP로 전달하는 상품ID와 동일해야 함
          },              
          { // #2번 item
                id: "8123"                   //string 상품 id (필수), 네이버쇼핑 상품EP로 전달하는 상품ID와 동일해야 함
          }
    ];  
 
    _conv.value = "50000"; // item #1, item #2 2개의 총 구매비용
 
    wcs.trans(_conv); // 전환이벤트 정보를 담은 object를 서버에 전송(위 item #1, #2 포함)
}

</script>
```

구매완료(purchase)가 아닌 다른 전환이벤트의 경우, `_conv.value` 는 넣지 않으셔도 됩니다.

(c) 위 전환이벤트 외에 다른 전환이벤트도 설치하시는 것은 문제 없으며, 이 경우 전환이벤트type코드 입력 외에 다른 필수사항은 없습니다.

### 2.4.2. 전환이벤트 종류 및 Property 설명

전환이벤트에는 
용도가 정해져 있는 총 24종 + 용도가 정해져 있지 않은 사용자정의(custom) 10종 이 존재합니다.

전환이벤트별 명칭, 코드명과 의미는 다음과 같습니다.
Script에 해당 전환이벤트 를 삽입할 경우, 한글 전환이벤트명이 아닌, 반드시 전환이벤트 type 코드명을 넣으셔야 합니다. (예: 구매완료의 경우, \_conv.type='purchase')

| **전환이벤트명**<img width=100/> | **전환이벤트 type 코드명**<img width=100/> | **설명**<img width=480/>                | **광고보고서제공 여부**  |
| -------------------------- | ---------------------------------- | ------------------------------------- | ------------------------------------- |
| 구매완료                       | purchase                           | 재화/용역 등을 주문완료/결제완료 하였음                | O |
| 결제 시작                      | begin_checkout                     | 결제를 하기 위한 과정에 들어갔음                    | X |
| 회원가입 완료                    | sign_up                            | 회원가입을 완료하였음                           | O |
| 장바구니 담기                    | add_to_cart                        | 장바구니에 상품을 담았음                         | O |
| 예약 완료                      | schedule                           | 예약을 완료하였음                             | O |
| 스토어 찜/저장                   | save_store                         | 스토어/사이트를 즐겨찾기 등에 저장하였음                | X |
| 상품 찜/저장 (위시리스트)        | add_to_wishlist                    | 재화/용역 등을 찜/즐겨찾기 등에 저장하였음              | O |
| 문의/상담하기                    | inquiry                            | 문의(상담) 내용을 전달완료함                      | X |
| 전화하기                       | call                               | 전화를 하였음. (전화 버튼을 클릭하였음)               | X |
| 신청완료                       | lead                               | (재화/용역에 관심이 생겨서) 연락처 등을 남기고 상담 등을 신청함 | O |
| 상품 상세보기                    | view_product                       | 특정 상품의 상품상세페이지를 보았음                   | X |
| 컨텐츠 보기                     | view_content                       | 특정 컨텐츠의 상세페이지를 보았음                    | O |
| 검색하기                       | search                             | 사이트 내에서 키워드 검색을 하였음                   | X |
| 공유하기                       | share                              | 특정 컨텐츠를 공유하였음. (공유하기를 클릭하였음)          | X |
| 쿠폰 발급받기                    | clip_coupon                        | 쿠폰을 발급 받았음.                           | X |
| 상품/서비스 목록 보기               | view_item_list                     | 상품/서비스의 목록들을 보았음                      | X |
| 업체 소개보기                    | about_us                           | 업체에 대해 소개하는 페이지를 보았음.                 | X |
| 서비스 제공자 보기                 | staff                              | 서비스를 제공하는 사람들을 소개하는 페이지를 보았음          | X |
| 사업장 위치 보기/길찾기              | location                           | 사업장의 위치 혹은 사업장을 가는 방법을 소개하는 페이지를 보았음     | X |
| 이벤트/프로모션 신청                | promotion                          | 이벤트나 프로모션을 신청하였음.                                                     | X |
| 의사소통채널 추가                  | add_contact_method                 | 메신저 추가, SNS팔로우 등 업체에 보다 쉽게 컨택할 수 있도록 <br>의사소통 채널을 추가하였음.       | X |
| 마케팅 수신동의                   | opt_in_marketing                   | 업체에서 마케팅 정보를 고객에게 보내는 데에 고객이 동의하였음             | X |
| 구독                         | subscribe                          | 업체가 정기적으로 정보를 보내는 것을 받는 데에 고객이 동의하였음.              | O |
| 리뷰/평가 쓰기                   | review                             | 상품/서비스에 대한 평가를 작성하였음                                  | X |
| 사용자정의                      | custom001 부터 <br>custom010 까지      | 용도가 정해져 있지 않은 이벤트입니다. 자유롭게 활용 가능합니다. <br>10개 타입이 있습니다. (예: custom007) | O |


각 전환이벤트별로 전환 행동에 대한 세부내역 혹은 전환행동의 대상에 대한 정보(Property)를 서버에 전송할 수 있습니다. 
전환이벤트에 추가되는 세부정보(Property)에는 다음과 같은 것들이 있습니다.

| **Property항목** | **세부항목** | **Data Type** | **의미** | **예시** | **Script예제** |
| ---- | ---- | ---- | ---- | ---- | ---- |
| id |  | string | 해당 전환 이벤트의 이용자 행동 ID. (광고주 사이트에서 생성하는 정보) | 주문번호: 20231220 | _conv.id= \"20231220\" |
| items(#1) | item.id | string | 행동의 대상이 되는 재화/용역의 ID (예: 상품ID) | 상품번호: 7789 | \_conv.items=[<br>  {  <br>    id:\"7786\",|
|  | item.name <br>(WARNING!) | string | 행동의 대상이 되는 재화/용역의 이름 | 상품명: 설화수 탄력크림 | name:\"설화수 탄력크림\", |
|  | item.category | string | 재화/용역의 카테고리 | 카테고리: 화장품/스킨케어/크림 | category:\"화장품/스킨케어/크림\", |
|  | item.quantity | number | 재화/용역의 수량 | 구매(결제)수량: 3개 | quantity:3, |
|  | item.payAmount | number | 재화/용역의 금액 (위 재화/용역 ID에 대한 총 결제금액. 단가 아님) | 결제금액: 90,000원 | payAmount:90000, |
|  | item.option | string | 재화/용역의 옵션 | 용량: 120ml | option: \"용량:120\" <br>  }<br>] |
| items(#2) | (위 #1 내용 반복) | ... |  |  |  |
| value |  | string | 복수개의 재화/용역의 전체 금액 (배송비 제외 권장) | 결제금액: 50,000원 | _conv.value=\"50000\" |

> ##### WARNING
> item.name (상품명)의 값에 single/double quotation이 있는 경우 광고전환추적을 위한 Script에 에러가 발생할 수 있습니다. item.name 의 값에서 single/double quotation을 제거하거나, 다른 문자로 대체해주시는 것을 강력하게 권장합니다. 
{: .block-warning }

----
## 2.5. 전환이벤트 와 Property 및 PV(page view)이벤트 전송 Script 예시 
전환이벤트 와 Property 및 PV(page view)이벤트를 함께 전송하는 페이지의 전체적인 Script의 모습은 다음과 같습니다. (구매완료 페이지를 예시로 설명드립니다)

```html
// (1) wcslog.js import
<script type="text/javascript" src="//wcs.naver.net/wcslog.js"></script>
 
<script type="text/javascript">
if (window.wcs) {
    if(!wcs_add) var wcs_add = {};
 
    // (2) 각 사이트별 식별자 설정
    wcs_add["wa"] = "AccountId";       // 사이트 식별자 (=네이버공통키, na_account_id)
 
    // (3) 광고 전환추적을 위한 cookie domain설정
 
    wcs.inflow("site-host");
 
    // 아래 Script는 개별이벤트 발생시 로그를 전송하기 위한 구역. 결제 완료 페이지이므로, PV이벤트 와 결제완료 전환이벤트를 전송함
 
    // (4) PV이벤트 전송
    wcs_do(); // PV이벤트 전송
 
    // (5) 결제완료 전환이벤트 전송
 
    var _conv = {}; // 전환이벤트 정보 담을 object 생성
 
    _conv.type = "purchase" ; // object에 구매(purchase) 이벤트 세팅
 
    _conv.id = "20231220"; // 아래 item전체 결제 행동에 대한 번호(예: 주문번호)
 
    _conv.items = [       // 전환이벤트 행동의 내용 및 대상에 대한 정보 기술
          { // #1번 item
                id: "7786",                    //string 상품 id (필수)
                name: "설화수 탄력크림",        //string 상품 명 (필수)
                category: "화장품/스킨케어/크림",   //string 카테고리
                quantity: 3,                     //number 결제 수량 (필수)
                payAmount: 90000,                //number 결제 금액 (필수), 1개당 3만원, 3개 9만원
                option: "용량:120" //string 상품 자체의 옵션 (예: 색상 )
           },              
           { // #2번 item
                id: "8123",                    //string 상품 id (필수)
                name: "윤조에센스",        //string 상품 명 (필수)
                category: "화장품/스킨케어/에센스",   //string 카테고리
                quantity: 2,                     //number 결제 수량 (필수)
                payAmount: 200000,                //number 결제 금액 (필수), 1개는 10만원, 2개 20만원
                option: "용량:120" //string 상품 자체의 옵션 (예: 색상 )
            }
    ];  
 
    _conv.value = "290000";     // item별 결제금액의 합
 
    wcs.trans(_conv); // 전환이벤트 정보를 담은 object를 서버에 전송(위 item #1, #2 포함)
}

</script>
```

만약, PV이벤트와 전환이벤트로그 전송이 서로 다른 시점에 발생한다면, 아래와 같은 (a), (b), (c) Script에서, 페이지 로딩시 (a)를 먼저 로딩 하고, (b), (c)는 필요한 시점에 해당 Script가 동작하도록 설정하시면 됩니다. 
((b) PV(page view)용 Script는 기본적으로 모든 페이지에서, 페이지가 열릴 때 마다 실행이 되어야 합니다. (c) 전환이벤트 용 Script는 행동이 완료되는 시점에 실행 되도록 구현하는 것을 권장 합니다. 이렇게 구현이 어려운 경우 행동이 시작되는 시점(예: 버튼 클릭)에 실행이 되도록 구현하는 것도 가능합니다.)

#### # (a) 페이지별 공통설정 Script
```html
<script type="text/javascript" src="//wcs.naver.net/wcslog.js"></script>
 
<script type="text/javascript">
 
if (window.wcs) {
    if(!wcs_add) var wcs_add = {};
    
    // (2) 각 사이트별 식별자 설정
    wcs_add["wa"] = "AccountId";      // 사이트 식별자 (=네이버공통키, na_account_id)
  
    // (3) 광고 전환추적을 위한 cookie domain설정
    wcs.inflow("site-domain");
 }
 
</script>
```


#### # (b) PV(Page View) 이벤트 전송 Script
```html
<script type="text/javascript" src="//wcs.naver.net/wcslog.js"></script>

<script type="text/javascript">
 
 if (window.wcs) {
    if(!wcs_add) var wcs_add = {};
     
    // (2) 각 사이트별 식별자 설정 
    wcs_add["wa"] = "AccountId";       // 사이트 식별자 (=네이버공통키, na_account_id)
 
    // (4) PV 이벤트 전송
    wcs_do(); // PV 이벤트 전송
  }
 
</script>
```


#### # (c) 전환이벤트 전송 Script
```html
<script type="text/javascript" src="//wcs.naver.net/wcslog.js"></script>

<script type="text/javascript">
 
 if (window.wcs) {
    if(!wcs_add) var wcs_add = {};
     
    // (2) 각 사이트별 식별자 설정
    wcs_add["wa"] = "AccountId";     // 사이트 식별자 (=네이버공통키, na_account_id)
 
    // (5) 결제완료 전환 이벤트 전송
    var _conv = {}; // 이벤트 정보 담을 object 생성
   
    _conv.type = "purchase";  // object에 구매(purchase) 이벤트 세팅  
    _conv.id = "20231220" // 전환행동에 대한 ID (구매의 경우 주문번호 권장)
    _conv.items = [       // 구매(purchase) 이벤트에 대한 세부 내용(Property) 세팅
          { // #1번 item
                id: "7786",                    //string 상품 id (필수)
                name: "설화수 탄력크림",        //string 상품 명 (필수)
                category: "화장품/스킨케어/크림",   //string 카테고리
                quantity: 3,                     //number 결제 수량 (필수)
                payAmount: 90000,                //number 결제 금액 (필수), 1개당 3만원, 3개 9만원
                option: "용량:120" //string 상품 자체의 옵션 (예: 색상 )
           },              
           { // #2번 item
                id: "8123",                    //string 상품 id (필수)
                name: "윤조에센스",        //string 상품 명 (필수)
                category: "화장품/스킨케어/에센스",   //string 카테고리
                quantity: 2,                     //number 결제 수량 (필수)
                payAmount: 200000,                //number 결제 금액 (필수), 1개는 10만원, 2개 20만원
                option: "용량:120" //string 상품 자체의 옵션 (예: 색상 )
            }
    ];  
    
    _conv.value = "290000";     // item별 결제금액(payAmount)의 합       

    wcs.trans(_conv); // 전환 이벤트 정보를 담은 object를 서버에 전송
  }
 
</script>
```

(a)와 (b)는 사이트의 모든 페이지에서 실행이 되어야 하므로, 합해서 설치하고, (c)는 전환이 발생하는 상황(버튼클릭 혹은 페이지 열림)에만 설치할 수도 있습니다

이 경우 (a)와 (b)을 합한 Script 블록의 형태는 다음과 같습니다.

#### # (a) 페이지별 공통설정 Script + (b) PV(Page View) 이벤트 전송 Script

```html
<script type="text/javascript" src="//wcs.naver.net/wcslog.js"></script>
 
<script type="text/javascript">
 
if (window.wcs) {
    if(!wcs_add) var wcs_add = {};
    
    // (2) 각 사이트별 식별자 설정
    wcs_add["wa"] = "AccountId";      // 사이트 식별자 (=네이버공통키, na_account_id)
  
    // (3) 광고 전환추적을 위한 cookie domain설정
    wcs.inflow("site-domain");

    // (4) PV 이벤트 전송
    wcs_do(); // PV 이벤트 전송
 }
 
</script>
```

Version: 20240703_01
