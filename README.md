#  Customer Analysis (RFM-Segmentation)
Customer segmentation using SQL (RFM model) to identify loyal, at-risk, and churned customers.

## Question-01: 
##### Analyse Customer Lifecycle Value (CLV) with regards to (RFM Model) Recency, Frequency and Monetary Value

###### STEP-i:
        
                SELECT  max(str_to_date(orderdate, '%d/%m/%y')) FROM SALES_DATA;  -- '2020-12-17' {Last Business Day}
                SELECT  min(str_to_date(orderdate, '%d/%m/%y')) FROM SALES_DATA;  -- '2020-01-02' 

###### STEP-ii:
                SELECT 
                	  CUSTOMERNAME,
                    ROUND(SUM(SALES),0) AS CLV,
                    count( DISTINCT ORDERNUMBER) AS FREQUENCY,
                    SUM(QUANTITYORDERED) AS TOTAL_QANITITY,
                    max(str_to_date(orderdate, '%d/%m/%y')) as customer_last_transaction_date,
                    datediff((SELECT  max(str_to_date(orderdate, '%d/%m/%y')) FROM SALES_DATA), max(str_to_date(orderdate, '%d/%m/%y'))) as recency
                FROM sales_data
                GROUP BY CUSTOMERNAME;

### Output:
### SQL Skills Applied:

## Question-02: 
##### Find out customerwise Recency, Frequency, Monetary and Total Purchase

###### STEP-I: Aggregation and CTEs
                              a. RFM with Aggregate Functions
                              b. with Common Table Expression CTE

                WITH CLV AS(
                SELECT 
                	  CUSTOMERNAME,
                    max(str_to_date(orderdate, '%d/%m/%y')) as customer_last_transaction_date,
                    datediff((SELECT  max(str_to_date(orderdate, '%d/%m/%y')) FROM SALES_DATA), max(str_to_date(orderdate, '%d/%m/%y'))) as Recency,
                    count( DISTINCT ORDERNUMBER) AS Frequency,
                	  ROUND(SUM(SALES),0) AS Monetary,
                    SUM(QUANTITYORDERED) AS TOTAL_QANITITY   
                FROM sales_data
                GROUP BY CUSTOMERNAME),
                
###### STEP-II: Window and CTEs
                
                RFM_Score as(
                SELECT 
                  *,
                  NTILE(5) OVER (ORDER BY Recency 	desc) 	as R_Score,
                  NTILE(5) OVER (ORDER BY Frequency 	    ) 	as F_Score,
                  NTILE(5) OVER (ORDER BY Monetary 	    ) 	as M_Score
                    
                FROM CLV),

###### STEP-III: RFM Score
                           
                  
                RFM_COMBINATION AS(
                
                select 
                  *,
                  concat_ws('',R_SCORE,F_SCORE,M_SCORE ) AS RFM_COMBINATION
                from RFM_SCORE)

                          STEP-04:
                            a. RFM Category with CASE WHEN in response to RFM Score
                            b. Final RFM Model (Customer Analysis)
                
                SELECT 
                	   *,
                     CASE
                		    WHEN RFM_COMBINATION IN (455, 515, 542, 544, 552, 553, 452, 545, 554, 555) THEN "Champions"
                        WHEN RFM_COMBINATION IN (344, 345, 353, 354, 355, 443, 451, 342, 351, 352, 441, 442, 444, 445, 453, 454, 541, 543, 515, 551) THEN 'Loyal Customers'
                        WHEN RFM_COMBINATION IN (513, 413, 511, 411, 512, 341, 412, 343, 514) THEN 'Potential Loyalists'
                        WHEN RFM_COMBINATION IN (414, 415, 214, 211, 212, 213, 241, 251, 312, 314, 311, 313, 315, 243, 245, 252, 253, 255, 242, 244, 254) THEN 'Promising Customers'
                        WHEN RFM_COMBINATION IN (141, 142,143,144,151,152,155,145,153,154,215) THEN 'Needs Attention'
                        WHEN RFM_COMBINATION IN (113, 111, 112, 114, 115) THEN 'About to Sleep'
                        ELSE "Other"
                        END AS CUSTOMER_SEGMENT
                FROM RFM_COMBINATION;
