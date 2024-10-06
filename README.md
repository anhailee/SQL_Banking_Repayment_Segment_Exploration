# SQL_Banking_Repayment_Segment_Exploration
SQL_Banking_Repayment_Segment_Exploration

## **1. INTRODUCTION**

This project aims to compile and analyze customer payment data at VNC Bank for Q1 2024. The report tracks detailed transactions for January, February, and March, including cash inflows, outflows, deposits, withdrawals, and the closing balance. Additionally, the project highlights the Top 3 customers with the highest total transaction volumes, serving the purpose of analyzing customer behavior and optimizing banking services.

## **2. THE GOAL OF PROJECT**

**1. Monitor and evaluate payment activities:** Analyze customer cash flow on a monthly basis to assess transaction activity and the level of usage of banking services.
   
**2. Manage account balances:** Track changes in opening and closing balances to ensure effective cash flow control and manage the bank’s liquidity status.
   
**3. Identify key customers:** Identify the top customers with the highest transaction volumes to create tailored strategies for customer relationship management, offering services that enhance customer satisfaction and loyalty.

## **3. EXPLORING DATASET**

**Query 1: Report on the Credit Situation for the First Half of the Year**

```
sql

-- I. BÁO CÁO TÌNH HÌNH THANH TOÁN QUÝ I/2024

-- SỐ DƯ ĐẦU KỲ
SELECT 
  MONTH(actiondate) AS Thang,
  SUM(pre_balance) AS So_du_dau_ky
FROM 
(
    SELECT 
        casa_account,
        actiondate,
        pre_balance,
        ROW_NUMBER() OVER(PARTITION BY casa_account, MONTH(actiondate) ORDER BY actiondate) AS [RANK]
    FROM Transfer_history
) X
WHERE [RANK] = 1
AND MONTH(actiondate) IN (1, 2, 3)
GROUP BY MONTH(actiondate);

-- TIỀN RA
SELECT 
  MONTH(actiondate) AS Thang,
  SUM(decrease) AS Tien_ra
FROM Transfer_history
WHERE casa_account = Sent_account
AND destination_account != ''
AND MONTH(actiondate) IN (1, 2, 3)
GROUP BY MONTH(actiondate);

-- TIỀN VÀO
SELECT 
  MONTH(actiondate) AS Thang,
  SUM(increase) AS Tien_vao
FROM Transfer_history
WHERE casa_account = destination_account
AND Sent_account != ''
AND MONTH(actiondate) IN (1, 2, 3)
GROUP BY MONTH(actiondate);

-- RÚT TIỀN MẶT
SELECT 
  MONTH(actiondate) AS Thang,
  SUM(decrease) AS Rut_tien_mat
FROM Transfer_history
WHERE destination_account = ''
AND MONTH(actiondate) IN (1, 2, 3)
GROUP BY MONTH(actiondate);

-- NỘP TIỀN MẶT
SELECT 
  MONTH(actiondate) AS Thang,
  SUM(increase) AS Nop_tien_mat
FROM Transfer_history
WHERE Sent_account = ''
AND MONTH(actiondate) IN (1, 2, 3)
GROUP BY MONTH(actiondate);

-- SỐ DƯ CUỐI KỲ
SELECT 
  MONTH(actiondate) AS Thang,
  SUM(af_balance) AS So_du_cuoi_ky
FROM 
(
    SELECT 
        casa_account,
        actiondate,
        af_balance,
        ROW_NUMBER() OVER(PARTITION BY casa_account, MONTH(actiondate) ORDER BY actiondate DESC) AS [RANK]
    FROM Transfer_history
) X
WHERE [RANK] = 1
AND MONTH(actiondate) IN (1, 2, 3)
GROUP BY MONTH(actiondate);

``

**RESULT**

![image](https://github.com/user-attachments/assets/4b3730b4-e878-46a1-b8ef-a5a189523ee1)

Query 2:

```
sql
SELECT TOP 3
	CUS.custid								AS [MÃ KHÁCH HÀNG]
	,IC.iccbrname								AS [CHI NHÁNH]
	,CUS.custname 								AS [TÊN KHÁCH HÀNG]
	,TH.casa_account							AS [TÀI KHOẢN CASA]
	,SUM(TH.increase) + SUM(decrease)					AS [TỔNG TIỀN GIAO DỊCH]
	,RANK() OVER (ORDER BY (SUM(TH.increase) + SUM(decrease)) DESC )	AS [RANK]
FROM TRANSFER_HISTORY AS TH
LEFT JOIN CUSTOMER AS CUS
ON TH.casa_account = CUS.casa_account
LEFT JOIN ICC_BRANCH AS IC
ON CUS.branch = IC.iccbrcode
GROUP BY 
CUS.custid				
,IC.iccbrname			
,CUS.custname 				
,TH.casa_account	
```

**RESULT**

![image](https://github.com/user-attachments/assets/2937d988-d7f9-4fad-8174-9b2d8a7ec915)


