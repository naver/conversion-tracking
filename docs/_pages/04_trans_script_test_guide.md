---
title: 네이버 광고 웹 전환 추적 Script 테스트 가이드
layout: post
lesson: 4
---
------


# 1. 개요
## 1.1. 본 문서의 대상 서비스 및 목적
본 문서는 네이버 검색광고(SA)의 프리미엄로그분석, 네이버 성과형디스플레이광고(GFA)의 전환추적 서비스를 이용하기 위하여 사이트에 신 스크립트(trans버전)를 설치하는 경우, Script가 정상적으로 설치되어 있는지를 테스트하는 방법을 가이드하는 것을 목적으로 합니다. 

## 1.2. 설치 및 테스트시 주의사항

> ##### WARNING
> 스크립트 설치테스트시 아래 중복전환필터링 로직을 반드시 숙지하시고 테스트 하시기 바랍니다.
신 스크립트(trans)에 의한 전환이 발생하는 경우, 그 전환과 동일한 유형의 전환으로 취급되는 구 스크립트(cnv)전환은 영구히 필터링 되므로 설치테스트시 주의가 필요하며 아래 내용을 반드시 숙지해주시기 바랍니다.
{: .block-warning }

현재 네이버 광고플랫폼(검색광고, 성과형디스플레이광고)에는 전환중복로그 발생으로, 광고 보고서에 중복으로 전환값이 잡히는 것을 방지하고자, 중복전환필터링 로직이 적용되어 있습니다. 

중복전환 필터링 로직은 다음과 같습니다.<br>
 . 특정 사이트식별자(네이버공통키, na_account_id) 의 동일한 것으로 취급되는 전환유형에서 신 스크립트에 의한 전환이 발생한 경우, 그 구 스크립트(cnv)에 의한 전환은 영구적으로 필터링 함

예를 들어<br>
 . 네이버공통키 s_123 인 사이트에서, 구 스크립트(cnv)버전에서의 전환유형 `3번` 장바구니 를 사용하고 있었는데<br>
 . 해당 사이트에서 신 스크립트(trans)버전에서의 장바구니인 `add_to_cart` 로그가 한번이라도 발생하면, 그 뒤 부터는 구 스크립트(cnv)버전의 전환유형 `3번` 장바구니는 영구히 필터링이 됩니다. (구 스크립트(cnv)버전의 로그를 다시 보고서에 반영할 수 있도록 돌리는 방법은 없으니 주의해주시기 바랍니다.)<br>

구 스크립트(cnv) 버전에서의 전환유형 번호 와 신 스크립트(trans) 버전에서의 전환유형간에는 다음과 같이 mapping됩니다.

![04_tsg_01_02-01-01]({{"/assets/img/04_tsg_01_02-01-01.png"| relative_url}})

위 장바구니의 예시와 마찬가지로 신 스크립트(trans)버전에서 `lead` 인 전환이 발생하면, 구 스크립트(cnv)버전에서의 `4`번 전환은 영구히 필터링 됩니다.

만약, 신 스크립트(trans)에 대한  설치를 테스트하고 있으시다면, 위와 같은 필터링 로직에 해당되는 경우, 구 스크립트(cnv)에 의해 발생하는 전환이 필터링될 수 있으므로<br>
 . 신 스크립트(trans)버전의 전환유형 string을 넣으실 때(예:  `_conv.type = "OOOOOO";` 에서 `OOOOOO`부분), 사용하고자 하시는 전환유형 string의 앞에 `test_`를 넣으시는 것을 추천드립니다 (예: `add_to_cart` 를 테스트 하신다면, `test_add_to_cart`로 설정) 이렇게 하시면 `add_to_cart` 를 테스트 하시다가, 기존 구 스크립트(cnv)에 의해 발생하는 3번(장바구니)전환이 영구히 필터링 되어버리는 불상사를 방지할 수 있을 것입니다.<br>
 . 테스트하시고 문제 없음이 모두 확인되었을 때 원래 스펙에 있는 전환유형 string을 넣으시는 것을 권장드립니다.
 
※ 참고<br>
  . 신 스크립트(trans)버전을 이용하여 테스트를 하실 때, 전환유형 string을 넣으실 때(예:  `_conv.type = "OOOOOO";` 에서 `OOOOOO`부분), 구 스크립트(cnv)버전과 신 스크립트(trans)버전의 mapping이 존재하는 `purchase`, `sign_up`, `add_to_cart`, `lead`, `custom001` 외 다른 string을 사용하시거나<br>
 . 또는 공식문서에 존재하는 string이 아닌 다른 string (예: `test_add_to_cart`)을 사용하셔서 테스트 하시는 것도 가능합니다. <br>
 . 다만, 혹시 모를 실수를 방지하기 위해, 그리고 편의를 위해, 사용하고자 하시는 전환유형 string의 앞에 `test_OOOOOO`와 같이 사용하시는 것이 보다 편리하실 것으로 보여 이 방법을 권장드립니다.<br>
 

# 2. 테스트 방법
## 2.1. 테스트 전 준비사항
 (1) 테스트를 하고자 하는 사이트에 광고전환추적을 위한 일련의 Script를 삽입을 완료합니다. (테스트하고자하는 특정 페이지에만 Script삽입을 하고 테스트를 진행해도 무방합니다)<br>
 (2) 테스트시 이용하려는 단말기(예: 데스크톱 PC)에서 HTTP 를 모니터링 할 수 있는 툴을 준비합니다.<br>
 (3) json 을 쉽게 보여주는 json parser 툴/웹서비스를 준비합니다. <br>

※ [참고1] HTTP모니터링 툴 관련<br>
 . 단말기에서 로그 수집서버로 전송하는 모든 로그를 보여주는 HTTP 모니터링 툴을 준비해주시기 바랍니다. (Chrome개발자도구 의 경우, 해당 브라우저창에서 발생하는 HTTP만 보여주기 때문에 모니터링 툴로는 적합하지 않은 경우가 있습니다. 본 문서에는 Fiddler Classic 이라는 툴을 이용한 테스트 방법을 안내드립니다)<br>
  . Fiddler Classic 프로그램은 https://www.telerik.com/fiddler/fiddler-classic 에서 다운로드 받으실 수 있습니다.<br>

※ [참고2] json parser 관련<br>
 . 수집서버에 전송되는 정보는 plain text인 json format으로 되어 있어, 눈으로 바로 보기에는 가독성이 많이 떨어집니다. json parser를 이용하시면 정보를 보다 쉽게 확인하실 수 있습니다.<br>
 . json을 parsing 하는 툴/웹서비스를 준비함에 있어, 2중 json (nested json)까지 볼 수 있는 툴/웹서비스 준비가 필요합니다.<br>
 . 본 문서에서는 https://jsongrid.com/ 를 이용한 테스트 방법을 안내드립니다.<br>

## 2.2. 툴 설정 / 사용방법
### 2.2.1. HTTP모니터링 툴 Fiddler Classic 설정 / 사용 방법

HTTP 모니터링 툴 Fiddler Classic 을 다음과 같이 설정하면 테스트를 좀 더 편하게 진행할 수 있습니다.

■ Fiddler Classic 설정방법<br>
 . Fiddler Classic을 실행합니다.<br>
 . 좌측 하단에 `Capturing`이 표시되도록 합니다 (표시가 안되면 해당 위치를 마우스로 클릭하면 변경됨)<br>
 . 우측 TAB중에 Filters를 선택합니다.<br>
 . UserFilters 를 클릭하여 Check 합니다.<br>
 . Hosts 부분에 combo box에 `-No Zone Filter - `, `Show only the following Hosts`를 선택합니다.<br>
 . Text Area 부분에 `wcs.naver.com`을 입력합니다. (`wcs.naver.com` 은 로그 수집 서버 입니다.)<br>
 . 다른 TAB으로 이동하면 저장됩니다.<br>
위와 같이 설정하면, 테스트시 발생하는 여러가지 HTTP 중, 광고전환관련 로그 수집서버인 `wcs.naver.com` 으로 전송되는 로그만 툴에 보여져 테스트를 보다 편하게 하실 수 있습니다.<br>

![04_tsg_02_02_00-01]({{"/assets/img/04_tsg_02_02_00-01.png"| relative_url}})

■ Fiddler Classic 에서 로그 확인 방법

로그 발생시, 좌측 HTTP 목록에서 (host) wcs.naver.com 을 클릭하시고, 우측 TAB중에 `Inspectors`를 선택, 하단 TAB중에 `Raw`를 선택합니다.<br>
하늘색 패널의 하단 부분의 `{"wa":...`로 시작하는 부분 부터 끝까지가 사이트에서 발생하는 로그의 `정보`에 해당되는 부분입니다.<br>

![04_tsg_02_02_01-02]({{"/assets/img/04_tsg_02_02_01-02.png"| relative_url}})


### 2.2.2. json parser 사이트 jsongrid.com 사이트 사용 방법

■ PV(Page View)로그 확인 방법

브라우저를 열고 https://jsongrid.com/ 에 접속합니다.
HTTP모니터링 툴에서 로그의 `정보`에 해당되는 부분을 복사합니다. (아래는 로그의 `정보`부분 예시 입니다.)
```
{"wa":"s_305c16ba63b1","u":"http://OOOOOO.www267.freesell.co.kr/index.html","e":"http://OOOOOO.www267.freesell.co.kr/","bt":"1721270523","os":"Win32","ln":"ko-KR","sr":"2560x1440","bw":1319,"bh":804,"c":24,"j":"N","jv":"1.8","k":"Y","ct":"","cs":"EUC-KR","tl":"","vs":"0.8.13","nt":1721270775298,"fwb":"1974FD3cvdMsebsbpmSZFtK.1720764497469","ui":"{\"nac\":\"s3aQBMAedgpJB\"}","ext":"{\"wot\":54.60000002384186}"}
```

복사한 Text를 사이트 창 왼편에 붙이고, 중간에 화살표 버튼을 클릭합니다. 우측 `GRID` 부분에서 json 정보를 표의 형태로 깔끔하게 확인할 수 있습니다.

![04_tsg_02_02_02-01]({{"/assets/img/04_tsg_02_02_02-01.png"| relative_url}})

■ 전환로그 확인 방법
브라우저를 열고, 창을 2개를 띄우고, 2개의 창 모두에서 https://jsongrid.com/ 에 접속합니다. (창 2개를 각각 `1번 윈도우`, `2번 윈도우` 로 칭합니다.)
HTTP모니터링 툴에서 로그의 `정보`에 해당되는 부분을 복사합니다. (아래는 로그의 `정보`부분 예시 입니다.)