### Output:
### SQL Skills Applied:

## Question-03: 
##### Create a VIEW in the database for the RFM Model and find out Customer Segmentation

##### STEP-I: VIEW for RFM Model
                
          CREATE VIEW RFM_ANALYSIS AS
          
          WITH CLV AS(
                SELECT 
                	  CUSTOMERNAME,
                    max(str_to_date(orderdate, '%d/%m/%y')) as customer_last_transaction_date,
                    datediff((SELECT  max(str_to_date(orderdate, '%d/%m/%y')) FROM SALES_DATA), max(str_to_date(orderdate, '%d/%m/%y'))) as Recency,
                    count( DISTINCT ORDERNUMBER) AS Frequency,
                	  ROUND(SUM(SALES),0) AS Monetary,
                    SUM(QUANTITYORDERED) AS TOTAL_QANITITY
                    
                FROM sales_data
                GROUP BY CUSTOMERNAME),      
                
          RFM_Score as(
                SELECT 
                  *,
                  NTILE(5) OVER (ORDER BY Recency 	desc) 	as R_Score,
                  NTILE(5) OVER (ORDER BY Frequency 	    ) 	as F_Score,
                  NTILE(5) OVER (ORDER BY Monetary 	    ) 	as M_Score    
                FROM CLV),
                
          RFM_COMBINATION AS(
               select 
                  *,
                  concat_ws('',R_SCORE,F_SCORE,M_SCORE ) AS RFM_COMBINATION
                  from RFM_SCORE)
                
                SELECT 
                	      *,
                        CASE
                		    WHEN RFM_COMBINATION IN (455, 515, 542, 544, 552, 553, 452, 545, 554, 555) THEN "Champions"
                        WHEN RFM_COMBINATION IN (344, 345, 353, 354, 355, 443, 451, 342, 351, 352, 441, 442, 444, 445, 453, 454, 541, 543, 515, 551) THEN 'Loyal Customers'
                        WHEN RFM_COMBINATION IN (513, 413, 511, 411, 512, 341, 412, 343, 514) THEN 'Potential Loyalists'
                        WHEN RFM_COMBINATION IN (414, 415, 214, 211, 212, 213, 241, 251, 312, 314, 311, 313, 315, 243, 245, 252, 253, 255, 242, 244, 254) THEN 'Promising Customers'
                        WHEN RFM_COMBINATION IN (141, 142,143,144,151,152,155,145,153,154,215) THEN 'Needs Attention'
                        WHEN RFM_COMBINATION IN (113, 111, 112, 114, 115) THEN 'About to Sleep'
                        ELSE "Other"
                        END AS CUSTOMER_SEGMENT
                FROM RFM_COMBINATION;
                
                SELECT * FROM RFM_ANALYSIS;

###### STEP-II: 
                SELECT 
	                      CUSTOMER_SEGMENT,
	                      SUM(MONETARY) AS TOTAL_SPENDING,
	                      ROUND(AVG(MONETARY),0) AS AVERAGE_SPENDING,
                        SUM(FREQUENCY) AS TOTAL_ORDER
                        -- SUM(TOTAL_QUANTITY) AS TOTAL_QTY_ORDERED
                FROM rfm_analysis
                      GROUP BY CUSTOMER_SEGMENT;

### Output:
### SQL Skills Applied:

## Question-04: 
#### Stored Procedure
                    delimiter $$
                        create procedure Customer_LV( in customername varchar(50), out CLV int)
                            begin
                        		    SELECT 
                        			      ROUND(SUM(SALES),0) into CLV
                        		    FROM sales_data
                        		        where CUSTOMERNAME = 'Cruz & Sons Co.';
                            end $$
                    delimiter ;
                        
                    call Customer_LV ('Cruz & Sons Co.', @CLV);
                    select @CLV;
    
### Output:
### SQL Skills Applied:





