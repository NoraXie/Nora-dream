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
 
 
 
 select * from user where identity_card is null and is_advisor = 1 and status = 2 and external = 0;

select * from `advisor_transfer_records` 
-- where new_depart_id = 161;
where advisor_id = 385930834083839;
--  where `new_depart_name` = '顶尖勇士队';
-- where advisor_id = 417230979784703;

select user.telephone, tmp_card.phone, tmp_card.card, user_id, user.identity_card from user left join tmp_card on tmp_card.phone = user.telephone;

select * from user where real_name = '邱建伟';

select * from depart where dept_name = '争锋队';

select * from depart where depart_id = 348;

select * from user where department_id = 161 and status = 2 and is_advisor = 1;

select * from user where user_id = 418961196988414;

select * from deals where `receivable_money`< 0;

select * from project_fenxiaos where loupan_id = 343466073901058;

select * from tuiguang_targets;

select * from `transfer_records` tr join customer c on tr.customer_id = c.customer_id 
where tr.created_at <= '2017-11-15 00:00:00' and c.is_del = 0;

select * from  customer c join `transfer_records` tr on tr.customer_id = c.customer_id 
where tr.created_at <= '2017-11-15 00:00:00';

select count(distinct(customer_id)) from customer where created_at <= '2017-11-15 00:00:00';

select count(distinct(customer_id))from `transfer_records` where created_at <= '2017-11-15 00:00:00';

select * from `appointment` where customer_id = 340662704922623;

select * from `customer_follow_up` where customer_id = 340662704922623;

select * from customer where customer_id = 340662704922623;

select * from deals where item_id is null;

select * from area c left join area p
on c.`parent_id` = p.`area_id` 
where c.parent_id = 263804351010818;

select distinct(item.area_id), area.`area_name` from item join area 
on item.area_id = area.`area_id` where item.area_id lis not null and area.parent_id = -1;

select * from area where area_id = 316028062087167;

select * from appointment where customer_id is null and status in (5,6,10);

select * from appointment_item where created_at is null;

select identity_card, user_id, real_name from user where user_id = 417778625433599;
```


> BDP_SQL

```sql
TEMP ABC = 
SELECT count(distinct 客户id) as 跟进量
FROM 顾问线索跟进合表
WHERE 查询日期 >= '2017-11-01 00:00:00' and 查询日期 <= '2017-11-28 23:59:59'
and 顾问id = 419028448167934  
GROUP BY 顾问id
TEMP D = 
SELECT count(distinct 被跟进号码) as 跟进量
FROM 客户跟进表
WHERE 跟进日期 >= '2017-11-01 00:00:00' and 跟进日期 <= '2017-11-28 
and 跟进人id = 404158480396287 
GROUP BY 跟进人id
OUTPUT
SELECT count(distinct 上户跟进量) as 跟进量
FROM 顾问线索跟进合表
WHERE 查询日期 >= '2017-11-01 00:00:00' and 查询日期 <= '2017-11-28 23:59:59'
and 顾问id = 419028448167934  
GROUP BY 顾问id
```

```
select * from customer where customer_id =425517525063678;

select * from user where 
-- real_name like '%司';
user_id = 412474054924286;

select * from `customer_sourses` ;

select * from depart where 
-- dept_name = '雪狼队';
parent_id = 286;

