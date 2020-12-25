# soru 4)Gereken sorguyu çalıştırdıktan sonra sample.content_category ve sample.content_category_target tablolarının içerikleri birebir aynı olmalıdır!
```SQL
merge fatma_gezer.content_category t
using fatma_gezer.content_category_20201222_00_59 s
on t.id=s.id
when not matched by source then update set t.is_deleted=true
when not matched by target then insert(cdc_date,is_deleted,id,category) values(s.cdc_date,s.is_deleted,s.id,s.category)
When Matched and t.cdc_date <> s.cdc_date Or t.category<>s.category Then 
Update Set t.cdc_date=s.cdc_date,t.category=s.category;
```
## Alternatif sorgu//hash ile
```SQL
merge fatma_gezer.content_category t
using fatma_gezer.content_category_20201222_00_59 s
on farm_fingerprint(to_json_string(t.id))=farm_fingerprint(to_json_string(s.id))
WHEN NOT MATCHED  by source then update set t.is_deleted=true
when not matched by target then insert(cdc_date,is_deleted,id,category) values(s.cdc_date,s.is_deleted,s.id,s.category)
When Matched and SHA512(STRING(t.cdc_date, "UTC")) <>SHA512( STRING(s.cdc_date, "UTC"))  
Or  SHA512(t.category)<>SHA512(s.category) Then 
Update Set t.cdc_date=s.cdc_date,t.category=s.category;
```
