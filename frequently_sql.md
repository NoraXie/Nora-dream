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

# 组织架构 
select 
c.战队id, c.战队名称, 
c.野战军id, c.野战军名称,
p.depart_id as 集团军id, 
p.dept_name as 集团军名称, 
p.parent_id from depart p left join (select 
c.depart_id as 战队id, c.dept_name as 战队名称, 
c.parent_id as 野战军id, p.dept_name as 野战军名称, 
p.parent_id as 集团军id from depart c left join depart p 
on c.parent_id = p.depart_id) c on
p.depart_id = c.集团军id 
where c.集团军id in (192,193,194,195)
order by c.集团军id, c.野战军id, c.战队id desc;

# 查询顾问及基所属组织架构
select 
u.user_id as 顾问id, u.real_name as 顾问姓名, u.sex as 性别,
d.战队id, d.战队名称, d.野战军id, d.野战军名称, 
d.集团军id, d.集团军名称 from user u join (
select 
c.战队id, c.战队名称, 
c.野战军id, c.野战军名称,
p.depart_id as 集团军id, 
p.dept_name as 集团军名称, 
p.parent_id from depart p left join (select 
c.depart_id as 战队id, c.dept_name as 战队名称, 
c.parent_id as 野战军id, p.dept_name as 野战军名称, 
p.parent_id as 集团军id from depart c left join depart p 
on c.parent_id = p.depart_id) c on
p.depart_id = c.集团军id 
where c.集团军id in (192,193,194,195)) d on d.战队id = u.department_id
where u.is_advisor = 1 and u.status = 2 and u.role_bitmap = 2 order by d.集团军id, 野战军id, 战队id;

# 在职顾问且身份证号为空的
select * from user where is_advisor = 1 and status = 2 and role_bitmap = 2 and identity_card is null;

# 身份证号关联并更新
select user_id, real_name, identity_card, telephone, tmp_card.card, tmp_card.phone, tmp_card.`lastname` from user 
left join tmp_card on user.telephone = tmp_card.phone
where user.identity_card is null 
order by tmp_card.card desc;


select user.telephone, tmp_card.phone, tmp_card.card, user_id, user.identity_card from user left join tmp_card on tmp_card.phone = user.telephone 
where user.user_id in (361463675400191,
379883902433278,
407468271351806,
356905262188542,
417600638337023,
 355608241117182,
 353412533794814,
 380565261742079,
 412118073544703,
 413525760014334,
 363075931160575,
 362432923652095,
 412118073538558,
 353361102213119,
 360531474315262,
 360280523831294,
 417778625433599,
 359319627943935,
 359671941226495,
 361789609988095,
 358970000924671,
 353722939506687,
 413525760010238,
 358726076264447,
 357640854124542,
 413525760018430,
 361090319157246,
 357953681631230,
 387817021204478,
 361201010692095,
 377936542824446,
 359371537174527,
 357017584777214,
 412118073548799,
 360201383997439,
 361484144719870,
 359870615240703,
 358092511418366,
 358800891191295,
 362570637692927,
 403021601058815,
 403021601058815,
 357939820822527,
 375790971363327,
 412118073546750,
 359515411439615,
 362601777397759,
 362540692815871,
 411925167800319,
 422174213283839,
 359951772010494,
 362384488134654,
 354191617808382,
 406759308523518) ;
 
 
 # 没有身份证号的在职顾问, 顾问姓名也没有, 需要去oa上查
 select 
u.user_id as 顾问id, u.real_name as 顾问姓名, u.identity_card as 身份证号, u.sex as 性别,
d.战队id, d.战队名称, d.野战军id, d.野战军名称, 
d.集团军id, d.集团军名称 from user u left join (
select 
c.战队id, c.战队名称, 
c.野战军id, c.野战军名称,
p.depart_id as 集团军id, 
p.dept_name as 集团军名称, 
p.parent_id from depart p left join (select 
c.depart_id as 战队id, c.dept_name as 战队名称, 
c.parent_id as 野战军id, p.dept_name as 野战军名称, 
p.parent_id as 集团军id from depart c left join depart p 
on c.parent_id = p.depart_id) c on
p.depart_id = c.集团军id 
where c.集团军id in (192,193,194,195)) d on d.战队id = u.department_id
where u.is_advisor = 1 and u.status = 2 and u.role_bitmap = 2 and u.user_id in (361463675400191,
379883902433278,
407468271351806,
356905262188542,
417600638337023,
 355608241117182,
 353412533794814,
 380565261742079,
 412118073544703,
 413525760014334,
 363075931160575,
 362432923652095,
 412118073538558,
 353361102213119,
 360531474315262,
 360280523831294,
 417778625433599,
 359319627943935,
 359671941226495,
 361789609988095,
 358970000924671,
 353722939506687,
 413525760010238,
 358726076264447,
 357640854124542,
 413525760018430,
 361090319157246,
 357953681631230,
 387817021204478,
 361201010692095,
 377936542824446,
 359371537174527,
 357017584777214,
 412118073548799,
 360201383997439,
 361484144719870,
 359870615240703,
 358092511418366,
 358800891191295,
 362570637692927,
 403021601058815,
 403021601058815,
 357939820822527,
 375790971363327,
 412118073546750,
 359515411439615,
 362601777397759,
 362540692815871,
 411925167800319,
 422174213283839,
 359951772010494,
 362384488134654,
 354191617808382,
 406759308523518) order by d.集团军id, 野战军id, 战队id;
```