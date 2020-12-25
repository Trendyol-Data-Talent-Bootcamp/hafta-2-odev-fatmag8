# Soru 3) "sample.pageview": tablosunda 1 gün içerisinde trendyol.com a gelen tüm ziyaretlerin logu var.Bu çalışmada çıkarmak istediğimiz bilgi, günün her bir dakikası için aktif kullanıcı sayısının hesaplanması.
```SQL
-----Bu sorgu dakikadaki distinctleri hesaplıyor ama o 5dk içinde kişi çıkıp tekrar girdiyse kullanıcıyı iki defa sayıyor,ama alttakine göre daha hızlı çalışıyor. 
with solution as(with subsolution as (select  timestamp_trunc( view_ts,MINUTE) as minutime,count(distinct deviceid) as active_user
from fatma_gezer.pageview
group by minutime order by minutime)
select minutime,active_user,
first_value(active_user) over(order by  minutime rows between 4 preceding and 1 following) as four_previous,
first_value(active_user) over(order by  minutime rows between 3 preceding and 1 following) as three_previous,
first_value(active_user) over(order by  minutime rows between 2 preceding and 1 following) as two_previous,
first_value(active_user) over(order by  minutime rows between 1 preceding and 1 following) as previous,
row_number() over( order by minutime ) as row_number
from subsolution)

select minutime,(CASE WHEN row_number =1 THEN 0 ELSE previous END +
                Case when row_number=1 or row_number =2 then 0 else two_previous end+
                case when row_number=4 or row_number=5 then three_previous else 0 end+
                case when row_number=5 then four_previous else 0 end )+
                active_user as user_count
from solution;
```
##Alternatif sorgu
```SQL
CREATE TABLE IF NOT EXISTS fatma_gezer.pageview_minute(
  `view_period` TIMESTAMP,
  `active_user` INT64
);

DECLARE init_time TIMESTAMP DEFAULT '2020-03-02 23:55:00';

WHILE init_time < '2020-03-03 23:55:00' DO
  INSERT INTO  fatma_gezer.pageview_minute (view_period, active_user)
        SELECT timestamp_add(init_time,INTERVAL 5 MINUTE) as view_period,
        HLL_COUNT.extract(HLL_COUNT.init(deviceid,24)) as active_user FROM fatma_gezer.pageview
          WHERE view_ts BETWEEN init_time AND timestamp_add(init_time,INTERVAL 5 MINUTE);
  SET init_time = timestamp_add(init_time, INTERVAL 1 MINUTE);
END WHILE
```
