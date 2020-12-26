# Soru 1) 1980’den itibaren spor grubu bazında en çok madalya alan 1. 3. 5. ülkeyi bulalım.

```SQL
--Bu sorguda 1,3,5 alt alta geliyor ama bir CTE olduğundan daha hızlı. 
with allcountrysport as
(
  select
   Country,sport,count(1) s,
    row_number() over(PARTITION BY sport order by sport,count(1) desc ) as row_number,
  from `dsmbootcamp.sample.summer_medals`
  where year>1980 
  Group by Country,sport
)
SELECT
  Country as country,
  sport as sport,
  row_number
FROM allcountrysport
where row_number in(1,3,5)
;
```
## Soru 1 Alternatif
--Bu sorguda spor dalı,1,3,5 bir satırda geliyor,daha okunaklı ama iki CTE bu yüzden daha yavaş. 
```SQL
with result as(with allcountrysport as
(
  select
   Country,sport,count(1) s,
    row_number() over(PARTITION BY sport order by sport,count(1) desc ) as row_number,
   
  from `dsmbootcamp.sample.summer_medals`
  where year>1980 
  Group by Country,sport
)
SELECT
  sport as Sport,
  Country as first_champ,
  row_number as rw,
  last_value(Country) over(partition by sport order by sport rows between 1 preceding and 1 following) as third_champ,
  last_value(Country) over(partition by sport order by sport rows between 1 preceding and 2 following) as fifth_champ,
FROM allcountrysport
where row_number in(1,3,5))

select Sport,first_champ,third_champ,fifth_champ
from result
where rw=1
order by Sport
;
```
