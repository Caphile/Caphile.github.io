---
title: Accomodation
layout: default
parent: Crawling
grand_parent: Studies
---

# 숙박시설 정보 크롤링
{: .no_toc }

<br/>

[네이버 호텔](https://hotels.naver.com/)에서 숙박시설 정보를 가져오기 위한 웹 스크래퍼.

전체 코드와 수집한 데이터는 [GitHub](https://github.com/Caphile/myTools/tree/main/crawling/accomodation)에 저장되어 있음.

코드는 2021.09 기준으로 작성, 웹사이트의 데이터 획득 방법이 변경되면서 현재는 동작하지 않음.

---

## 배경

<br/>
국내의 숙박시설 정보가 필요하여 검색을 해봤지만 어디에서도 그런 정보를 찾을 수 없어 OTA 서비스에서 제공하는 정보를 크롤링하여 데이터를 긁어오기로 했다. 다양한 OTA가 존재하지만 국내 숙박시설 정보를 가장 많이 보유한 것처럼 보이며, 지역 기준 검색 기능이 강력한 네이버의 웹서비스를 대상으로 크롤링을 진행했다.

---

## 데이터 수집

<br/>
웹사이트의 스크립트를 뜯어보니 서비스 화면에 뿌려주기 위한 데이터를 특정 url에 request하여 가져오고 있었다. 따라서 파이썬의 requests 라이브러리를 사용하여 데이터를 긁어왔다.

1. 데이터 요청 URL:

   - https://m-hotel.naver.com/hotels/api/hotels?chains=&checkin=&checkout=&destination=place:{place}&features=&guestRatings=&includeLocalTaxesInTotal=false&includeTaxesInTotal=false&maxPrice=&minPrice=&pageIndex={page}&pageSize=100&propertyTypes=&radius=&rooms=&sortDirection=descending&sortField=popularityKR&starRating=&type=pc

2. 데이터 구조:
   - propertyTypes에서 숙박유형, 성급 등의 정보를 가져옴
   - results에서 각 숙소의 정보를 얻음
     - id: 각 숙소의 고유 식별번호
     - key: 각 숙소에 대한 키 값
     - propertyType: 숙소의 유형에 대한 식별번호
     - starRating: 숙소의 성급
     - name: 숙소의 이름
     - place: 숙소의 위치 정보
     - address: 숙소의 주소 정보
     - domestic: 국내 숙소의 추가 정보
       - pinID: 지도에서 사용되는 고유 식별번호

3. 추가 데이터 요청 URL:
   - https://m-hotel.naver.com/hotels/api/hotels/{key}?groupedFeatures=true&type=pc
   - https://map.naver.com/v5/api/sites/summary/{pinID}?lang=ko

4. 엑셀에 저장되는 컬럼 정보:
   - 도·시별 숙박유형, 성급, 이름, 도로명주소, 지번주소, 전화번호

5. 데이터 수집 과정:
   - 주요 URL로부터 각 지역별 숙소 리스트를 가져옴
   - 가져온 리스트에서 각 숙소의 정보를 추출하고, 추가 정보를 위해 추가적인 요청을 보냄
   - 추출한 정보를 엑셀 파일에 저장

---

## 체크 포인트

<br/>
정적 크롤링을 통해 데이터를 가져오면서 처리 시간이 많이 소요되고, 중간에 얻어진 데이터를 안전하게 백업해야 하는 필요성이 있었다. 이러한 상황에서 프로그램이 종료될 때 자동으로 엑셀 파일을 업데이트하는 기능을 atexit 라이브러리를 활용하여 구현했다. 그러나 데이터 크롤링을 다시 시작할 때 중복된 데이터 문제가 발생할 수 있기 때문에 이를 방지하기 위해 삽입정렬과 이진탐색을 활용했다.

삽입정렬을 통해 이미 얻어진 데이터의 식별자(id)를 오름차순으로 정렬된 인덱스에 저장하고, 이진탐색을 활용하여 새로운 데이터의 id가 이미 존재하는지를 로그 시간에 확인한다. 이를 통해 중복된 데이터를 효과적으로 방지하고, 중간에 프로그램이 종료되었을 때에도 안정적으로 중복 처리를 수행할 수 있다. 재시작 시에도 중복을 방지하여 데이터 크롤링이 안정적으로 진행된다.

---

## 결과

<br/>
dsㄴ