select c.customer_id, b.customer_id from customer c
left join 
(select customer_id from customer 
where date(created_at) >= '2017-11-20' and date(created_at) <= '2017-11-26'
and sourse_level1_id = 1
and is_del = 0 
and city_id = 263804351010818) b
on b.customer_id = c.customer_id
where c.customer_id in (424656824461310,
424849434769406,
424842627084287,
425162782932991,
424860436940798,
424501819719679,
424653903898622,
424832649240575,
425020662589438,
424792782854142,
425202090113022,
424618084358142,
425506011766783,
425048750090238,
424967716886526,
424887285766142,
424490791448575,
425370747338751,
424983435280383,
424639893528575,
424680179591166,
425144494104575,
424641556049919,
425561253949439,
424443621423102,
425326172399614,
424638570518526,
425149227110399,
425341360328703,
425144379410430,
424668979021822,
425325809891326,
425179439386623,
424630599712767,
424689428398079,
424678531205118,
424618194954238,
425144434710526,
424961353736191,
424793616410622,
424817422614527,
425533463402494,
425150910574591,
425163942092799,
425009875738623,
424661901094911,
424691620014078,
424814022594559,
424499706150911,
424967860232191,
425325957353471,
424616404299775,
424833581103102,
424627742744575,
425326100717567,
425572480053247,
424623996952575,
424473999433727,
425321777225727,
425013334063102,
424623933460479,
424792895498238,
424967653396479,
424499624228862,
424838746169343,
424798402664446,
425513210558462,
424978774609919,
424624066590718,
424793499672575,
424995194920959,
424502258006014,
425189353480191,
424838682673151,
424839942242303,
424489583116287,
424802863251455,
425511711410174,
425504317456383,
425365637146622,
425005072429055,
425182050631679,
425331644553214,
424805490872319,
425010647050238,
425503268870142,
425144561690622,
424798748708863,
425499675203582,
424634202191870,
424482714025983,
425504069644287,
425336389914623,
424850993340415,
425182879770622,
424792838152191,
425504231438334,
425004640335870,
424792954892287,
425073455736834,
425556056072191,
425341112516607,
425336637616127,
424830011428863,
424972294668287,
424650420262910,
424979604051966,
425557920792575,
425191029082110,
424795871244287,
424623509516287,
424443195430910,
425504477526014,
425504849965055,
424631496710142,
425325914343422,
425358125377534,
424631443460095,
424499198238719,
424523246280703,
425002311735294,
425504714788863,
424976038455294,
425017135902719,
424969433202687,
424690358280191,
425336465692670,
424456487131134,
424624795037695,
424985284622334,
424457306335230,
425139189770238,
424656969422846,
425507357319167,
424668575561726,
424966936580095,
424623372296191,
424837578776575,
425358489929726,
425167706159102,
425318748217343,
425336473886719,
424688457639934,
424489587216383,
425327071236095,
424971645448191,
425494714935294,
425504379484158,
424838826045439,
424697217032191,
425511238313982,
425400572450814,
424967780356095,
424532599500799,
424487817414654,
424657575946238,
424626427957247,
424454140012542,
424622044538878,
424827232270334,
424502674098174,
425504415762430,
424624027676671,
424487188971519,
424990103574526,
424612853768191,
425517525063678,
424670978054142,
424487708860415,
425505425465343,
424681617297407,
424460290275326,
425505120309247,
424473946183678,
424684541960191,
425550700656620,
425505464379390,
424791373821950,
424821772302334,
424502274392063,
424636239908863,
424842524837886,
425017830184958,
425558699044863,
424487764164607,
424454193264638,
424846893088767,
424675854417919,
425011019792383,
424824445059071,
425505349687294,
425183166496767,
424670564358142,
424623593486334,
424618260492287,
425505075251198,
424502182225918,
424838408235006,
424684638218238,
424443301928959,
425572809785343,
425503885320191,
425144231981055,
425504002058238,
424995137574910,
424463276269567,
424878641162238,
424493779515390,
425335099447294,
425147457619966,
425005017131006,
424797538392062,
424998910683134,
424622155806718,
424471847188478,
424443502635006,
424631934437374,
424443570221055,
425012676644863,
425345335394303,
425000828968959,
424995055652863,
424680534173694,
425360322590718,
424618135560191,
424618301456383,
424631322634238,
425504657438718,
424630511646718); 


select customer_id from customer 
where date(created_at) >= '2017-11-20' and date(created_at) <= '2017-11-26'
and sourse_level1_id = 1
and is_del = 0 
and city_id = 263804351010818;
-- group by city_id;




	
	commit;


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

select u.user_id, u.department_id, b.* from user u left join (select 
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
order by c.集团军id, c.野战军id, c.战队id desc) b
on b.战队id = u.department_id

where user_id in (422491365650434,
422494436106242,
422507818330114,
422509758367746,
422700843599874,
423944135794690,
377739514044415,
418961196984318,
422702992232450,
423945226315778,
418797579775999,
418667318468607,
420015371597826,
348034166663166,
362693404526590,
375790971363327,
376542537017342,
377588390451198,
423947643516930,
418961196980222,
378079181207550,
418961196986367,
404158480396287,
407294036463615,
418961196988414,
388018807746559,
407982826637310,
410853566468095,
419677731688447,
419677731690494,
420236933713922,
420367813511170,
420240188145666,
417778625437695,
419028448167934,
417778625433599,
418081854154750,
419158973075455,
418477236709374,
417230979784703,
417778625449983,
420371028049922,
418797579780095,
417778625441791,
420371279179778,
417778625435646,
417778625439742,
418084432619518,
421306138382338,
418435139405822,
421139592904706,
418480997650431,
418486582990846,
421647413909506,
418993309089791,
422191153438722,
422314561996802,
423559531542530,
422491365650434,
419665707298815,
419665707294719,
362693404526590,
423947643516930,
423945226315778,
422702992232450,
423944135794690,
422700843599874,
419665707292670,
419158973075455,
419028448167934,
418993309089791,
418961196988414,
418961196986367,
418961196984318,
418797579780095,
418797579775999,
418486582990846,
418477236709374,
418435139405822,
418084432619518,
418081854154750,
417778625433599,
417778625437695,
417778625449983,
407982826637310,
404158480396287,
388018807746559,
377739514044415,
376542537017342,
375790971363327,
419665707292670,
419677731680255,
419677731686398,
421095667599362,
420225889548290,
420254434086914,
420414956853250,
421075118534658,
421270129778690,
421323250399234,
421427403655170,
377588390451198,
378079181207550,
422494436106242,
422507818330114,
422509758367746,
348034166663166,
407294036463615,
410853566468095,
417230979784703,
417778625441791,
418480997650431,
419665707298815,
418961196980222,
418667318468607,
417778625435646,
417778625439742,
419665707294719);

select c.user_id, c.customer_id, c.created_at, c.sourse_level1_name,u.department_id, b.*from customer c
left join user u on u.user_id = c.user_id
left join (select 
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
order by c.集团军id, c.野战军id, c.战队id desc) b
on b.战队id = u.department_id 
where c.sourse_level1_id = 1
and c.customer_id in (425733341188095,
425725790928895,
425692641531902,
425742291048446,
425725749964799,
425717502490622,
425734237075454,
425741879379966,
425740714043390,
425733710712831,
425720068184062,
425742217316350,
425733595326462,
425717635616767,
425740949569535,
425692453107710,
425734809823230,
425741695049727,
425719951968254,
425734200209407,
425717768744959,
425742186594303,
425715349977086,
425720205404158,
425733186398206,
425689766105087,
425734932705279,
425734161293311,
425741944920062,
425725922004991,
425734855587838,
425733559142398,
425689559242750,
425675446276095,
425717895223294,
425733299042302,
425721399939071,
425740677146622,
425721467525118,
425679233036287,
425734484183038,
425725882490879,
425717547548671,
425734083465215,
425734046599166,
425734445268991,
425716023347198,
425742084188158,
425725595768830,
425733673846782,
425692502261759,
425734892453887,
425721516679167,
425733827454974,
425733225312255,
425733913475070,
425715945521151,
425734120331262,
425715565031423,
425733634926590,
425740881954814,
425733864321023,
425735058348030,
425715435995135,
425741801551870,
425733505892351,
425756805128191,
425740816416767,
425748959361022,
425741729867774,
425742116958207,
425734410450942,
425733143388159,
425733261500415,
425741840465919,
425721617037310,
425717457432575,
425741762637823,
425692547319806,
425734363344895,
425690085613567,
425689853964287,
425742256230399,
425733747578878,
425740986435582,
425740781598718,
425683673120767,
425717590558718,
425719984738303,
425720137818111,
425734968233983,
425742149728254,
425740748828671,
425717723686910,
425711501729791,
425741914198015,
425740914751486,
425734519711742,
425689411780607,
425725643464703,
425734286229503,
425721571977214,
425720285816830,
425692401905663,
425725694666750,
425689862148095,
425689278652415,
425734321047550,
425716058165247,
425692346607614,
425733788540927,
425741977690111,
425725960316926,
425741656135678,
425689934053375,
425733947598846,
425725835986942,
425692596473855);

