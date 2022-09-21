# DealingWithMessyData SQL training

Let assume the company has an internal policy to determine its customers' credit limit, but this procedure has been questioned recently by the board as being too conservative.

CEO wants to increase the current customer base credit limits in order to upsell a new line of products. In order to do that, the company hired several external consultancies to produce new credit limit estimates.

The problem is that each agency has produced the report in its own format. Some use the format "First-name Last-name" to identify a person, others use the format "Last-name, First-name". There is also no consensus on how to capitalize each word, so some used all uppercase, others used all lowercase, and some used mixed-case.

Also, some names are titled, for example: "Dr. Hannibal Lecter", "Robert Downey Jr." etc, so you will need to pay attention to any such or similar cases.

Internally, the data is structured as follows:

Table: customers
================

id: INT
first_name: TEXT
last_name: TEXT
credit_limit: FLOAT
The data you've received from all agencies was consolidated in the following table:

Table: prospects
================
full_name: TEXT
credit_limit: FLOAT
The agencies had access only to a partial customer base. There is also the possibility of more than one agency prospecting the same customer, so it's highly likely that there will be duplicates. 

For this task i am interested in the prospected customers that are already in the customer base and the prospected credit limit is higher than  internal estimate. When more than one agency prospected the same customer, chose the highest estimate.

The result should be with the following fields:

first_name
last_name
old_limit [the current credit_limit]
new_limit [the highest credit_limit found]

WORKS ON POSTGREESQL 9.6

WITH
-- separate full_name with the space delimeter
devidedNames
AS
(

SELECT arr[1] as ar1,arr[2] as ar2, 
       arr[3] as ar3, full_name, credit_limit from
(
select regexp_split_to_array(initcap(full_name) ,' ') as arr,initcap( full_name) as full_name, credit_limit from prospects)
as subq
)
,
cleanedNames 
--  if full_name starts form 'MS.', 'MISS', 'Mr.', take the second item of array
--  if there is a ',' in the full_name, then the last_name goes first in the  full_name
-- erace comma in the last_name and first_name
AS
(
SELECT  
CASE WHEN position('.' in ar1)>0 or position('Miss' in ar1)>0 or position(',' in ar1)>0 THEN
     trim(regexp_replace( ar2,'[,]+','','g')) 
     Else 
     trim(regexp_replace( ar1,'[,]+','','g'))
END as first_name,
CASE WHEN position('.' in ar1)>0 or position('Miss' in ar1)>0  THEN
     trim(regexp_replace( ar3,'[,]+','','g'))
     WHEN  position(',' in ar1)>0 THEN 
     trim(regexp_replace( ar1,'[,]+','','g'))
     Else 
     trim(regexp_replace( ar2,'[,]+','','g'))
END as last_name, full_name, credit_limit
FROM  devidedNames order by first_name, last_name
),
GroupedNames
--grouping the same customers and defining the highest credit_limit
AS
(
SELECT first_name as first_name ,
       last_name as last_name, 
       
       MAX(cleanedNames.credit_limit) as new_limit from cleanedNames 
GROUP BY first_name ,
  last_name
  ORDER BY first_name , last_name)

-- the resulting table showing only those customers , who have new_limit > then old_limit 
SELECT CUS.first_name , CUS.Last_name ,
credit_limit as old_limit , New_limit
FROM GroupedNames
RIGHT JOIN customers as CUS
ON UPPER(GroupedNames.first_name) =UPPER(CUS.first_name )
and UPPER( GroupedNames.last_name) =UPPER(cus.last_name)
WHERE GroupedNames.new_limit>=cus.credit_limit
ORDER BY first_name,last_name


