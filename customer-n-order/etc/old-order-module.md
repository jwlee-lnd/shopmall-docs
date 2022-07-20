# old  shopmall (2020) 모듈 정리

## 주문모듈
1. 주문접수 acceptOrder
    1. 주문 생성 mAcceptOrder.createOrder
2. 주문취소 cancelOrder
    1. mCancelOrder.createCancelOrder
3. 주문변경(?) modifyOrder
4. 반품 returnOrder
5. 교환 exchangeOrder

### 주문서브모듈
#### 주문 생성 createOrder
1. 고객정보설정 saveOrderDto.setMOdOrdCstDto
2. 상품정보조회 및 프로모션, 쿠폰적용 mSetOrdPrdInfo.setOrdPrdInfo
    - 상품 정보 조회, 프로모션 정보 조회, 쿠폰 정보 조회
    1. get 주문상품
        1. 주문상품 내 단품ID로 단품 조회로 주문상품에 단품 정보 세팅
        2. get 쿠폰 목록, 주문상품에 쿠폰목록 세팅
        3. 주문상품에 수취 관련정보, 영수증 정보 세팅 세팅
        5. get 프로모션 목록
            1. 프로모션 정보조회
            2. 존재 시 할인적용여부 Y 세팅
        6. get 쿠폰 목록
            1. 쿠폰 정보 조회
                - 쿠폰 발행번호 세팅
            2. 존재 시 할인적용여부 Y 세팅
        7. 주문순번 세팅
        8. 주문상태코드 변경, setOrdStsCd: "1200"
        9. 프로모션, 쿠폰 적용 후 실주문 금액 계산 calcPrdPrice
            1. 단위주문금액 = 단위판매가 + 옵션 가격
            2. 프로모션 조회해서 프로모션할인금액계산
                - 할인율액코드 - 정율: 프로모션할인금액 = 단위주문금액 * 할인율 / 100
                - 할인율액코드 - 정액: 프로모션할인금액 = 할인율액
            3. 주문프로모션에 할인적용금액 세팅
            4. 단위주문금액 = 단위주문금액 - 프로모션할인금액
            5. 쿠폰 조회해서 쿠폰할인금액계산
                - 할인율액코드 - 정율: 쿠폰할인금액 = 단위주문금액 * 할인율 / 100
                - 할인율액코드 - 정액: 쿠폰할인금액 = 할인율액
            6. 주문쿠폰에 할인적용금액 세팅
            7. 단위주문금액 = 단위주문금액 - 쿠폰할인금액
            8. 총주문금액 계산
                - 총주문금액 = 단위주문금액 * 주문수량
            9. 단위주문금액 세팅
            10. 총주문금액 세팅
3. 유효성검증 mOrderValidator.accptOrderValidate
    - 주문접수 검증
    1. 고객 유효성 검증 mOrderCstValidator.validate
        1. 사용여부 Y/N 체크
        2. 휴면고객 체크
        3. 탈퇴고객 체크
    1. get 주문상품
    2. 주문상품 유효성 검증 mOrderPrdValidator.validate
        1. 상품판매상태 체크
        2. 단품판매상태 체크
        3. 성인상품여부 체크
            - 주문고객 생년월일 존재확인
                - 없을 시 본인인증 필요
            - 생년월일로 나이 확인
        4. 프로모션 검증 mOrderMktValidator.pmtValidate
            1. 프로모션 존재 확인
            2. 프로모션 상태 확인
            3. 프로모션 기간확인
            4. 프로모션 대상확인 (상품분류/상품)
        5. 쿠폰 검증 mOrderMktValidator.cpnValidate
            1. 쿠폰 존재 확인
            2. 쿠폰개별상태 확인
            3. 쿠폰 기간 확인
            4. 쿠폰 최소주문금액 확인
    3. 배송비 검증
    4. 결제검증
        1. 주문 금액 + 배송비금액 - 결제금액 = 0 확인
4. 주문데이터 생성 saveAcceptOrder
5. 결제생성 saveOdPay
6. 배송비생성 saveOdOrdDlvf
7. 주문상태변경 (주문상태: 결제완료) saveOrderStatus
    - setOrdStsCd: "1300"

#### 취소 생성 createCancelOrder
1. 주문조회
    1. get 주문상품 mGetOrderInfo.getOrderInfo
