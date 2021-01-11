---
title: "문자 인코딩 (ASCII, EUC-KR, UTF-8, UTF-16, ...)"
categories:
  - web
tags:
  - web
---

#### 1. ASCII 코드
ASCII(American Standard Code for Information Interchange)는 1967년 처음 개정된 대표적인 문자 인코딩이다.  
아스키는 7비트 인코딩이며 33개의 출력 불가능한 제어 문자, 95개의 출력 가능한 문자(영문 대소문자, 숫자, 특수문자)를 포함하여 총 128개로 구성된다.  

아스키가 널리 사용되면서 아스키를 기반으로 한 여러 확장 인코딩이 등장했는데, 이를 묶어서 아스키라고 부르기도 한다.  
여기에는 기존 7비트 인코딩을 유지한 ISO/IEC 646, 7비트 앞에 0을 추가해 8비트 인코딩을 만든 IBM 코드 페이지, ISO 8859가 있다.  
보통 ASCII 코드라고 하면 ISO 8859라고 생각하면 된다. 

##### ASCII Table
http://www.asciitable.com/
아스키는 16진수 기준으로 0x00 ~ 0x7F 까지 128개의 값을 통해 문자를 표현한다.   
여기서 각 숫자가 아스키의 어떤 문자에 해당하는 지를 정리한 표를 ASCII Table이라고 한다.  

