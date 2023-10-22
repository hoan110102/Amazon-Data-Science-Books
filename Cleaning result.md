1. Delete unnecessary columns
`alter table dbo.amazon_books
drop column
	pages,
	weight,
	price_including_used_books,
	dimensions,
	ISBN_13,
	link,
	complete_link`

2. Check and treat null value
`select * from dbo.amazon_books
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
	or publisher is null`
4. Remove symbol '[]', change 'and' into '&' in author
5. Round price and avg_reviews to 2 decimal places
6. Remove symbol '%' in stars and convert to numeric data type
7. Breaking out publisher into individual columns