```
{"wa":"s_305c16ba63b1","e":"http://OOOOOO.www267.freesell.co.kr/index.html","u":"http://OOOOOO.www267.freesell.co.kr/shop/shopdetail.html?branduid=12235601&search=&xcode=001&mcode=001&scode=&special=2&GfDT=bm9%2FW1w%3D","vs":"0.8.13","nt":1721271053466,"t":"conv","trans":"{\"type\":\"view_product\",\"items\":[{\"id\":\"12235601\",\"name\":\"고양이 사료\"}]}","fwb":"1974FD3cvdMsebsbpmSZFtK.1720764497469","ui":"{\"nac\":\"s3aQBMAedgpJB\"}","ext":"{\"wot\":478.69999998807907}"}
```

복사한 Text를 `1번 윈도우` 왼편에 붙이고, 중간에 화살표 버튼을 클릭합니다. 우측 `GRID` 부분에서 json 정보를 표의 형태로 깔끔하게 확인할 수 있습니다.

![04_tsg_02_02_02-02]({{"/assets/img/04_tsg_02_02_02-02.png"| relative_url}})

우측 `GRID` 부분의 trans항목의 값 부분을 마우스로 빠르게 3번클릭을 합니다. 그러면 값부분의 전체를 선택할 수 있습니다.
(혹은 마우스로 drag를 하셔도 됩니다)
이 Text를 복사를 하신 뒤, `2번 윈도우` 의 왼쪽에 불여 넣고, 중간에 화살표 버튼을 클릭한 뒤, 우측 상단에 Expand All 버튼을 클릭합니다.

![04_tsg_02_02_02-03]({{"/assets/img/04_tsg_02_02_02-03.png"| relative_url}})

그러면 우측 창에서 trans 전환 값의 모양을 확인할 수 있으며, 전환유형 및 item 관련 정보를 확인하실 수 있습니다.

## 2.2. 발생 로그에 대한 이해

수집서버로 전송되는 로그는 크게 2가지가 있습니다.

#### (1) PV(Page View)로그
모든 페이지에서 페이지가 로딩할 때마다 발생하는 로그입니다. (새로고침을 할 경우에도 발생합니다)
PV 로그 발생시, 서버로 전송되는 HTTP(HTTP Request)의 모습은 다음과 같으며 
```
POST https://wcs.naver.com/b HTTP/1.1
Host: wcs.naver.com
Connection: keep-alive
Content-Length: 603
sec-ch-ua: "Not/A)Brand";v="8", "Chromium";v="126", "Google Chrome";v="126"
sec-ch-ua-platform: "Windows"
sec-ch-ua-mobile: ?0
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/126.0.0.0 Safari/537.36
Content-Type: text/plain;charset=UTF-8
Accept: */*
Origin: http://OOOOOO.www267.freesell.co.kr
Sec-Fetch-Site: cross-site
Sec-Fetch-Mode: no-cors
Sec-Fetch-Dest: empty
Referer: http://OOOOOO.www267.freesell.co.kr/shop/shopdetail.html?branduid=12235601&search=&xcode=001&mcode=001&scode=&special=2&GfDT=bm9%2FW1w%3D
Accept-Encoding: gzip, deflate, br, zstd
Accept-Language: ko-KR,ko;q=0.9,en-US;q=0.8,en;q=0.7,ja;q=0.6
Cookie: ASID=0a1924b30000018b33ea9f2a00008e9c; buid=DSC5wraurNnKpdy8r7Sg64j54w; BUID=BRCdtrSSEK_Z-thTqjEAp1WaDQ; NWB=25eb1f3500203b16c7d92f9337cde4fb.1719127593463; NNB=KG346LE72R3WM; BUC=UnkOGSf9sRN7fM2a_lD0b8Sp69wSE2GP04vRQF6O-BE=

{"wa":"s_305c16ba63b1","u":"http://OOOOOO.www267.freesell.co.kr/shop/shopdetail.html?branduid=12235601&search=&xcode=001&mcode=001&scode=&special=2&GfDT=bm9%2FW1w%3D","e":"http://OOOOOO.www267.freesell.co.kr/index.html","bt":"1721277427","os":"Win32","ln":"ko-KR","sr":"2560x1440","bw":1319,"bh":804,"c":24,"j":"N","jv":"1.8","k":"Y","ct":"","cs":"EUC-KR","tl":"%5B%EA%B3%A0%EC%96%91%EC%9D%B4%20%EC%82%AC%EB%A3%8C%20%ED%85%8C%EC%8A%A4%ED%8A%B8%5D","vs":"0.8.13","nt":1721277432718,"fwb":"1974FD3cvdMsebsbpmSZFtK.1720764497469","ui":"{\"nac\":\"s3aQBMAedgpJB\"}","ext":"{\"wot\":397.30000001192093}"}
```

정보로 이용되는 부분은 다음 부분 입니다.
```
{"wa":"s_305c16ba63b1","u":"http://OOOOOO.www267.freesell.co.kr/shop/shopdetail.html?branduid=12235601&search=&xcode=001&mcode=001&scode=&special=2&GfDT=bm9%2FW1w%3D","e":"http://OOOOOO.www267.freesell.co.kr/index.html","bt":"1721277427","os":"Win32","ln":"ko-KR","sr":"2560x1440","bw":1319,"bh":804,"c":24,"j":"N","jv":"1.8","k":"Y","ct":"","cs":"EUC-KR","tl":"%5B%EA%B3%A0%EC%96%91%EC%9D%B4%20%EC%82%AC%EB%A3%8C%20%ED%85%8C%EC%8A%A4%ED%8A%B8%5D","vs":"0.8.13","nt":1721277432718,"fwb":"1974FD3cvdMsebsbpmSZFtK.1720764497469","ui":"{\"nac\":\"s3aQBMAedgpJB\"}","ext":"{\"wot\":397.30000001192093}"}
```

