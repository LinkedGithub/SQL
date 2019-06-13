## Question:
![](https://github.com/LinkedGithub/sql/blob/master/images/f1.jpg)

## Answer:
``` sql
#For question 5
create table orders
(
  num int not null primary key auto_increment,
	ordertime datetime not null,
  customer char(50) not null
);
select * from orders;
insert into orders (ordertime, customer) values ('2017-01-01 12:35:01', 'zhangsan');
insert into orders (ordertime, customer) values ('2017-01-02 12:35:01', 'zhangsan'), 
('2017-01-02 12:40:01', 'lisi'),('2017-01-02 13:40:01', 'lisi'), ('2017-01-03 12:40:01', 'lisi');

select date(t1.ordertime), count(distinct t1.customer) as total, count(distinct t2.customer) as old, 
count(distinct t1.customer) - count(distinct t2.customer) as newly 
from orders as t1 left join orders as t2 on (t1.customer = t2.customer) and 
(date(t2.ordertime) < date(t1.ordertime))
group by date(t1.ordertime) order by date(t1.ordertime);
```

## Question:
Given the below subset of a travel app’s schema, write executable SQL queries to answer the two questions below. 
Please answer in a single query and assume read-only access to the database (i.e. do not use CREATE TABLE).
Assume a PostgreSQL database, server timezone is UTC
![](https://github.com/LinkedGithub/sql/blob/master/images/f2.JPG)
1.	Between Oct 1, 2013 at 10am PDT and Oct 22, 2013 at 5pm PDT, what percentage of requests made by 
unbanned clients each day were canceled in each city?
2.	For city ids 1, 6, and 12, list the top three drivers by number of completed trips for each week 
between June 3, 2013 and June 24, 2013.

## Answer:
``` sql
1.select Request_at as 'Day', City_id as 'City',
(round(
sum(
   		case when status = 'cancelled_by_client' then 1
            else 0 end) / count(*)
 	,2)
) as 'Rate' 
from trips join users on Client_Id = Users_Id 
where Banned = 'No' and Role ='client' and
(Request_at between (convert_tz('2013-10-01 10:00:00','-07:00','+00:00')) and 
(convert_tz ('2013-10-22 17:00:00','-07:00','+00:00'))) 
group by Request_at, city_id order by Request_at asc

2.select info.city_id as 'City', info.users_id as 'driver', info.week as 'week' from 
(select city_id, users_id, date_format(request_at, '%Y-%u') as week, count(*) as orders 
from trips join users on Driver_Id = Users_Id 
where trips.status = 'completed' and City_Id in (1,6,12)
and (Request_at between '2013-06-03' and '2013-06-24') 
group by date_format(request_at, '%Y-%u'), city_id) 
as info
where 3 > 
(select count(distinct temp.users_id) from 
(select city_id, users_id, date_format(request_at, '%Y-%u') as week, count(*) as orders 
from trips join users on Driver_Id = Users_Id 
where trips.status = 'completed' and City_Id in (1,6,12)
and (Request_at between '2013-06-03' and '2013-06-24') 
group by date_format(request_at, '%Y-%u'), city_id) 
as temp
where temp.city_id = info.city_id and temp.week = info.week and 
temp.orders > info.orders)
```

## Question:
![](https://github.com/LinkedGithub/sql/blob/master/images/f3.png)
MySQL查询各个用户最长的连续登陆天数

## Answer:
``` sql
create table orders
(
	num int not null primary key auto_increment,
    uid int not null,
    loadtime datetime not null
);
select * from orders;
(select * from orders order by uid asc, loadtime desc);
insert into orders (uid, loadtime) values (201, '2017-01-01');
insert into orders (uid, loadtime) values (201, '2017-01-02'),
(202, '2017-01-02'), (202, '2017-01-03'), (203, '2017-01-03'),
(201, '2017-01-04'), (202, '2017-01-04'), (201, '2017-01-05'),
(202, '2017-01-05'), (201, '2017-01-06'), (203, '2017-01-06'),
(203, '2017-01-07');
insert into orders (uid, loadtime) values (204, '2017-01-02');

select uid, max(results) from
  (select uid, (case when datediff(@cur, @cur := loadtime) = 1 and (@id - (@id := uid) = 0) 
                                          then @count := @count + 1
              else  (@count := 1 and  @id := uid) END) as results
  from (select * from orders order by uid asc, loadtime desc) as c, 
        (select @cur := -1, @count := 0, @id := -1) as b) as temp
group by uid order by uid asc;
```