select sum(消费) as 总消费 from (
select 1 as 列, city_id, sum(money) as 消费 from `seo_promotions`
where date(promote_at) >= '2017-11-01' and date(promote_at) <= '2017-11-27'
group by city_id ) b
group by b.列;

select * from seo_promotions;

select * from transfer_records limit 1;

select count(distinct phone) as 去重手机号, count(phone) as 手机号, city_id 
from transfer_records where clue_type = 1
group by city_id;

select customer_id, user_id, city_id, sourse_level1_id, sourse_level1_name from customer where city_id is null
and created_at >= '2017-11-27 00:00:00';

select count(*) from customer;

select * from user where real_name = '李耀辉';

select count(distinct phone) as 去重电话 from clue_promotions
where created_at >= '2017-11-01';

select count(distinct phone) from capitals 
where created_at >= '2017-11-01';

select * from area where area_id = 331803546648575;

```

# 2017-12-05
> 存储过程

```

CREATE DEFINER=`root`@`%` PROCEDURE `proc_generate_schedules_old`()
BEGIN
	DECLARE cityCount int;/*接收城市数量*/
	DECLARE cityId BIGINT (20);/*接收城市id*/
	DECLARE achieveRule int; /*规则*/
	DECLARE timeBasis int; /*时间依据*/
	DECLARE advisorNum int; /*排备数量*/
	DECLARE bgnTime datetime;
	DECLARE endTime datetime;
	DECLARE userId BIGINT(20);/*接收客户的顾问id*/
	DECLARE deptId int;
	DECLARE dealCount int;
	DECLARE userCount int;
	DECLARE branchId int;
	DECLARE weekId int;
	DECLARE weekOne datetime;
	DECLARE primaryId BIGINT(20);/*接收主键ID*/
	DECLARE i int; /*城市循环参数*/
	DECLARE j int; /*未到访客户循环参数*/
	DECLARE workCategory int; /*当天是否为工作日，1为工作日*/
	DECLARE noWorkCount int; /*非工作日天数*/
	/**/DECLARE t_error INTEGER DEFAULT 0;  
	DECLARE EXIT HANDLER FOR SQLEXCEPTION SET t_error=1;
	START TRANSACTION;#开始事务
	/*先查询城市数量再根据城市循环*/
	select subdate(date_add(CURDATE(), INTERVAL -1 day),date_format(date_add(CURDATE(), INTERVAL -1 day),'%w')-1) INTO weekOne;
	SELECT COUNT(d.city_id) INTO cityCount FROM depart d WHERE d.parent_id = 186 AND `status` =0 AND (role_bitmap & 2) !=0;
	SELECT week_id INTO weekId FROM weeks WHERE start_date LIKE CONCAT('%',date_add(CURDATE(), INTERVAL 1 day),'%');
	set i = 0;
	while i < cityCount DO
		SET dealCount = 0;
		SELECT d.city_id, d.depart_id INTO cityId, branchId FROM depart d WHERE d.parent_id = 186 AND `status` =0 AND (role_bitmap & 2) !=0 LIMIT i,1;
		
		-- 查询大定信息
		SELECT COUNT(deal_id) INTO dealCount FROM deals d WHERE DATE_FORMAT(d.created_at,'%Y-%m-%d') >= weekOne AND DATE_FORMAT(d.created_at,'%Y-%m-%d') <= DATE_FORMAT(CURDATE(),'%Y-%m-%d') AND step = 4 AND city_id = cityId ORDER BY create_date ASC;
		SET j = 0;
		WHILE j < dealCount do
			SELECT user_id, department_id INTO userId, deptId FROM deals d WHERE DATE_FORMAT(d.created_at,'%Y-%m-%d') >= weekOne AND DATE_FORMAT(d.created_at,'%Y-%m-%d') <= DATE_FORMAT(CURDATE(),'%Y-%m-%d') AND step = 4 AND city_id = cityId ORDER BY create_date ASC LIMIT j,1;
			SELECT idWork() INTO primaryId;
			INSERT INTO schedules(created_at,updated_at,schedule_id,depart_id,user_id,order_id,city_id,`status`,week_id)
			VALUES(NOW(),NOW(),primaryId,deptId,userId,j+1,cityId,1,weekId);
			SET j = j + 1;
		END WHILE;
		-- 打定后面加上公司全部顾问
		SET j = 0;
		SELECT COUNT(user_id) INTO userCount FROM `user` WHERE branch_id = branchId AND `status` = 2 AND (role_bitmap & 2) !=0 ORDER BY create_date ASC;
		WHILE j < userCount do
			SET userId = NULL;
			SELECT user_id, department_id INTO userId, deptId  FROM `user` WHERE branch_id = branchId AND `status` = 2 AND (role_bitmap & 2) !=0 ORDER BY create_date ASC LIMIT j,1;
			SELECT idWork() INTO primaryId;
      IF userId IS NOT NULL THEN
				INSERT INTO schedules(created_at,updated_at,schedule_id,depart_id,user_id,order_id,city_id,`status`,week_id)
				VALUES(NOW(),NOW(),primaryId,deptId,userId,j+1+dealCount,cityId,1,weekId);
	    END IF;
			SET j = j + 1;
		END WHILE;

		UPDATE schedules SET `status`=2 WHERE `status`=1 AND city_id = cityId AND created_at < CURDATE();
		UPDATE schedules SET `status`=1 WHERE `status`=3 AND city_id = cityId AND created_at > DATE_ADD(SYSDATE(),INTERVAL -1 WEEK) AND created_at < CURDATE();
		SET i = i + 1;
	/*查询是否是工作日*/
	END WHILE;
	
	/*判断执行结果*/
	IF t_error = 1 THEN  
		ROLLBACK;  
	ELSE  
		COMMIT;  
	END IF; 
