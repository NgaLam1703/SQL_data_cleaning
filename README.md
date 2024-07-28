# SQL_data_cleaning

```sql
	SELECT *
	FROM club_member_info LIMIT 10;
```
**The result:**

|full_name|age|martial_status|email|phone|full_address|job_title|membership_date|
|---------|---|--------------|-----|-----|------------|---------|---------------|
|addie lush|40|married|alush0@shutterfly.com|254-389-8708|3226 Eastlawn Pass,Temple,Texas|Assistant Professor|7/31/2013|
|      ROCK CRADICK|46|married|rcradick1@newsvine.com|910-566-2007|4 Harbort Avenue,Fayetteville,North Carolina|Programmer III|5/27/2018|
|Sydel Sharvell|46|divorced|ssharvell2@amazon.co.jp|702-187-8715|4 School Place,Las Vegas,Nevada|Budget/Accounting Analyst I|10/6/2017|
|Constantin de la cruz|35||co3@bloglines.com|402-688-7162|6 Monument Crossing,Omaha,Nebraska|Desktop Support Technician|10/20/2015|
|  Gaylor Redhole|38|married|gredhole4@japanpost.jp|917-394-6001|88 Cherokee Pass,New York City,New York|Legal Assistant|5/29/2019|
|Wanda del mar       |44|single|wkunzel5@slideshare.net|937-467-6942|10864 Buhler Plaza,Hamilton,Ohio|Human Resources Assistant IV|3/24/2015|
|Joann Kenealy|41|married|jkenealy6@bloomberg.com|513-726-9885|733 Hagan Parkway,Cincinnati,Ohio|Accountant IV|4/17/2013|
|   Joete Cudiff|51|divorced|jcudiff7@ycombinator.com|616-617-0965|975 Dwight Plaza,Grand Rapids,Michigan|Research Nurse|11/16/2014|
|mendie alexandrescu|46|single|malexandrescu8@state.gov|504-918-4753|34 Delladonna Terrace,New Orleans,Louisiana|Systems Administrator III|3/12/1921|
| fey kloss|52|married|fkloss9@godaddy.com|808-177-0318|8976 Jackson Park,Honolulu,Hawaii|Chemical Engineer|11/5/2014|


## Copy Table
### Create a new table for cleaning

```sql
    CREATE TABLE club_member_info_cleaned (
	full_name VARCHAR(50),
	age INTEGER,
	martial_status VARCHAR(50),
	email VARCHAR(50),
	phone VARCHAR(50),
	full_address VARCHAR(50),
	job_title VARCHAR(50),
	membership_date VARCHAR(50)
    );
```

### Copy all values from club_member_info table

```sql
	insert into club_member_info_cleaned
	select * from club_member_info;
```

## Full_name column clean
 The full_name column contains a lot of different points. Idea:
- Cut white blanks
- Convert all names to text
- Replace name blanks with NULL

### Trim whitespaces and to uppercase

```sql
	SELECT TRIM(UPPER(full_name)) Full_name
	FROM club_member_info_cleaned cmic;

	UPDATE club_member_info_cleaned SET full_name = TRIM(UPPER(full_name));
```

### Replace name blanks with NULL

```sql
	UPDATE club_member_info_cleaned
		SET full_name = CASE WHEN full_name = '' THEN NULL ELSE full_name END;
```

## Age cleaning

```sql
	SELECT COUNT(*) FROM club_member_info_cleaned cmic 
	WHERE age NOT BETWEEN 18 AND 90 OR age IS NULL ;

	|COUNT(*)|
	|--------|
	|18|
```

#### Fix replacing all wrong values using the Median

```sql
	UPDATE club_member_info_cleaned
		SET age = (SELECT ROUND(MEDIAN(age)) FROM club_member_info_cleaned)
		WHERE age NOT BETWEEN 18 AND 90 OR age ISNULL;
```
## Marital status cleaning

There are only 3 possible values for marital status:
- single
- married
- divorced

There is still an empty value in the Marital_status column

First I will use trim and lower to standardize the font

```sql
	UPDATE club_member_info_cleaned SET martial_status = TRIM(LOWER(martial_status));
```
Start cleaning empty data

```sql
	UPDATE club_member_info_cleaned 
		SET martial_status = NULL 
		WHERE NOT martial_status IN ('single','married', 'divorced') OR  martial_status ISNULL ;
```

## Other columns

First use the Trim and Lower functions to adjust the text shape accordingly

```sql
	UPDATE club_member_info_cleaned SET 
		phone = TRIM(LOWER(phone)) ,
		full_address = TRIM(LOWER(full_address)) ,
		job_title = TRIM(LOWER(job_title)) ,
		membership_date = TRIM(LOWER(membership_date)) ;
```
Replace all empty values with `NULL`

```sql
	UPDATE club_member_info_cleaned SET 
		phone = NULL WHERE phone = '';
	UPDATE club_member_info_cleaned SET 
		full_address = NULL WHERE full_address = '';
	UPDATE club_member_info_cleaned SET 
		job_title = NULL WHERE job_title = '';
	UPDATE club_member_info_cleaned SET 
		membership_date = NULL WHERE membership_date = '';
```

## This is is the cleaned data

```sql
SELECT * FROM club_member_info_cleaned cmic LIMIT 10;
```
**The result:**