위 json을 보기좋게 정리하면 다음과 같은 항목들이 있음을 알 수 있습니다.<br>
(`wa`: 네이버공통키, `u`: 방문한 페이지의 URL, `e`: referrer)

![04_tsg_02-02-00-01]({{"/assets/img/04_tsg_02-02-00-01.png"| relative_url}})


#### (2) 전환로그
사이트에서 발생하는 이용자의 행동 중, 측정하고자 하는 특정 행동이 발생했을 때만 발생하는 로그입니다.<br>
특정 페이지를 방문할 경우, 특정 행동이 완료된 경우, 혹은 특정 버튼이 클릭된 경우등에만 발생하도록 할 수 있습니다. <br>

예를 들어 상품상세페이지 방문을 측정하고자 하는 경우, 올바르게 설치가 된 경우라면, 상품상세페이지에 방문한 경우 PV(Page View)로그와 전환로그 2개가 발생하며, 이 중 전환로그에 대한 HTTP(HTTP Request)의 모습은 다음과 같으며

```
POST https://wcs.naver.com/b HTTP/1.1
Host: wcs.naver.com
Connection: keep-alive
Content-Length: 498
sec-ch-ua: "Not/A)Brand";v="8", "Chromium";v="126", "Google Chrome";v="126"
sec-ch-ua-platform: "Windows"
sec-ch-ua-mobile: ?0
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/126.0.0.0 Safari/537.36
Content-Type: text/plain;charset=UTF-8
Accept: */*
Origin: http://OOOOOO.www267.freesell.co.kr
Sec-Fetch-Site: cross-site
Sec-Fetch-Mode: no-cors
Sec-Fetch-Dest: empty
Referer: http://OOOOOO.www267.freesell.co.kr/shop/shopdetail.html?branduid=12235601&search=&xcode=001&mcode=001&scode=&special=2&GfDT=bm9%2FW1w%3D
Accept-Encoding: gzip, deflate, br, zstd
Accept-Language: ko-KR,ko;q=0.9,en-US;q=0.8,en;q=0.7,ja;q=0.6
Cookie: ASID=0a1924b30000018b33ea9f2a00008e9c; buid=DSC5wraurNnKpdy8r7Sg64j54w; BUID=BRCdtrSSEK_Z-thTqjEAp1WaDQ; NWB=25eb1f3500203b16c7d92f9337cde4fb.1719127593463; NNB=KG346LE72R3WM; BUC=UnkOGSf9sRN7fM2a_lD0b8Sp69wSE2GP04vRQF6O-BE=

{"wa":"s_305c16ba63b1","e":"http://OOOOOO.www267.freesell.co.kr/index.html","u":"http://OOOOOO.www267.freesell.co.kr/shop/shopdetail.html?branduid=12235601&search=&xcode=001&mcode=001&scode=&special=2&GfDT=bm9%2FW1w%3D","vs":"0.8.13","nt":1721271053466,"t":"conv","trans":"{\"type\":\"view_product\",\"items\":[{\"id\":\"12235601\",\"name\":\"고양이 사료\"}]}","fwb":"1974FD3cvdMsebsbpmSZFtK.1720764497469","ui":"{\"nac\":\"s3aQBMAedgpJB\"}","ext":"{\"wot\":478.69999998807907}"}
```

정보로 이용되는 부분은 다음 부분 입니다.

```
{"wa":"s_305c16ba63b1","e":"http://OOOOOO.www267.freesell.co.kr/index.html","u":"http://OOOOOO.www267.freesell.co.kr/shop/shopdetail.html?branduid=12235601&search=&xcode=001&mcode=001&scode=&special=2&GfDT=bm9%2FW1w%3D","vs":"0.8.13","nt":1721271053466,"t":"conv","trans":"{\"type\":\"view_product\",\"items\":[{\"id\":\"12235601\",\"name\":\"고양이 사료\"}]}","fwb":"1974FD3cvdMsebsbpmSZFtK.1720764497469","ui":"{\"nac\":\"s3aQBMAedgpJB\"}","ext":"{\"wot\":478.69999998807907}"}
```

위 json을 보기좋게 정리하면 다음과 같은 항목들이 있음을 알 수 있습니다.<br>
(`trans`: 발생한 전환에 대한 정보, `trans`의 `type`은 전환유형영문코드, `items`는 상품에 대한 정보)

![04_tsg_02-02-00-02]({{"/assets/img/04_tsg_02-02-00-02.png"| relative_url}})

## 2.3. 테스트 방법

`2.1. 테스트 전 준비사항` 에서 안내드린 것과 같은 준비를 마치셨다면, 다음 STEP에 따른 테스트를 진행해주시기 바랍니다.

#### STEP 01. 테스트 지원 툴을 실행합니다.
HTTP모니터링 툴을 실행합니다. 테스트전 준비사항과 같이 설정이 되어 있는지 확인합니다.
(HTTP 모니터링 툴, json parser 툴/웹사이트)

#### STEP 02. 테스트 대상 사이트의 메인페이지에 들어갑니다.
테스트 대상 사이트의 메인페이지에 들어갑니다. 
PV 로그 1개가 발생합니다. (일반적인 경우 PV로그가 발생함만 확인하면 됩니다.)

PV로그 예시)

