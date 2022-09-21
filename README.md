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