![ASCII Table](http://www.asciitable.com/index/asciifull.gif)

#### 2. EUC-KR
대소문자를 합쳐도 52자밖에 되지 않는 영문에 비해 한자나 한글같이 표현해야 할 문자 수가 많은 언어는 8비트라는 공간이 턱없이 부족했다.  
따라서 16비트로 문자를 표시하는 인코딩 방식이 등장하기 시작했다.  
이 중에서도 EUC(Extended Unix Code)는 한국어, 중국어, 일본어 문자 표현에 사용되는 ISO 2022 표준에 기반한 8비트 문자 인코딩 방식이다.  
동아시아 언어를 표현하기 위해 만들어진 인코딩 방식으로 94개의 값을 2바이트씩 붙여 사용한다.  

##### EUC-KR 구성
EUC-KR 인코딩은 완성형이라고도 불리며 KS X 1001과 KS X 1003로 구성된다.  
  - KS X 1003 : 0x00 ~ 0x7F 까지가 해당되며 1바이트로 표현된다.  
    - 역슬래시(0x5C) 자리에 원화 기호(₩, U+20A9)가 들어간 것을 빼면 ASCII와 동일하다.  
    - (대부분의 시스템이 ASCII와의 호환서을 위해 원화 기호를 역슬래시로 처리하고 있어 똑같다고 봐도 무방하다.)
  - KS X 1001 : KS X 1003 과 겹치지 않는 0xA1 ~ 0xFE 까지의 값을 연달아 사용해 2바이트로 표현한다.   

![EUC-KR Table](https://upload.wikimedia.org/wikipedia/commons/5/5e/Euckr-map.png)

두 개의 구성이 완벽하게 분리되어 있어 첫 비트가 0이면 아스키와 동일하게 1바이트로, 첫 비트가 1이면 2바이트로 읽으면 된다는 명확한 규칙이 있다.

##### EUC-KR의 문제점 
다만 EUC-KR은 특수문자나 한자를 함께 배치하면서 한글을 2350자 밖에 할당하지 못했다는 문제가 있다.  
실제 한글의 개수는 11172자인데, EUC-KR에서는 이 중에서 자주 사용되는 문자를 제외한 대부분의 문자가 생략되었다.  

#### 3. CP949
모든 한글을 표현하지 못한다는 EUC-KR의 문제를 해결하기 위해 다양한 인코딩 방식이 등장했고, 그 중에서도 국내에 널리 보급된 것은 CP949이다.  
CP949는 Microsoft에서 Windows 95에 확장 완성형(UHC, Unified Hangul Codeset) 또는 통합현 한글 코드라는 명칭으로 처음 도입되었다.  
이후 Windows가 한국 OS를 장악하면서 현재는 EUC-KR을 거의 대체하였으며, 대부분의 브라우저에서 EUC-KR로 표시되는 인코딩 옵션은 사실 CP949를 가리킨다.  
CP949는 확장 완성형이라는 이름에서 보여지는 것 같이 EUC-KR을 확장한 형태이다.  
기본적으로는 EUC-KR을 그대로 가져오고, EUC-KR에서 사용되지 않던 영역에 나머지 8822 글자를 가나다 순으로 채워넣었다.  
EUC-KR의 KS X 1001 영역과 마찬가지로 2바이트로 표현되며, 다음 규칙에 의해 구성된다.  

##### CP949 확장
 - 첫 번째 바이트는 0x81 ~ 0xC6 범위로 구성된다.
 - 두 번째 바이트는 0x41 ~ 0x5A(로마자 대문자), 0x61 ~ 0x7A(로마자 소문자), 0x81 ~ 0xFE 범위로 구성된다.
 - 단 첫 번째 바이트가 0xA1 이상인 경우 두 번째 바이트는 0xA1 미만으로 사용한다.
    - 첫 번째 바이트와 두 번째 바이트 모두 0xA1 이상인 영역은 이미 EUC-KR에서 사용하는 영역 

##### CP949 Table
CP949는 문자 코드표를 5개의 장으로 분류하며 이 중에서 3개는 EUC-KR과 동일한 코드표로 구성된다.  
 - 2350 Hangul Syllables : 한글이 배정된 영역
 - Special Symbols : 특수문자 대상 영역
 - Chinese Characters : 한자 대상 영역
CP949에서 추가된 영역은 Additional Hangul Syllable Set로 표시되어 있다.  
 - Additional Hangul Syllable Set 1 : 첫 번째 바이트가 0xA1 보다 작은 영역
 - Additional Hangul Syllable Set 2 : 첫 번째 바이트가 0xA1 이상인 영역

![CP949 Table](https://upload.wikimedia.org/wikipedia/commons/5/54/Cp949-map.png)

##### CP949의 문제점
앞서 말했던 것 같이 EUC-KR은 1바이트로 표현되는 문자와 2바이트로 표현되는 문자가 명확한 규칙에 의해 구분되어 있었다.  
그러나 CP949는 남는 영역에 나머지 문자들을 배치하면서 이런 규칙이 사라져 버렸다.  
읽는 위치에 따라 문자가 완전히 달라질 수도 있으며, 몇 글자가 소실되는 경우 이후의 문자를 모두 읽을 수 없게 될 수도 있다.
또한 한글 가나다 순서와 16진수 값의 순서가 전혀 달라서 가나다순 정렬 시에도 의도하지 않는 결과가 나올 수 있다.  
ex) 가(0xB0A1) 보다 롹(0x8EF4)가 먼저다.  

#### 4. 유니코드
EUC-KR과 CP949는 한글을 위해 만들어진 인코딩이다.  
마찬가지로 영어를 사용하는 나라를 제외한 나라에서는 자신의 문자를 표현하기 위해 다른 인코딩을 사용할 것이다.  
이렇게 문자 별로 다른 인코딩을 사용함으로써 동일한 문서에 두 종류 이상의 문자를 표현할 수 없고, 특정 문자에 종속된 인코딩으로 작성된 문서는 다른 인코딩으로 열어볼 수 없다는 문제점이 생겼다.  

그리하여 전세계의 모든 문자를 일관되게 표현할 수 있도록 설계된 유니코드가 1991년에 발표되었다.  
초기에 0x0000 부터 0xFFFF까지 65536개의 문자를 부여하였으나 이 중 절반 이상이 한자에 배정되면서 다른 문자를 넣을 자리가 부족해졌다.  
이후 0x0000 부터 0xFFFF 까지의 세트를 17개로 늘린다.  
이 세트를 평면(Plane)이라고 부르고 0번부터 16번까지 번호를 붙였다.  
현재는 0~2번, 14~16번 6개의 평면을 사용하며, 3~13번 까지의 11개 평면은 미래를 위해 비워놓은 상태이다.  

평면들 중 0번에는 현재 전세계에서 쓰이는 모든 문자를 할당했는데, 이것을 기본 다국어 평면(BMP, Basic Multilingual Plane)이라고 한다.  
그 외에 1번 평면은 보조 다국어 평면(SMP, Supplementary Multilingual Plane), 2번 평면은 보조 상형 문자 평면(SIP, Supplementary Ideographic Plane)이라고 한다.  

17개의 평면이 존재하면 각 평면마다 동일한 0x0000 ~ 0xFFFF 값을 가지고 있기 때문에 숫자만 가지고는 어떤 문자를 의미하는 지 알 수 없다.  
따라서 유니코드는 2바이트 앞에 ``U+(평면번호)``를 추가하기로 한다. 단, 기본 다국어 평면의 경우 평면번호를 생략한다.  
 - ex) U+B000 -> 0번 기본 다국어 평면에 포함
 - ex) U+10B000 -> 16번 평면에 포함