```
POST https://wcs.naver.com/b HTTP/1.1
Host: wcs.naver.com
Connection: keep-alive
Content-Length: 519
sec-ch-ua: "Not/A)Brand";v="8", "Chromium";v="126", "Google Chrome";v="126"
sec-ch-ua-platform: "Windows"
sec-ch-ua-mobile: ?0
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/126.0.0.0 Safari/537.36
Content-Type: text/plain;charset=UTF-8
Accept: */*
Origin: http://gradion7.www267.freesell.co.kr
Sec-Fetch-Site: cross-site
Sec-Fetch-Mode: no-cors
Sec-Fetch-Dest: empty
Referer: http://gradion7.www267.freesell.co.kr/index.html
Accept-Encoding: gzip, deflate, br, zstd
Accept-Language: ko-KR,ko;q=0.9,en-US;q=0.8,en;q=0.7,ja;q=0.6
Cookie: ASID=0a1924b30000018b33ea9f2a00008e9c; buid=DSC5wraurNnKpdy8r7Sg64j54w; BUID=BRCdtrSSEK_Z-thTqjEAp1WaDQ; NWB=25eb1f3500203b16c7d92f9337cde4fb.1719127593463; NNB=KG346LE72R3WM; BUC=UnkOGSf9sRN7fM2a_lD0b8Sp69wSE2GP04vRQF6O-BE=

{"wa":"s_305c16ba63b1","u":"http://gradion7.www267.freesell.co.kr/index.html","e":"http://gradion7.www267.freesell.co.kr/shop/shopdetail.html?branduid=12235601&search=&xcode=001&mcode=001&scode=&special=2&GfDT=bm9%2FW1w%3D","bt":"1721277432","os":"Win32","ln":"ko-KR","sr":"2560x1440","bw":1319,"bh":804,"c":24,"j":"N","jv":"1.8","k":"Y","ct":"","cs":"EUC-KR","tl":"","vs":"0.8.13","nt":1721280238429,"fwb":"1974FD3cvdMsebsbpmSZFtK.1720764497469","ui":"{\"nac\":\"s3aQBMAedgpJB\"}","ext":"{\"wot\":1397.5999999940395}"}
```

#### STEP 03. 사이트에서 전환로그를 발생시킵니다.
##### (1) 페이지에서 전환이 발생하는 경우 (예: 상품상세페이지)
PV로그 1개와 페이지에서 발생하는 전환로그 1개로 총 2개의 로그가 발생합니다.

상품상세페이지에서 전환이 발생하는 경우 예시)

PV로그
```
POST https://wcs.naver.com/b HTTP/1.1
Host: wcs.naver.com
Connection: keep-alive
Content-Length: 601
sec-ch-ua: "Not/A)Brand";v="8", "Chromium";v="126", "Google Chrome";v="126"
sec-ch-ua-platform: "Windows"
sec-ch-ua-mobile: ?0
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/126.0.0.0 Safari/537.36
Content-Type: text/plain;charset=UTF-8
Accept: */*
Origin: http://gradion7.www267.freesell.co.kr
Sec-Fetch-Site: cross-site
Sec-Fetch-Mode: no-cors
Sec-Fetch-Dest: empty
Referer: http://gradion7.www267.freesell.co.kr/shop/shopdetail.html?branduid=12235601&search=&xcode=001&mcode=001&scode=&special=2&GfDT=bml0W1w%3D
Accept-Encoding: gzip, deflate, br, zstd
Accept-Language: ko-KR,ko;q=0.9,en-US;q=0.8,en;q=0.7,ja;q=0.6
Cookie: ASID=0a1924b30000018b33ea9f2a00008e9c; buid=DSC5wraurNnKpdy8r7Sg64j54w; BUID=BRCdtrSSEK_Z-thTqjEAp1WaDQ; NWB=25eb1f3500203b16c7d92f9337cde4fb.1719127593463; NNB=KG346LE72R3WM; BUC=UnkOGSf9sRN7fM2a_lD0b8Sp69wSE2GP04vRQF6O-BE=

{"wa":"s_305c16ba63b1","u":"http://gradion7.www267.freesell.co.kr/shop/shopdetail.html?branduid=12235601&search=&xcode=001&mcode=001&scode=&special=2&GfDT=bml0W1w%3D","e":"http://gradion7.www267.freesell.co.kr/index.html","bt":"1721280238","os":"Win32","ln":"ko-KR","sr":"2560x1440","bw":1319,"bh":804,"c":24,"j":"N","jv":"1.8","k":"Y","ct":"","cs":"EUC-KR","tl":"%5B%EA%B3%A0%EC%96%91%EC%9D%B4%20%EC%82%AC%EB%A3%8C%20%ED%85%8C%EC%8A%A4%ED%8A%B8%5D","vs":"0.8.13","nt":1721280286865,"fwb":"1974FD3cvdMsebsbpmSZFtK.1720764497469","ui":"{\"nac\":\"s3aQBMAedgpJB\"}","ext":"{\"wot\":170.10000002384186}"}
```

