<img width="979" height="54" alt="스크린샷 2026-06-18 122920" src="https://github.com/user-attachments/assets/597e59f1-3ea9-489a-8d67-8d47e366382c" /># ops-recovery-drills
장애 대응 시나리오


**장애 시나리오: [R5] Iceberg mart empty**

**장애 전 정상 기준**
olist_category_daily = 759 rows (Iceberg)
StarRocks BI category_daily 정상 집계 (날짜 × 카테고리)

**장애 후 관찰 증상**
1. Airflow DAG run은 여전히 success
2. BI는 오류를 보임

**계층별 증거 3개**
Iceberg / Spark 계층
<img width="979" height="54" alt="스크린샷 2026-06-18 122920" src="https://github.com/user-attachments/assets/c12966e6-2b69-4bc9-bb4f-44e5912e2a99" />

<img width="152" height="17" alt="스크린샷 2026-06-18 122539" src="https://github.com/user-attachments/assets/d7abd97e-2f14-431e-b3a5-94d9d7b633b1" />
category_daily table이 비어있다는 사실을 알 수 있다.


**StarRocks / BI 계층**
<img width="1808" height="592" alt="스크린샷 2026-06-18 121704" src="https://github.com/user-attachments/assets/7ca1be3f-c1e3-4938-a0b7-850eb89f7bc8" />

<img width="1738" height="583" alt="스크린샷 2026-06-18 123411" src="https://github.com/user-attachments/assets/edc90758-5b8a-4841-bcf8-7ac91dfb2fe7" />
BI가 작동이 안되거나 KeyError 크래시를 표시한다.



**복구 명령과 복구 후 결과**
REBUILD 방법 채택, Paimon에서 다시 derive하는 방식이라 느리고 원천이 정상이어야 한다.
Airflow DAG를 재실행하여 mart를 재생성한다.

<복구 기준점 확인>
<img width="724" height="209" alt="스크린샷 2026-06-18 123048" src="https://github.com/user-attachments/assets/4df38327-b733-4186-ac67-0279bf71b2a0" />


<복구 후 759 row 확인>
<img width="651" height="69" alt="스크린샷 2026-06-18 123752" src="https://github.com/user-attachments/assets/0e505278-7da6-40c2-a1e8-eb6924283405" />

<KeyError 대신 Analytics 테이블 표시>
<img width="1782" height="455" alt="image" src="https://github.com/user-attachments/assets/8538bf89-6198-4b67-b1d0-478a2d57f938" />



**재발 방지 / 개선 아이디어**
1. DAG의 validate 단계에 서빙 mart row count 검증 추가
2. 모니터링 시스템 추가 (임계치, airflow failure callback)
 
