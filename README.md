
# 비대면 개인과외 스토어앱

본 시스템은 MSA/DDD/Event Storming/EDA 를 포괄하는 분석/설계/구현 전단계를 커버하도록 구성한 시스템입니다.

# Table of contents

- [비대면 개인과외 스토어앱](#---)
  - [서비스 시나리오](#서비스-시나리오)
  - [분석/설계](#분석설계)
  - [추가요구사항](#추가요구사항)

# 서비스 시나리오

기능적 요구사항
1. 개인과외 구매고객은 비대면 개인과외를 선택하여 구매를 요청한다.
1. 구매요청 되면 결제시스템에서 결제가 완료된다.
1. 송금시스템에서 판매고객 계좌에 수수료를 제외하고 돈을 송금한다. 
1. 개인과외 판매고객은 비대면 개인과외를 등록하여 판매를 요청한다.
1. 판매승인이 요청되면 승인시스템에서 관리자가 조회한 후 승인한다.
1. 판매요청이 승인되면 강의시스템에 개인과외가 등록된다.
1. 개인과외 구매/판매완료 고객은 강의시스템에서 실시간 화상강의를 진행한다.
1. 개인과외 구매고객은 구매를 취소할 수 있다.
1. 구매가 취소되면 결제가 취소된다.
1. 개인과외 구매고객이  개인과외를 중간 중간 조회한다.
1. 개인과외 판매고객이 자신이 등록한 개인과외 상태를 중간 중간 조회한다.
1. 승인시스템 관리자가 판매 승인요청 대상을 중간 중간 조회한다.

비기능적 요구사항
1. 트랜잭션
    1. 결제가 되지 않은 구매건은 아예 거래가 성립되지 않아야 한다 Sync 호출
1. 장애격리
    1. 결제기능이 수행되지 않더라도 구매 요청은 365일 24시간 받을 수 있어야 한다. Async (event-driven), Eventual Consistency
    1. 과외승인 기능이 수행되지 않더라도 판매 요청은 365일 24시간 받을 수 있어야 한다. Async (event-driven), Eventual Consistency
1. 성능
    1. 구매고객이 판매대상 개인과외를 구매시스템(프론트엔드)에서 확인할 수 있어야 한다. CQRS
    1. 판매고객이 등록한 개인과외건 판매요청 상태를 판매등록시스템(프론트엔드)에서 확인할 수 있어야 한다. CQRS
    1. 승인관리자가 판매요청목록을 승인시스템(프론트엔드)에서 확인할 수 있어야 한다. CQRS

# 분석/설계

### 이벤트 도출

![캡처1](https://user-images.githubusercontent.com/63624014/81873992-e1ea3d80-95b7-11ea-81af-82dd79422780.PNG)


### 부적격 이벤트 탈락

![캡처2](https://user-images.githubusercontent.com/63624014/81874011-edd5ff80-95b7-11ea-9e7e-cad6d2afe518.png)

    - 과정중 도출된 잘못된 도메인 이벤트들을 걸러내는 작업을 수행함
      . 등록과외 조회됨, 승인요청 조회됨 제외 : UI 이벤트 (업무적인 의미의 이벤트 아님)
      . 강의진행됨 제외 : 다른 마이크로서비스에서 받을 필요 없는 

### 액터, 커맨드, 폴리시 부착

![캡처3](https://user-images.githubusercontent.com/63624014/81874027-fb8b8500-95b7-11ea-8c48-4ade5435d0cd.PNG)


### 어그리게잇으로 묶기

![캡처4](https://user-images.githubusercontent.com/63624014/81874048-06deb080-95b8-11ea-800a-1795ed782370.PNG)


### 바운디드 컨텍스트로 묶기

![캡처5](https://user-images.githubusercontent.com/63624014/81874059-10681880-95b8-11ea-96fa-21d7fc2d93e6.PNG)

    - 도메인 서열 분리
      . Core Domain : 구매, 판매등록 
      . Supporting Domain : 판매승인, 과외상품
      . General Domain : 결제


### 폴리시의 이동과 컨텍스트 매핑

![캡처8](https://user-images.githubusercontent.com/63624014/81874124-2f66aa80-95b8-11ea-8e62-1112bdbae86a.PNG)

  - Correlation Key
      . 구매, 결제 : purchaseId
      . 판매등록, 판매승인, 과외관리 : tutorId
      . 결제, 송금 : accountId

### 1차 완성본에 대한 기능적/비기능적 요구사항을 커버하는지 검증

![캡처9](https://user-images.githubusercontent.com/63624014/81874143-368db880-95b8-11ea-88bd-da1049a610be.PNG)

    - 개인과외 구매고객은 비대면 개인과외를 선택하여 구매를 요청한다. (ok)
    - 구매요청 되면 결제시스템에서 결제가 완료된다. (ok)
    - 송금시스템에서 판매고객 계좌에 수수료를 제외하고 돈을 송금한다. (ok)
    - 개인과외 판매고객은 비대면 개인과외를 등록하여 판매를 요청한다 (ok)    
    - 판매승인이 요청되면 승인시스템에서 관리자가 조회한 후 승인한다. (ok)
    - 판매요청이 승인되면 강의시스템에 개인과외가 등록된다. (ok)
    - 개인과외 구매/판매완료 고객은 강의시스템에서 실시간 화상강의를 진행한다. (ok)
    - 개인과외 구매고객은 구매를 취소할 수 있다. (ok)
    - 구매가 취소되면 결제가 취소된다. (ok)
    - 개인과외 구매고객이  개인과외를 중간 중간 조회한다. (ok)
    - 개인과외 판매고객이 자신이 등록한 개인과외 상태를 중간 중간 조회한다. (ok)
    - 승인시스템 관리자가 판매 승인요청 대상을 중간 중간 조회한다.(ok)
 
    - 트랜잭션 (1) 
      . 결제가 되지 않은 구매건은 아예 거래가 성립되지 않아야 한다 Sync 호출 (fail)
    - 장애격리 (2)
      . 결제기능이 수행되지 않더라도 구매 요청은 365일 24시간 받을 수 있어야 한다. (ok)
      . 과외승인 기능이 수행되지 않더라도 판매 요청은 365일 24시간 받을 수 있어야 한다. (ok)
    - 성능 (3)
      . 구매고객이 판매대상 개인과외를 구매시스템(프론트엔드)에서 확인할 수 있어야 한다. [CQRS] (ok)
      . 판매고객이 등록한 개인과외건 판매요청 상태를 판매등록시스템(프론트엔드)에서 확인할 수 있어야 한다. [CQRS] (ok)
      . 승인관리자가 판매요청목록을 승인시스템(프론트엔드)에서 확인할 수 있어야 한다. [CQRS] (ok)

### 수정 모형    

    - 모든 요구사항을 커버함.

![캡처10](https://user-images.githubusercontent.com/63624014/81874156-41484d80-95b8-11ea-9d11-752a76ea2fdc.PNG)

    - 구현을 위해 영어로 표기함.

![캡쳐1](https://user-images.githubusercontent.com/63624014/81885052-75ca0280-95d4-11ea-918a-a2f37d1dcb18.PNG)
    

## 헥사고날 아키텍처 다이어그램 도출

![캡쳐2](https://user-images.githubusercontent.com/63624014/81885065-7ebad400-95d4-11ea-8a0a-576d81528718.PNG)


# 추가요구사항 

## 요구사항

기능적 요구사항
1. 구매고객에게 비대면 개인과외 만족도를 조사한다.

비기능적 요구사항
1. 장애격리
    1. 고객만족도 응답기능이 행되지 않더라도 구매 요청은 365일 24시간 받을 수 있어야 한다. Async (event-driven), Eventual Consistency

## 추가기능 반영모형

![캡쳐3](https://user-images.githubusercontent.com/63624014/81885086-8a0dff80-95d4-11ea-9b49-ebb5870596c1.PNG)

## 추가기능 헥사고날 아키텍처 다이어그램 도출

![캡처4](https://user-images.githubusercontent.com/63624014/81885117-96925800-95d4-11ea-8ac5-4ee287dceafb.PNG)