|full_name|age|martial_status|email|phone|full_address|job_title|membership_date|
|---------|---|--------------|-----|-----|------------|---------|---------------|
|ADDIE LUSH|40|married|alush0@shutterfly.com|254-389-8708|3226 eastlawn pass,temple,texas|assistant professor|7/31/2013|
|ROCK CRADICK|46|married|rcradick1@newsvine.com|910-566-2007|4 harbort avenue,fayetteville,north carolina|programmer iii|5/27/2018|
|SYDEL SHARVELL|46|divorced|ssharvell2@amazon.co.jp|702-187-8715|4 school place,las vegas,nevada|budget/accounting analyst i|10/6/2017|
|CONSTANTIN DE LA CRUZ|35||co3@bloglines.com|402-688-7162|6 monument crossing,omaha,nebraska|desktop support technician|10/20/2015|
|GAYLOR REDHOLE|38|married|gredhole4@japanpost.jp|917-394-6001|88 cherokee pass,new york city,new york|legal assistant|5/29/2019|
|WANDA DEL MAR|44|single|wkunzel5@slideshare.net|937-467-6942|10864 buhler plaza,hamilton,ohio|human resources assistant iv|3/24/2015|
|JOANN KENEALY|41|married|jkenealy6@bloomberg.com|513-726-9885|733 hagan parkway,cincinnati,ohio|accountant iv|4/17/2013|
|JOETE CUDIFF|51|divorced|jcudiff7@ycombinator.com|616-617-0965|975 dwight plaza,grand rapids,michigan|research nurse|11/16/2014|
|MENDIE ALEXANDRESCU|46|single|malexandrescu8@state.gov|504-918-4753|34 delladonna terrace,new orleans,louisiana|systems administrator iii|3/12/1921|
|FEY KLOSS|52|married|fkloss9@godaddy.com|808-177-0318|8976 jackson park,honolulu,hawaii|chemical engineer|11/5/2014|

# Check for duplicate data
```
SELECT COUNT(phone),
	COUNT(DISTINCT(phone)) 
	FROM club_member_info_cleaned cmic;
```

|COUNT(phone)|COUNT(DISTINCT(phone))|
|------------|----------------------|
|2001|1991|

There is duplicate data in the phone column, we will start by removing duplicate data.

```
SELECT *
FROM club_member_info_cleaned a
JOIN (SELECT phone, email FROM club_member_info_cleaned 
GROUP BY phone) b
ON a.email = b.email
LIMIT 10;
```
|full_name|age|martial_status|email|phone|full_address|job_title|membership_date|phone|email|
|---------|---|--------------|-----|-----|------------|---------|---------------|-----|-----|
|ADDIE LUSH|40|married|alush0@shutterfly.com|254-389-8708|3226 eastlawn pass,temple,texas|assistant professor|7/31/2013|254-389-8708|alush0@shutterfly.com|
|ROCK CRADICK|46|married|rcradick1@newsvine.com|910-566-2007|4 harbort avenue,fayetteville,north carolina|programmer iii|5/27/2018|910-566-2007|rcradick1@newsvine.com|
|SYDEL SHARVELL|46|divorced|ssharvell2@amazon.co.jp|702-187-8715|4 school place,las vegas,nevada|budget/accounting analyst i|10/6/2017|702-187-8715|ssharvell2@amazon.co.jp|
|CONSTANTIN DE LA CRUZ|35||co3@bloglines.com|402-688-7162|6 monument crossing,omaha,nebraska|desktop support technician|10/20/2015|402-688-7162|co3@bloglines.com|
|GAYLOR REDHOLE|38|married|gredhole4@japanpost.jp|917-394-6001|88 cherokee pass,new york city,new york|legal assistant|5/29/2019|917-394-6001|gredhole4@japanpost.jp|
|WANDA DEL MAR|44|single|wkunzel5@slideshare.net|937-467-6942|10864 buhler plaza,hamilton,ohio|human resources assistant iv|3/24/2015|937-467-6942|wkunzel5@slideshare.net|
|JOANN KENEALY|41|married|jkenealy6@bloomberg.com|513-726-9885|733 hagan parkway,cincinnati,ohio|accountant iv|4/17/2013|513-726-9885|jkenealy6@bloomberg.com|
|JOETE CUDIFF|51|divorced|jcudiff7@ycombinator.com|616-617-0965|975 dwight plaza,grand rapids,michigan|research nurse|11/16/2014|616-617-0965|jcudiff7@ycombinator.com|
|MENDIE ALEXANDRESCU|46|single|malexandrescu8@state.gov|504-918-4753|34 delladonna terrace,new orleans,louisiana|systems administrator iii|3/12/1921|504-918-4753|malexandrescu8@state.gov|
|FEY KLOSS|52|married|fkloss9@godaddy.com|808-177-0318|8976 jackson park,honolulu,hawaii|chemical engineer|11/5/2014|808-177-0318|fkloss9@godaddy.com|

```
SELECT COUNT(phone),
COUNT(DISTINCT(phone)) 
FROM (SELECT phone, email FROM club_member_info_cleaned 
GROUP BY phone) b;
```
|COUNT(phone)|COUNT(DISTINCT(phone))|
|------------|----------------------|
|1991|1991|
