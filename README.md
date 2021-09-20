# Budget Web App 
Author: Jennifer Zeng

Contributors: Alan Chen, Andy Huang, Daanyal Akhtar, Michelle Tang, Sebastian Lopez, Tomer Yaakov

[Design Doc](https://docs.google.com/document/d/1yhA75AGxCYOdqQVFb3dU0rH4Cb3hcuQu0ZmYiSqXpFA/edit#heading=h.wmr0r83w17jn)

## Motivation 
Online transactions have become increasingly popular with the advent of ecommerce and online banking. 
Budget management naturally shifts from paper and books to online applications. 
A good budgeting app helps you understand your income and spending, so you have maximum control over your money. 

Typically, budgeting apps work by connecting to your bank account and credit cards to automatically download 
transactions and categorize your spending to match the budget you choose. However, one limitation of most 
current budget apps is that they don’t offer customers the option to customize categories. 
This is the problem we will address in this product. 
Besides providing category suggestions, we allow customers to customize and detail categories according to their preferences. 


## Customer Pain Point 
As mentioned in Motivation, one customer pain point is that most budget apps don’t offer 
the option to customize categories. The lack of customization results in lower accuracy in 
categorizing transactions. For example, on most budget apps, transactions at CVS are often 
categorized as health, fitness, or medicine, since CVS as a merchant is categorized as a pharmacy. 
This is often inaccurate. Think about how many times you buy food, drinks, and household items from CVS. 
Thus, this is the problem we want to solve with customizable categories.


## Audience 
We target those who want to have a detailed view of their spending, gain insights into their spending habits, and cultivate 
a budgeting mindset. More specifically, young people fresh out of high school/college are one of our target groups. During this time, 
they transition from spending parents' money to making money on their own. Previously, they may spend purely based on their likings without 
thinking about if the money is well-spent or the cost-effectiveness of such spending. Our app exists to cultivate an awareness of money and 
budgeting. By providing them with customizable categories, we enable our audience to see detailed breakdowns of their spending, 
which will in turn help them learn more about their spending habits and invoke their reflections about money. 

## Implementation 

### Architecture 
<p align="center">  
    <img src="./images/architecture.png" width="568" height="242"> 
</p>

According to the course requirements, we need to have frontend (javascript), backend (nodejs, expressjs), 
and database (mongoDB) components. The overall architecture is illustrated above. 

The frontend serves as an entry point for users to select their bank and enter bank account info 
(here we can use the interface that Plaid provides). This information is passed to the backend, 
which will query the database directly and return results from the database. At the same time, 
backend can run a background scheduler to periodically query APIs to update the database. 
The goal for adding the background scheduler is to hide the latency from users, since API query 
normally takes quite some time to populate back to frontend. 

However, one concern is that it is actually hard to pre-populate users’ bank information ahead of time, 
since we need to wait for them to input their bank account info before we can actually fetch from API, 
ie. query to API requires users’ bank account information as input. But the good side of having a database as 
a cache is that if a user repeatedly query our backend in a fairly short period of time, we will be able to 
seamlessly return the results back to the user from the database without having to query API. 


### Fetching Bank Transactions 

#### Method 1: Plaid API 
Language: Python, NodeJS 

We are going to use the API gateways that Plaid provided. By following [this tutorial](https://dashboard.plaid.com/overview/sandbox), 
I was able to get bank transactions: 
<p align="center">  
    <img src="./images/transaction.png" width="310" height="384"> 
</p>

More thoughts on Plaid: think of Plaid as a middle layer where it already connects to all the banks and is 
able to fetch bank account information including transaction, investment, savings, and etc. Plaid APIs provide a 
gateway for us to use its interface (think of this interface as starting from the point where it asks you to select the bank). 
We only need to select a bank and input bank account information. Plaid is responsible for getting all the results back. 


#### Method 2: Mint API 
Language: Python, NodeJS

[Mintapi](https://github.com/dhleong/pepper-mint) provides APIs for directly getting both bank transactions and categories. 
This may be an easy path for us. However, it uses Mint accounts to log in, but not everyone is a Mint user. 


### Categorize Transactions 
We allow customers to customize their own categories. After we get back transactions, enabling the customization 
falls into frontend work. We will have the following: 
* A modal for creating categories; 
* Mechanisms that allow customers to categorize transactions, for example, 
draggable React components of transactions that can be directly dragged into a newly-created category. 


## Appendix 
### Steps for querying Plaid APIs 
1. Go to [this tutorial](https://dashboard.plaid.com/overview/sandbox) and follow instructions. This tutorial is for testing in Sandbox environment.  
Note that, you’ll be able to run the frontend page up, but if you enter your actual bank info, you will not get through. 
The reason is that Sandbox environment doesn’t use real-world data. Regarding different environment terminology, see [here](https://plaid.com/docs/quickstart/glossary/#environments).
2. To use actual bank account, modify your env to ```'development'``` in .env file of your directory 
3. You may encounter an SSL certificate error. To solve this error, follow [this post](https://stackoverflow.com/questions/50236117/scraping-ssl-certificate-verify-failed-error-for-http-en-wikipedia-org). 

### Justification for scope
This project is stackable in nature, meaning that we can make it easier or more complex depending on the progress. 

I envision the frontend will have two pages, one for users to input their bank account information and the other 
one for displaying transactions and customize their categories. This is the basic functionality. To make it more complex,
we can add other navigation tabs that can navigate to other functionalities, like investment, savings, and etc. 

For the backend, it could be as simple as querying remote APIs. If we want it to be more performant, we can add the background scheduler
as I described in the implementation section. Background scheduler periodically pings remote servers to update the database. Once users make
query, we can directly return whatever is in the database to them, without having API query delay. 



