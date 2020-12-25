# Soru 2) 1980’den itibaren herhangi bir spor grubunda üst üste 3 veya daha fazla madalya almış atletleri bulalım.
```SQL
with champions as (
select  athlete,year,last_value(year) over(partition by athlete order by year rows between  1 preceding and 1 following) as next,
first_value(year) over(partition by athlete order by year rows between  1 preceding and 1 following) as previous
from fatma_gezer.summer_medals where year > 1980 and medal is not null
group by 1,2)



select  athlete as Athlete
from champions
where next-previous=8 and year!=previous and year!=next
group by athlete
order by athlete;
```
