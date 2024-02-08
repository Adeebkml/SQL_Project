# SQL_Project
•	In this project I have analyzed Zomato Dataset based on Tables such as Sales, GoldUsers_Signup, Products and Users.

Tables:
1) Sales Table.
drop table if exists sales;
CREATE TABLE sales(userid integer,created_date date,product_id integer); 

INSERT INTO sales(userid,created_date,product_id) 
 VALUES (1,'2017-04-19',2),
(3,'2019-12-18',1),
(2,'2020-07-20',3),
(1,'2019-10-23',2),
(1,'2018-03-19',3),
(3,'2016-12-20',2),
(1,'2016-11-09',1),
(1,'2016-05-20',3),
(2,'2017-09-24',1),
(1,'2017-11-03',2),
(1,'2016-11-03',1),
(3,'2016-10-11',1),
(3,'2017-12-07',2),
(3,'2016-12-15',2),
(2,'2017-11-08',2),
(2,'2018-09-10',3);

2) GoldUsers_Signup Table:
drop table if exists goldusers_signup;
CREATE TABLE goldusers_signup(userid integer,gold_signup_date date); 

INSERT INTO goldusers_signup(userid,gold_signup_date) 
 VALUES (1,'2017-09-22'),
(3,'2017-04-21');

3) Products Table:
CREATE TABLE product(product_id integer,product_name text,price integer); 

INSERT INTO product(product_id,product_name,price) 
 VALUES
(1,'p1',980),
(2,'p2',870),
(3,'p3',330);

4) User Table:
INSERT INTO users(userid,signup_date) 
VALUES (1,'2014-09-02'),
(2,'2015-01-15'),
(3,'2014-04-11');


Questions:
Q1 What is the total amount each customer spent on Zomato?
->select a.userid, sum(b.price) as Total_Amount from 
sales as a inner join product as b on a.product_id = b.product_id group by a.userid;

Q2 How many days has each customer visited zomato ?
select userid , count(created_date) from sales group by userid  ;

Q3 What was the first product purchased by each customer ?
select * from  (select * , rank() over(partition by userid order by created_date) as rk from sales) as a where rk = 1  ;

Q4 What is the most purchased item on the menu and how many times was it purchased by all customers?
select userid , count(product_id) from sales where product_id = 
(select product_id from sales group by product_id order by count(product_id) desc limit 1) group by userid;

Q5 which item was the most popular for each customer?
select userid ,product_id , count(product_id)
 from sales group by product_id , userid;

 Q6 which item was purchased first by the customers after they become a member?
 select * from 
 (select c.* , rank() over( partition by userid order by created_date ) as rk from
 (select a.userid , a.created_date , a.product_id , b.gold_signup_date from sales as a
 inner join goldusers_signup as b on a.userid = b.userid and created_date >= gold_signup_date) as c ) as d where rk = 1 ;

 Q7  Which item was purchased just before the customer become a member ? 
select * from 
(select c.* , rank() over( partition by userid order by created_date ) as rk from
(select a.userid , a.created_date , a.product_id , b.gold_signup_date from sales as a
inner join goldusers_signup as b on a.userid = b.userid and created_date <= gold_signup_date) as c ) as d where rk = 1 ;

Q8   What is the total orders and amount for each member before they became a member?
select userid , count(created_date) from
(select c.* , d.price  from 
(select a.userid , a.created_date , a.product_id , b.gold_signup_date from sales as a inner join 
goldusers_signup as b on a.userid = b.userid and created_date <= gold_signup_date) as c inner join product as d on c.product_id = d.product_id) as r  ;

Q9 If Buying each product generates points for eg 5rs = 2 zomato points and each points products has different purchasing points
for eg p1 5rs = 1 zomato points , for p2 10rs = 5 zomato points , for p3 5rs = 1 zomato points  
 Calculate points collected by each customers and for which products most points have been given till now.
 
select userid , Total_Points_Per_Customer * 2.5 as Total_Cash_Back_Earned from
(select userid , sum( Total_Points) as Total_Points_Per_Customer from
(select g.* , Amount/Rupees_Per_Points as Total_Points from
(select e.* , case when product_id = 1 then 5 when product_id = 2 then 2 when product_id = 3 then 5 else 0 end as Rupees_Per_Points from
(select c.userid , c.product_id , sum(c.price) as Amount from
(select a.* , b.price from sales as a inner join product as b on 
a.product_id = b.product_id) as c group by userid , product_id) as e ) as g) as i group by userid ) as t;

Q10 In the first one year after a customer joins gold program ( Including their join date )  
 irrespective of what the customer has purchased they earn 5 zomato points for every 10 rs spent who earned more  
 1 or 3 and what was their points earnings in their first yr?

select c.* , d.price*0.5 as Total_Points_earned from
(select a.userid , a.created_date , a.product_id , b.gold_signup_date from sales as a
 inner join goldusers_signup as b on a.userid = b.userid and created_date >= gold_signup_date and created_date <= Date_add(gold_signup_date , INTERVAL 1 year)) as c 
 inner join product as d on c.product_id = d.product_id;

 Q11 Rank all the transaction of all the Customers
 select * , rank() over(partition by userid order by created_date) as rnk from sales ;

 Q12  Rank all the transaction for each member whenever they are a zomato gold member for every non gold member transaction mark as NA
 select c.* ,case when gold_signup_date is null then 0 else rank() over(partition by userid  order by gold_signup_date desc) end as rnk from
 (select a.userid , a.created_date , a.product_id , b.gold_signup_date from sales as a
 left join goldusers_signup as b on a.userid = b.userid and created_date >= gold_signup_date) as c;