##### 유니코드의 문제점
평면의 번호를 추가로 붙여야 하기 때문에 기존 2바이트로 표현할 수 없는 상황이 되었다.  
또한 CP949와 동일한 이유로 읽는 위치에 따라 다른 글자로 해석될 여지가 있다.   

#### 5. UCS-2
UCS는 Universal Character Set의 줄임말이며 2는 해당 인코딩이 2바이트 인코딩이라는 것을 의미한다.   
UCS-2는 유니코드의 평면 중 기본 다국어 평면만 사용하는 인코딩이다.
하나의 평면만을 사용하기 때문에 평면을 표시하는 ``U+``는 제거하여 2바이트로 문자를 표현한다.   
당연히 모든 값들은 기본 다국어 평면과 동일하다.  

##### HTML Entity Number
HTML Entity Number는 UCS-2 방식의 16진수 코드를 10진수로 변환하여 앞에는 &#, 뒤에는 ;를 붙여서 사용한다.  

##### UCS-2의 한계
다만 UCS-2도 유니코드의 한 평면만을 선택하여 사용하는 방식으로 유니코드를 완전히 활용할 수는 없다는 한계가 있다.  

#### 6. UTF-32
UTF-32는 유니코드의 모든 문자를 표현할 수 있도록 한 글자당 32비트를 사용하는 인코딩이며 UCS-4라고도 불린다.  
앞의 2바이트는 0x00 ~ 0x10 까지를 할당하여 몇 번째 평면인 지를 표시하고 뒤의 2바이트는 문자를 표현한다.  

##### UTF-32의 단점
한 글자가 고정으로 4바이트를 차지하므로 UTF-32 인코딩을 사용하는 문서는 불필요하게 크기가 커지게 된다. 
또한 CP949와 같은 이유로 문자열 처리가 까다롭다는 단점이 있다.  
UTF-32는 거의 사용되지 않으며 해당 인코딩이 지원되는 에디터도 많지 않다.  

#### 7. UTF-16
UTF-16은 인코딩 규칙은 크게 기본 다국어 평면 내 문자와 그 외의 문자로 구분하여 UTF-32의 단점을 해결한 인코딩이다.  
기본 다국어 평면에 속한 문자는 평면 번호 U+를 제외한 2바이트 유니코드 값을 그대로 사용한다.  
단, 인코딩된 바이트 스트링의 엔디언을 주의해야 한다.
 - UTF16 문자가 바이트 기준으로 ``yyyyyyyy xxxxxxxx`` 라고 가정
 - UTF-16 BE 코드는 ``yyyyyyyy xxxxxxxx`` 이다.
    - Big-endian : 바이트 배열에 최상위 바이트부터 저장하는 방식
 - UTF-16 LE 코드는 ``xxxxxxxx yyyyyyyy`` 이다.
    - Little-endian : 바이트 배열에 최하위 바이트부터 저장하는 방식