END;;


CREATE DEFINER = root@% PROCEDURE = proc_deal_user() 
BEGIN
	DECLARE emptyDealidCount int;/*接收城市数量*/
	DECLARE primaryId BIGINT(20);
	DECLARE i int;
	/**/DECLARE t_error INTEGER DEFAULT 0;  
	DECLARE EXIT HANDLER FOR SQLEXCEPTION SET t_error=1;
	START TRANSACTION;#开始事务
	
	SELECT COUNT(b.deal_id) INTO emptyDealidCount From (select d.deal_id, d.user_id from deal_user du 
			right join deals d on du.deal_id  = d.deal_id
			where d.step = 4
			and d.status not in (2,4,6)
			and du.deal_id is null)	b;
			
	
	/*判断执行结果*/
	IF t_error = 1 THEN  
		ROLLBACK;  
	ELSE  
		COMMIT;  
	END IF; 
END;;


select d.deal_id, d.user_id from deal_user du 
right join deals d on du.deal_id  = d.deal_id
where d.step = 4
and d.status not in (2,4,6)
and du.deal_id is null;
```



```

-- 有效带看量
select count(appointment_id) from appointment 
where status in(5,6,10);

-- 签约客户量
select count(distinct customer_id) as 签约客户量 from deals
where step = 6 
and status not in (2,4,6,8);

-- 大定客户量
select count(distinct customer_id) as 大定客户量 from deals 
where step = 4
and status not in (2,4,6,8);

-- 大定成交额与全款金额
select sum(payment), sum(full_payment)
from deals 
where step = 4 
and status not in (2,4,6,8);

-- 签约佣金与全款金额
select sum(payment), sum(full_payment)
from deals 
where step = 6
and status not in (2,4,6,8);

```