전환로그(상품상세조회 `view_product`)
```
POST https://wcs.naver.com/b HTTP/1.1
Host: wcs.naver.com
Connection: keep-alive
Content-Length: 483
sec-ch-ua: "Not/A)Brand";v="8", "Chromium";v="126", "Google Chrome";v="126"
sec-ch-ua-platform: "Windows"
sec-ch-ua-mobile: ?0
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/126.0.0.0 Safari/537.36
Content-Type: text/plain;charset=UTF-8
Accept: */*
Origin: http://gradion7.www267.freesell.co.kr
Sec-Fetch-Site: cross-site
Sec-Fetch-Mode: no-cors
Sec-Fetch-Dest: empty
Referer: http://gradion7.www267.freesell.co.kr/shop/shopdetail.html?branduid=12235601&search=&xcode=001&mcode=001&scode=&special=2&GfDT=bml0W1w%3D
Accept-Encoding: gzip, deflate, br, zstd
Accept-Language: ko-KR,ko;q=0.9,en-US;q=0.8,en;q=0.7,ja;q=0.6
Cookie: ASID=0a1924b30000018b33ea9f2a00008e9c; buid=DSC5wraurNnKpdy8r7Sg64j54w; BUID=BRCdtrSSEK_Z-thTqjEAp1WaDQ; NWB=25eb1f3500203b16c7d92f9337cde4fb.1719127593463; NNB=KG346LE72R3WM; BUC=UnkOGSf9sRN7fM2a_lD0b8Sp69wSE2GP04vRQF6O-BE=

{"wa":"s_305c16ba63b1","e":"http://gradion7.www267.freesell.co.kr/index.html","u":"http://gradion7.www267.freesell.co.kr/shop/shopdetail.html?branduid=12235601&search=&xcode=001&mcode=001&scode=&special=2&GfDT=bml0W1w%3D","vs":"0.8.13","nt":1721280287073,"t":"conv","trans":"{\"type\":\"view_product\",\"items\":[{\"id\":\"12235601\",\"name\":\"고양이 사료 테스트\"}]}","fwb":"1974FD3cvdMsebsbpmSZFtK.1720764497469","ui":"{\"nac\":\"s3aQBMAedgpJB\"}","ext":"{\"wot\":377.5}"}
```

![04_tsg_02-03-00-01]({{"/assets/img/04_tsg_02-03-00-01.png"| relative_url}})

##### (2) 버튼 클릭으로 전환이 발생하는 경우
버튼 클릭시 페이지 로딩이 되지 않으므로, 버튼 클릭 전환에 의한 전환로그 1개만 발생합니다. (PV는 발생하지 않습니다.)

전환로그는 `Type 1번` 과 같은 형태일 수도 있고, `Type 2번` 과 같은 형태일 수도 있습니다

장바구니 버튼 클릭시 전환이 발생하는 경우 예시) `Type 1번`
```
POST https://wcs.naver.com/b HTTP/1.1
Host: wcs.naver.com
Connection: keep-alive
Content-Length: 505
sec-ch-ua: "Not/A)Brand";v="8", "Chromium";v="126", "Google Chrome";v="126"
sec-ch-ua-platform: "Windows"
sec-ch-ua-mobile: ?0
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/126.0.0.0 Safari/537.36
Content-Type: text/plain;charset=UTF-8
Accept: */*
Origin: http://gradion7.www267.freesell.co.kr
Sec-Fetch-Site: cross-site
Sec-Fetch-Mode: no-cors
Sec-Fetch-Dest: empty
Referer: http://gradion7.www267.freesell.co.kr/
Accept-Encoding: gzip, deflate, br, zstd
Accept-Language: ko-KR,ko;q=0.9,en-US;q=0.8,en;q=0.7,ja;q=0.6
Cookie: ASID=0a1924b30000018b33ea9f2a00008e9c; buid=DSC5wraurNnKpdy8r7Sg64j54w; BUID=BRCdtrSSEK_Z-thTqjEAp1WaDQ; NWB=25eb1f3500203b16c7d92f9337cde4fb.1719127593463; NNB=KG346LE72R3WM; BUC=UnkOGSf9sRN7fM2a_lD0b8Sp69wSE2GP04vRQF6O-BE=

{"wa":"s_305c16ba63b1","e":"http://gradion7.www267.freesell.co.kr/shop/shopdetail.html?branduid=12235601&search=&xcode=001&mcode=001&scode=&special=2&GfDT=bml0W1w%3D","u":"http://gradion7.www267.freesell.co.kr/html/log_basket.html","vs":"0.8.13","nt":1721280330988,"t":"conv","trans":"{\"type\":\"add_to_cart\",\"items\":[{\"id\":\"12235601\",\"name\":\"고양이 사료 테스트\"}]}","fwb":"1974FD3cvdMsebsbpmSZFtK.1720764497469","ui":"{\"nac\":\"s3aQBMAedgpJB\"}","ext":"{\"wot\":43.900000005960464}"}
```
![04_tsg_02-03-00-02]({{"/assets/img/04_tsg_02-03-00-02.png"| relative_url}})

또는 

