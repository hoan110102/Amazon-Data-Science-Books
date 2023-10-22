1. Delete unnecessary columns
```
alter table dbo.amazon_books
drop column
	pages,
	weight,
	price_including_used_books,
	dimensions,
	ISBN_13,
	link,
	complete_link
```

2. Check and treat null value
```
select * from dbo.amazon_books
where name is null
	or author is null
	or price is null
	or avg_reviews is null
	or n_reviews is null
	or star5 is null
	or star4 is null
	or star3 is null
	or star2 is null
	or star1 is null
	or language is null
	or publisher is null
```
**Result:**
![Check null](https://github.com/hoan110102/Amazon-Data-Science-Books/assets/148353234/dda2d18d-a87f-49a5-a57c-2c37bd11a62e)

```
delete from dbo.amazon_books -- Delete rows
where author is null 
	or publisher is null 
	or language is null 
	or price is null
-----
update dbo.amazon_books -- Set '0' for null values
set
	n_reviews=0,
	avg_reviews=0,
	star5=0,
	star4=0,
	star3=0,
	star2=0,
	star1=0
where avg_reviews is null
	or n_reviews is null
	or star5 is null
	or star4 is null
	or star3 is null
	or star2 is null
	or star1 is null
```
**Result:**
![delete null](https://github.com/hoan110102/Amazon-Data-Science-Books/assets/148353234/7af9b4a8-57dc-4ce8-aded-c6b923b80c04)

3. Remove symbol '[]', change 'and' into '&' in author
```
with cte_author as (
select
	author,
	replace(replace(author, '[ ', ''), ']', '') as author1 --Remove '[]'
from dbo.amazon_books
)
select
	author,
	case
		when charindex('et al', author1, 1)=1 then author1
		when charindex('and', author1, 1)=0 then author1
		else replace(author1,',  and , ', ' & ')
	end as new_author --Change 'and' into '&'
from cte_author
-----
update dbo.amazon_books
set author=
	case
		when charindex('et al', replace(replace(author, '[ ', ''), ']', ''), 1)=1
			then replace(replace(author, '[ ', ''), ']', '')
		when charindex('and', replace(replace(author, '[ ', ''), ']', ''), 1)=0
			then replace(replace(author, '[ ', ''), ']', '')
		else replace(replace(replace(author, '[ ', ''), ']', ''),',  and , ', ' & ')
	end
-- Note: I just replace "author1" by "replace(replace(author, '[ ', ''), ']', '')" in update syntax
```
**Result:**
![remove ](https://github.com/hoan110102/Amazon-Data-Science-Books/assets/148353234/0ad8baba-f7db-4fe4-9af4-3a13c56898eb)

4. Round price and avg_reviews to 2 decimal places
```
select
	price,
	avg_reviews,
	round(price, 2) as new_price,
	round(avg_reviews, 2) as new_avg_price
from dbo.amazon_books
-----
update dbo.amazon_books
set
	price=round(price, 2),
	avg_reviews=round(avg_reviews, 2)
```
**Result:**
![round number](https://github.com/hoan110102/Amazon-Data-Science-Books/assets/148353234/eb838a6b-8918-4995-83fa-0e0cda077438)

5. Remove symbol '%' in stars and convert to numeric data type
```
-- Example with star5 and similar to the remaining columns
select
	star5,
	substring(star5, 1, len(star5)-1) as new_star5
from dbo.amazon_books
-----
update dbo.amazon_books
set star5=substring(star5, 1, len(star5)-1)
-----
alter table amazon_books
alter column star5 tinyint
```
**Result:**
![remove symbol](https://github.com/hoan110102/Amazon-Data-Science-Books/assets/148353234/c5449914-38e5-4403-a27a-d9fb3e1b4b4d)

6. Breaking out publisher into individual columns
```
/* 
	- Split the publication time from publisher into new column publish_date
	- Split the edition into new column edition, if there is no value 'edition' then set the value as 'N/A'
*/

-- Add 2 new columns
alter table dbo.amazon_books
add
	publish_date date,
	edition nvarchar(15)
-----
select
	publisher,
	case
		when charindex(';', publisher, 1)=0 
			then left(publisher, charindex('(', publisher, 1)-2)
		else left(publisher, charindex(';', publisher, 1)-1)
	end as new_publisher,
	try_convert(date, substring(publisher, charindex('(', publisher, 1)+1, len(publisher)-(charindex('(', publisher, 1)+1)), 103) as publish_date,
	case
		when charindex('edition', publisher, 1)=0 then 'N/A'
		else substring(publisher, charindex(';', publisher, 1)+2, (charindex('(', publisher, 1)-1)-(charindex(';', publisher, 1)+1))
	end as edition
from dbo.amazon_books
-----
update dbo.amazon_books
set
	publish_date=try_convert(date, substring(publisher, charindex('(', publisher, 1)+1, len(publisher)-(charindex('(', publisher, 1)+1)), 103),
	edition=
		case
			when charindex('edition', publisher, 1)=0 then 'N/A'
			else substring(publisher, charindex(';', publisher, 1)+2, (charindex('(', publisher, 1)-1)-(charindex(';', publisher, 1)+1))
		end,
	publisher=
		case
			when charindex(';', publisher, 1)=0 
				then left(publisher, charindex('(', publisher, 1)-2)
			else left(publisher, charindex(';', publisher, 1)-1)
		end
```
**Result:**
![split column](https://github.com/hoan110102/Amazon-Data-Science-Books/assets/148353234/1fba0eee-1ea0-48b0-8652-31cb73b68221)

**Here is the final result:__
![final](https://github.com/hoan110102/Amazon-Data-Science-Books/assets/148353234/e160a40d-af14-4ebc-8279-17e0c73c8f5e)
