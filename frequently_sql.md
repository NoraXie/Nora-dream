# 2017-11-08

```sql
select * from depart where depart_id = 244;

select * from `advisor_transfer_records` where advisor_id = 282946977384450;

select * from user;

select * from area where area_name = '长春';

select * from user where real_name = '张小强';

select * from appointment where user_id = 399861219682302;

select * from transfer_records where customer_id = 421281027571711;

select * from user u left join customer c on u.user_id = c.user_id
left join ( select cfu.`customer_id`, min(cfu.`create_date`) as minday from `customer_follow_up` cfu 
group by customer_id ) b on b.customer_id = c.customer_id ;

select * from  (
select cfu.`customer_id`, min(cfu.`create_date`) as minday from `customer_follow_up` cfu 
group by customer_id ) b ;
```