# old  shopmall (2021) 모듈 정리

1. 세트상품, 초이스 , 크로스셀 주문시
    - 세트상품, 초이스 상품 단품생성 필요
    - 상품구성품, 상품추가구성품 테이블 조회 필요
    - 세트, 초이스의 경우 주문가격에 메인상품가격적용, 하위상품에 상품구성품 할인가격 반영 (세트, 초이스 상품은 실물없음)
    - 크로스셀의 경우 하위상품의 단품가격 조회 후 상품추가구성품 테이블의 할인가격 적용  (크로스셀 메인상품은 일반상품)
    - 배송비 계산시 세트, 초이스는 메인상품만 계산, 크로스셀은 하위상품 배송비 계산에 포함 (상품 구성시 제약조건 필요함).

2. 세트상품, 초이스 , 크로스셀 상품유효성 검증
    - 판매기간 확인

3. 세트상품 재고관리
    - 단품 재고 전부 확인 필요


## 주문공통모듈
1. 상품정보조회 OdCmmnService.getOrdGdsInfo
    > param odOrdGdsDto 주문상품DTO    
    > return 주문상품 정보설정
    1. 단품, 상품, 상품/단품가격변경, 상품이미지 조회 후 세팅
2. 구성상품조회 OdCmmnService.getOrdPcgprdInfo
    > param gdsTypCd 상품유형(세트, 초이스, 크로스셀)    
    > param mainGdsId 메인상품ID    
    > param odOrdPcgprdDto 구성품 DTO    
    1. 단품, 상품, 상품/단품가격변경
    2. 상품구분코드
        - 상품구분코드 세트/초이스: 상품구성품 확인 후 세팅
        - 그외: 상품추가구성품 조회 후 세팅
3. 재고관리 (차감, 복구) OdCmmnService.saveOrdStck
    > param orderDto 주문DTO
    1. 주문상품조회
        1. 주문상태코드 확인
            - "00"으로 끝날 시: 주문수량 get
            - 아닐 시: 취소수량 get
        2. 단품 재고 저장 saveUntStokQty(단품ID, 재고수량)
            - 남은 재고수량이 0보다 클 경우 해당 단품Id로 찾은 데이터의 재고수량 = 재고수량 - param재고수량
4. 결제데이터 생성 OdCmmnService.insertOdStlm
    > param orderDto 주문DTO
    1. 결제내역에서
        1. 결제그룹Id 1로 하드코딩
        2. 결제일시 = 현재일시
        3. 주문반품구분코드: 주문(100)
        4. 결제예정일시 = 현재일시
        5. 환불예정금액 = 0
        6. 환불예정금액 = 0
        7. 결제예정금액 = 결제금액
        8. 환불가능금액 = 결제금액
5. 환불데이터 생성 OdCmmnService.saveOrdRfnd
    > param ordId 주문id    
    > param mainRfndMnsId 주환불수단    
    > param rfndAmt 환불금액    
    > param ordRtnTypCd 주문반품구분코드(주문:100 , 반품:200, 교환:300    
    > param rtnGrpId 반품그룹코드 (반품이 아닐경우 0 )    
    > param ExnGrpId 교환그룹코드 (교환이 아닐경우 0 )    
    1. 결제 내역 조회
        - 없을 시 exception
    2. 반품상태코드 확인
        - 반품상태코드 반품(200)일 시 환불관련코드 반품(110)
        - 아닐 시 환불관련코드 주문(100)
    3. 결제내역에서
        1. 환불가능금액
        2. 환불제외금액 = 환불할 금액
        3. 환불가능금액 > 환불할 금액 일 경우
            - 환불가능금액 = 환불가능금액 - 환불할 금액, 세팅
            - 환불제외금액 = 환불예정금액 + 환불할 금액, 세팅
            - 환불제외금액 = 환불제외금액 - 환불할 금액
        4. 그외의 경우
            - 환불가능금액 = 0 세팅
            - 환불제외금액 = 환불예정금액 + 환불가능금액 세팅
            - 환불제외금액 = 환불제외금액 - 환불가능금액
        5. 결제내역 저장
        6. 환불데이터 생성
            - 반품그룹ID 없으면  0
            - 교환그룹ID 없으면 0
            - 환불예정금액 0 (0으로 덮어씌우고 있음)
            - 환불금액 0 
            - 환불이력에서 max 환불그룹ID 찾아서 + 1, 환불그룹ID 세팅
            - 환불수단정보 검증
                - 환불수단 조회 후 없을시 exception
            - 환불수단코드, 환불수단ID 세팅
            - 환불 후처리ID인 환불수단관련ID 세팅
            - 환불수단관련코드 세팅
            - 환불예정일시 = 결제예상일시 세팅
        7. 환불데이터 저장
        8. 결제제외금액 == 0 일 경우 break
6. 원주문 조회 OdCmmnService.getOrgOrdGdsInfo
    > param odOrdPrdDto 대상 상품 리스트
    1. 주문상품 조회 (단건)
        - 없을 시 exception
    2. 주문수량 - 반품수량 - 반품접수수량 - 취소접수수량 < 0 일시 exception
    3. 원주문 세팅

## 주문모듈
1. 주문접수 OdOrdService.rcptOrd
2. 주문취소 OdOrdService.rtrcnOrd

## 교환모듈
1. 교환 OdExnService.exnOrd

## 반품모듈
1. 반품 OdRtnService.rtnOrd
2. 반품승인 OdRtnService.aprvRtnOrd

## 환불모듈
1. 환불승인 OdRfndService.aprvRfnd
    - param apvRefundDto 환불승인DTO
2. 환불처리 OdRfndService.saveRfnd
    - param odRfn 환불대상
    - 환불대상 ID를 받아 환불처리 
        1. 환불금액 수정
        2. 원결제 환불금액 수정
        3. 환불수단 관련 처리


## 배송비모듈
1. 주문 배송비 계산
    1. 배송비계산 CalcDlvfService.calcOrdDlvf
    2. 현재배송정책 찾기 CalcDlvfService.findCurrDlvfPolic
    3. 장바구니배송비계산 CalcDlvfService.calcCartDlvf
2. 취소 배송비 계산
    1. 취소 배송비 계산 CalcDlvfService.calcRtrcnOrdDlvf
    2. 환불배송비계산 CalcDlvfService.calcRfndDlvf
        - param List<OdOrdGdsDto> dlvfGrpGdsList , OdDlvf dlvf
3. 반품 배송비 계산 CalcDlvfService.calcRtnOrdDlvf
    - 반품배송비의 경우 합포장 하지 않고 단품단위로 계산한다.
4. 반품 배송비 접수 전 반품배송비 안내용 배송비 계산 CalcDlvfService.calcCartRtnOrdDlvf
    - 반품배송비의 경우 합포장 하지 않고 단품단위로 계산한다.