장바구니 버튼 클릭시 전환이 발생하는 경우 예시) `Type 2번`
```
POST https://wcs.naver.com/b HTTP/1.1
Host: wcs.naver.com
Connection: keep-alive
Content-Length: 1047
sec-ch-ua: "Not/A)Brand";v="8", "Chromium";v="126", "Google Chrome";v="126"
sec-ch-ua-platform: "Windows"
sec-ch-ua-mobile: ?0
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/126.0.0.0 Safari/537.36
Content-Type: text/plain;charset=UTF-8
Accept: */*
Origin: http://gradion7.www267.freesell.co.kr
Sec-Fetch-Site: cross-site
Sec-Fetch-Mode: no-cors
Sec-Fetch-Dest: empty
Referer: http://gradion7.www267.freesell.co.kr/
Accept-Encoding: gzip, deflate, br, zstd
Accept-Language: ko-KR,ko;q=0.9,en-US;q=0.8,en;q=0.7,ja;q=0.6
Cookie: ASID=0a1924b30000018b33ea9f2a00008e9c; buid=DSC5wraurNnKpdy8r7Sg64j54w; BUID=BRCdtrSSEK_Z-thTqjEAp1WaDQ; NWB=25eb1f3500203b16c7d92f9337cde4fb.1719127593463; NNB=KG346LE72R3WM; BUC=UnkOGSf9sRN7fM2a_lD0b8Sp69wSE2GP04vRQF6O-BE=

{"wa":"s_305c16ba63b1","e":"http://gradion7.www267.freesell.co.kr/shop/shopdetail.html?branduid=12235601&search=&xcode=001&mcode=001&scode=&special=2&GfDT=bmx1W1w%3D","u":"http://gradion7.www267.freesell.co.kr/html/log_basket.html","vs":"0.8.13","nt":1721280500248,"t":"conv","trans":"{\"type\":\"add_to_cart\",\"items\":[{\"id\":\"12235601\",\"name\":\"고양이 사료 테스트\"}],\"ai\":{\"sa\":{\"ci\":\"0za0003w4Ivz9giLF1oB\",\"t\":\"1721280493\",\"u\":\"http%3A%2F%2Fgradion7.www267.freesell.co.kr%2Findex.html%3FNaPm%3Dct%253Dltfg01cg%257Cci%253D0za0003w4Ivz9giLF1oB%257Ctr%253Dsa%257Chk%253Dd60cbcba879cef5c2d2213ba59dea77a59c267fa\"},\"gfa\":{\"ci\":\"0zi0002dlkLADH6a8vn0\",\"t\":\"1721280451\",\"u\":\"http%3A%2F%2Fgradion7.www267.freesell.co.kr%2Findex.html%3FNaPm%3Dct%253Dlxettf2w%257Cci%253D0zi0002dlkLADH6a8vn0%257Ctr%253Dgfa%257Chk%253D929fb9328d4f344e1cf4bf164086084372d8025e%257Cnacn%253DfRhxCYgPqj2yA\"}}}","fwb":"1974FD3cvdMsebsbpmSZFtK.1720764497469","ui":"{\"nac\":\"s3aQBMAedgpJB\"}","ext":"{\"wot\":42.599999994039536}"}
```

![04_tsg_02-03-00-03]({{"/assets/img/04_tsg_02-03-00-03.png"| relative_url}})

##### (3) 구매완료 페이지의 경우
페이지에서 전환이 발생하므로, PV로그 1개와 구매완료(purchase) 전환로그 1개로 총 2개의 로그가 발생합니다 
구매완료의 경우, 다른 전환과 다르게, 여러가지 Property(conv.id, item(상품)정보, conv.value)가 포함되는 경우가 많습니다.

구매완료 페이지에서 구매완료 전환이 발생하는 경우 예시)

PV로그
```
POST https://wcs.naver.com/b HTTP/1.1
Host: wcs.naver.com
Connection: keep-alive
Content-Length: 542
sec-ch-ua: "Not/A)Brand";v="8", "Chromium";v="126", "Google Chrome";v="126"
sec-ch-ua-platform: "Windows"
sec-ch-ua-mobile: ?0
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/126.0.0.0 Safari/537.36
Content-Type: text/plain;charset=UTF-8
Accept: */*
Origin: http://gradion7.www267.freesell.co.kr
Sec-Fetch-Site: cross-site
Sec-Fetch-Mode: no-cors
Sec-Fetch-Dest: empty
Referer: http://gradion7.www267.freesell.co.kr/shop/orderend.html?ordernum=20240718143006-15739132451&paymethod=B&card_flag=
Accept-Encoding: gzip, deflate, br, zstd
Accept-Language: ko-KR,ko;q=0.9,en-US;q=0.8,en;q=0.7,ja;q=0.6
Cookie: ASID=0a1924b30000018b33ea9f2a00008e9c; buid=DSC5wraurNnKpdy8r7Sg64j54w; BUID=BRCdtrSSEK_Z-thTqjEAp1WaDQ; NWB=25eb1f3500203b16c7d92f9337cde4fb.1719127593463; NNB=KG346LE72R3WM; BUC=UnkOGSf9sRN7fM2a_lD0b8Sp69wSE2GP04vRQF6O-BE=

{"wa":"s_305c16ba63b1","u":"http://gradion7.www267.freesell.co.kr/shop/orderend.html?ordernum=20240718143006-15739132451&paymethod=B&card_flag=","e":"http://gradion7.www267.freesell.co.kr/shop/orderin.html","bt":"1721280589","os":"Win32","ln":"ko-KR","sr":"2560x1440","bw":1229,"bh":1014,"c":24,"j":"N","jv":"1.8","k":"Y","ct":"","cs":"EUC-KR","tl":"%EC%A3%BC%EB%AC%B8%20%EC%99%84%EB%A3%8C","vs":"0.8.13","nt":1721280605916,"fwb":"1974FD3cvdMsebsbpmSZFtK.1720764497469","ui":"{\"nac\":\"s3aQBMAedgpJB\"}","ext":"{\"wot\":114.59999999403954}"}
```

전환로그는 `Type 1번` 과 같은 형태일 수도 있고, `Type 2번` 과 같은 형태일 수도 있습니다

구매완료 (`purchase`)전환로그 예시) `Type 1번`