##### 기본 다국어 평면 외 문자 변환 방법
기본 다국어 평면 외의 문자는 조금 복잡한 규칙이 적용된다.  
2바이트로 값을 표현할 수 없는 기본 다국어 평면 외 문자는 서러게이트(Surrogate) 문자 영역에 해당하는 두 개의 16비트 문자로 변환된다.

![UTF-16 문자 변환](/image/utf16_surrogate.png)
 - High Surrogate
   - 앞의 110110 값은 고정이다.
   - ZZZZ 는 zzzzz -1의 값이다.
   - 마지막에는 xxxxxx 값이 그대로 온다.
 - Low Surrogate
   - 앞의 110111 값은 고정이다.
   - 뒤에는 yyyyyyyyyy 값이 그대로 온다.

##### Surrogate 영역
유니코드의 기본 다국어 평면에는 문자가 배정되지 않은 영역이 2군데 있다.
 - High Surrogate 문자 영역
    - 앞의 여섯 비트가 110110 인 영역 (U+D800부터 U+DB7F)
 - Low Surrogate 문자 영역
    - 앞의 여섯 비트가 110111 인 영역 (U+DC00부터 U+DFFF)  

따라서 기본 다국어 평면 외 문자는 해당 Surrogate에 배치하여 사용할 수 있는 것이다.  


#### 8. UTF-8
UTF-8은 ASCII의 하위 호환성을 보장하는 문자 인코딩 방식이다.
크게 4가지 영역으로 구분되며, 규칙이 적용되는 영역에 따라 16진수의 자리수가 다르다는 특징이 있다.

##### U+0000 ~ U+007F
ASCII와 같은 영역인 U+0000 ~ U+007F는 1바이트 값을 그대로 사용한다.
ASCII와 동일하게 이 영역의 값은첫 번째 bit가 0이라는 특징을 가지고 있다.
 
##### U+0080 ~ U+07FF
첫 바이트는 110으로 시작하고, 나머지 바이트들은 10으로 시작하는 2바이트 구성이다.
UTF-16(BE)으로 ``00000xxx xxxxxxxx`` 인 문자의 경우 UTF-8로 ``110xxxxx 10xxxxxx``이 된다.

##### U+0800 ~ U+FFFF
첫 바이트는 1110으로 시작하고, 나머지 바이트들은 10으로 시작하는 3바이트 구성이다.
UTF-16(BE)으로 ``xxxxxxxx xxxxxxxx`` 인 문자의 경우 UTF-8로 ``1110xxxx 10xxxxxx 10xxxxxx``이 된다.

##### U+10000 ~ U+10FFFF
유니코드의 기본 다국어 평면 외 문자의 경우 4바이트로 표현된다.
 - UTF-16(BE) : 110110ZZ ZZxxxxxx 110111xx xxxxxxxx
 - UTF-8 : 11110zzz 10zzxxx 10xxxxxx 10xxxxxx
   - 기존의 UTF-32에서 UTF-16으로 변환했던 것 처럼 ZZZZ = zzzzz-1 이다.

##### UTF-8의 규칙
이상 위 특징으로 UTF-8 문자는 앞의 몇 비트를 확인하여 각 바이트의 성격을 확인할 수 있으며, 어떤 문자가 정상인지 어디서부터 유실되었는지도 유추할 수 있다.
 - 어떤 바이트가 0으로 시작하면 해당 바이트는 ASCII와 같은 글자
 - 어떤 바이트가 110으로 시작하면 해당 바이트부터 2바이트가 한 글자
 - 어떤 바이트가 1110으로 시작하면 해당 바이트부터 3바이트가 한 글자
 - 어떤 바이트가 11110으로 시작하면 해당 바이트부터 4바이트가 한 글자
 - 어떤 바이트가 10으로 시작하면 해당 바이트는 어떤 글자의 중간 바이트