2. 취소가능 검증 mOrderValidator.cancelOrderValidator
    1. 취소금액 검증
    2. get 주문상품
        1. 취소상품 검증 mOrderPrdValidator.cancelOrderPrdValidate
            - 주문상태코드 확인 1200, 1300, 1400
3. 환불배송비계산 mGetDlvfInfo.getCancelOrderRfnDlvf
    1. 전체 취소일 때만 취소배송비 환불
    2. 주문ID로 배송비 목록 조회
    3. 배송결제금액 - 배송환불금액 > 0 이면 배송결제금액 - 배송환불금액으로 배송환불금액 세팅
    4. 취소배송비 목록 세팅 mOdOrderDto.setCnlDlvfList
4. 환불금액검증 mOrderRfnValidator.cancelOrderRfnAmtValidate
    1. 총주문 금액 도출
        주문상품목록.stream().map(디티오::getOrdTprc)
                .reduce(BigDecimal.ZERO, BigDecimal::add);
    2. 환불 금액 도출
        환불목록.stream().map(디티오::getRfnArrnAmt)
                .reduce(BigDecimal.ZERO, BigDecimal::add);
    3. 취소배송비금액 합계
        취소배송비목록.stream().map(디티오::getDlvfRfnAmt)
                .reduce(BigDecimal.ZERO, BigDecimal::add);
    4. 검증
        - 총주문 금액 + 취소배송비금액 = 0
5. 주문취소 mSaveOrder.saveCancelOrder
    1. get 주문상품
        1. 주문상품 주문상태 세팅
        2. 주문접수재고수량 저장 saveOrdAccptStckQty
            - 재고변경 mSaveStckData.saveOrdAccptStckQty
                1. 단품 조회
                2. 단품 존재여부 체크
                3. 주문상태 "00"으로 끝나면 변경재고 = 기존 재고 - 주문 재고, "90"으로 끝나면 변경재고 = 기존 재고 + 주문재고
                4. 주문상태 "00"으로 끝나면 변경재고 < 0 확인
                5. 재고수량 세팅
                6. 주문단품 저장
        3. 주문상태코드 저장 saveOrdStsCd
            - 주문상태변경 mSaveOrderStatus.saveOrderStatus
                1. 현재일시 추출
                2. get 주문상품
                    1. 주문상품 존재여부 체크
                    2. 기존 주문상품 주문상태 체크
                        - 1200, 1300, 1400, 1500, 1600 일 시 상태변경 실패
                    3. 주문상태 세팅
                    4. 주문상태일시 세팅
                    5. 주문상품 저장
6. 환불 mSavePaymentData.saveOdRfn
    1. 현재일시 추출
    2. get 환불목록
        1. 결제환불금액 추출
        2. get 결제 목록 (주문ID, 결제수단ID으로 검색)
        3. 환불가능금액 >= 결제 환불금액이면 환불진행금액 = 결제환불금액 아니면 환불진행금액 = 환불가능금액
        4. 환불가능금액 = 환불가능금액 - 환불진행금액
        5. 환불예정금액 = 환불예정금액 + 환불진행금액
        6. 환불금액 = 환불금액 + 환불진행금액
        7. 결제 저장
        8. 결제 생성
            1. 주문Id, 결제수단코드, 결제수단ID, 결제수단관련ID, 환불금액, 결제그룹Id, 환불예정금액 세팅
            2. 결제환불금액 = 결제환불금액 - 환불진행금액
            3. 결제환불금액 = 0 검증
    3. 결제수단별 환불처리
        1. 환불데이터 생성
        2. 주문ID, 결제그룹ID, 결제수단코드, 결제수단ID, 환불금액, 환불예정금액, 환별예정일시, 사용여부 세팅
        3. 저장
        4. 결제수단 코드 100일 시 카드취소 mSaveCrcrdApvData.crcrdCnl
            1. PG 취소 pgInisis.cnl
            2. 카드승인 조회
            3. 카드 환불적용금액 세팅
            4. 카드승인 저장
        5. 결제수단관련Id 세팅
        6. 환불 저장
    4. 배송비환불 mSaveOdDlvfData.saveRfnDlvf
        1. 배송비 환불처리
        2. get 취소배송비 목록
            1. 배송비 존재여부 체크
            2. 배송비환불금액 세팅
            3. 배송비 저장