```
POST https://wcs.naver.com/b HTTP/1.1
Host: wcs.naver.com
Connection: keep-alive
Content-Length: 571
sec-ch-ua: "Not/A)Brand";v="8", "Chromium";v="126", "Google Chrome";v="126"
sec-ch-ua-platform: "Android"
sec-ch-ua-mobile: ?1
User-Agent: Mozilla/5.0 (Linux; Android 13; SM-G981B) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/116.0.0.0 Mobile Safari/537.36
Content-Type: text/plain;charset=UTF-8
Accept: */*
Origin: http://gradion7.www267.freesell.co.kr
Sec-Fetch-Site: cross-site
Sec-Fetch-Mode: no-cors
Sec-Fetch-Dest: empty
Referer: http://gradion7.www267.freesell.co.kr/m/order_complete.html?ordernum=20240718143006-15739132451&paymethod=B&card_flag=
Accept-Encoding: gzip, deflate, br, zstd
Accept-Language: ko-KR,ko;q=0.9,en-US;q=0.8,en;q=0.7,ja;q=0.6
Cookie: ASID=0a1924b30000018b33ea9f2a00008e9c; buid=DSC5wraurNnKpdy8r7Sg64j54w; BUID=BRCdtrSSEK_Z-thTqjEAp1WaDQ; NWB=25eb1f3500203b16c7d92f9337cde4fb.1719127593463; NNB=KG346LE72R3WM; BUC=UnkOGSf9sRN7fM2a_lD0b8Sp69wSE2GP04vRQF6O-BE=

{"wa":"s_305c16ba63b1","e":"http://gradion7.www267.freesell.co.kr/shop/orderin.html","u":"http://gradion7.www267.freesell.co.kr/m/order_complete.html?ordernum=20240718143006-15739132451&paymethod=B&card_flag=","vs":"0.8.13","nt":1721280868009,"t":"conv","trans":"{\"type\":\"purchase\",\"id\":\"20240718143006-15739132451\",\"value\":\"4500\",\"items\":[{\"id\":\"12235601\",\"name\":\"고양이 사료 테스트\",\"quantity\":3,\"payAmount\":4500}]}","fwb":"1974FD3cvdMsebsbpmSZFtK.1720764497469","ui":"{\"nac\":\"s3aQBMAedgpJB\"}","ext":"{\"wot\":348.90000000596046}"}
```
![04_tsg_02-03-00-04]({{"/assets/img/04_tsg_02-03-00-04.png"| relative_url}})

구매완료 (`purchase`)전환로그 예시) `Type 2번`

```
POST https://wcs.naver.com/b HTTP/1.1
Host: wcs.naver.com
Connection: keep-alive
Content-Length: 1109
sec-ch-ua: "Not/A)Brand";v="8", "Chromium";v="126", "Google Chrome";v="126"
sec-ch-ua-platform: "Windows"
sec-ch-ua-mobile: ?0
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/126.0.0.0 Safari/537.36
Content-Type: text/plain;charset=UTF-8
Accept: */*
Origin: http://gradion7.www267.freesell.co.kr
Sec-Fetch-Site: cross-site
Sec-Fetch-Mode: no-cors
Sec-Fetch-Dest: empty
Referer: http://gradion7.www267.freesell.co.kr/shop/orderend.html?ordernum=20240718143006-15739132451&paymethod=B&card_flag=
Accept-Encoding: gzip, deflate, br, zstd
Accept-Language: ko-KR,ko;q=0.9,en-US;q=0.8,en;q=0.7,ja;q=0.6
Cookie: ASID=0a1924b30000018b33ea9f2a00008e9c; buid=DSC5wraurNnKpdy8r7Sg64j54w; BUID=BRCdtrSSEK_Z-thTqjEAp1WaDQ; NWB=25eb1f3500203b16c7d92f9337cde4fb.1719127593463; NNB=KG346LE72R3WM; BUC=UnkOGSf9sRN7fM2a_lD0b8Sp69wSE2GP04vRQF6O-BE=

{"wa":"s_305c16ba63b1","e":"http://gradion7.www267.freesell.co.kr/shop/orderin.html","u":"http://gradion7.www267.freesell.co.kr/shop/orderend.html?ordernum=20240718143006-15739132451&paymethod=B&card_flag=","vs":"0.8.13","nt":1721280606043,"t":"conv","trans":"{\"type\":\"purchase\",\"id\":\"20240718143006-15739132451\",\"value\":\"4500\",\"items\":[{\"id\":\"12235601\",\"name\":\"고양이 사료 테스트\",\"quantity\":3,\"payAmount\":4500}],\"ai\":{\"sa\":{\"ci\":\"0za0003w4Ivz9giLF1oB\",\"t\":\"1721280493\",\"u\":\"http%3A%2F%2Fgradion7.www267.freesell.co.kr%2Findex.html%3FNaPm%3Dct%253Dltfg01cg%257Cci%253D0za0003w4Ivz9giLF1oB%257Ctr%253Dsa%257Chk%253Dd60cbcba879cef5c2d2213ba59dea77a59c267fa\"},\"gfa\":{\"ci\":\"0zi0002dlkLADH6a8vn0\",\"t\":\"1721280451\",\"u\":\"http%3A%2F%2Fgradion7.www267.freesell.co.kr%2Findex.html%3FNaPm%3Dct%253Dlxettf2w%257Cci%253D0zi0002dlkLADH6a8vn0%257Ctr%253Dgfa%257Chk%253D929fb9328d4f344e1cf4bf164086084372d8025e%257Cnacn%253DfRhxCYgPqj2yA\"}}}","fwb":"1974FD3cvdMsebsbpmSZFtK.1720764497469","ui":"{\"nac\":\"s3aQBMAedgpJB\"}","ext":"{\"wot\":240.2999999821186}"}
```

![04_tsg_02-03-00-05]({{"/assets/img/04_tsg_02-03-00-05.png"| relative_url}})

Version: 20240718_01







