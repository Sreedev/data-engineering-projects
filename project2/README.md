# DVD Rental Data modelling for datawarehouse

![star_schema_dvd_rental](https://github.com/Sreedev/data-engineering-projects/assets/1217856/3dee52ed-23f9-4dc1-ab4f-0df9acc681d9)

## Creation of tables in the schema
**Creating the dimDate table**
```
Create TABLE dimDate
(
	date_key integer NOT NULL PRIMARY KEY,
	date date NOT NULL,
	year smallint NOT NULL,
	quater smallint NOT NULL,
	month smallint NOT NULL,
	day smallint NOT NULL,
	week smallint NOT NULL,
	is_weekend boolean
);

```
**Creating the dimCutomer table**
```
Create TABLE dimCustomer
(
	customer_key SERIAL PRIMARY KEY,
	customer_id smallint NOT NULL,
	date date NOT NULL,
	first_name varchar(45) NOT NULL,
	last_name varchar(45) NOT NULL,
	email varchar(50),
	address varchar(50) NOT NULL,
	address2 varchar(45),
	district varchar(20) NOT NULL,
	city varchar(50) NOT NULL,
	country varchar(50) NOT NULL,
	postal_code varchar(10),
	phone varchar(20) NOT NULL,
	active smallint NOT NULL,
	create_date timestamp NOT NULL,
	start_date date NOT NULL,
	end_date date NOT NULL
);

```
**Creating the dimStore table**
```
Create TABLE dimStore
(
	store_key SERIAL PRIMARY KEY,
	store_id smallint NOT NULL,
	address varchar(50) NOT NULL,
	address2 varchar(45),
	district varchar(20) NOT NULL,
	city varchar(50) NOT NULL,
	country varchar(50) NOT NULL,
	postal_code varchar(10),
	manager_first_name varchar(45) NOT NULL,
	manager_last_name varchar(45) NOT NULL,
	start_date date NOT NULL,
	end_date date NOT NULL
);

```
**Creating the dimMovie table**
```
Create TABLE dimMovie
(
	movie_key SERIAL PRIMARY KEY,
	film_id smallint NOT NULL,
	title varchar(225) NOT NULL,
	description text,
	release_year year,
	language varchar(20) NOT NULL,
	original_language varchar(20) NOT NULL,
	rental_duration smallint NOT NULL,
	length smallint NOT NULL,
	rating varchar(5) NOT NULL,
	special_features varchar(60) NOT NULL
);
```
## Inserting data in to the tables
**Inserting data in to the dimDate table**
```
	INSERT INTO dimDate
	(date_key, date, year,quater,month,day,week,is_weekend)
	SELECT
	DISTINCT (TO_CHAR(payment_date :: DATE, 'yyyMMDD')::integer)as date_key,
	date(payment_date) as date,
	EXTRACT(year from payment_date) AS year,
	EXTRACT(quarter from payment_date) AS quater,
	EXTRACT(month from payment_date) AS month,
	EXTRACT(day from payment_date) AS day,
	EXTRACT(week from payment_date) AS week,
	CASE WHEN EXTRACT (ISODOW FROM payment_date) IN (6,7) THEN true ELSE false END
	FROM payment;

```
**Inserting data in to the dimCustomer table**
```
	INSERT INTO dimCustomer
	(customer_key,customer_id,first_name,last_name,email,address,address2,district,
	city,country,postal_code,phone,active,create_date,start_date,end_date)
	SELECT 
	c.customer_id as customer_key,
	c.customer_id,
	c.first_name,
	c.last_name,
	c.email,
	a.address,
	a.address2,
	a.district,
	ci.city,
	co.country,
	postal_code,
	a.phone,
	c.active,
	c.create_date,
	now() AS start_date,
	now() AS end_date
	FROM customer c
	JOIN address a ON (c.address_id=a.address_id)
	JOIN city ci ON (a.city_id=ci.city_id)
	JOIN country co ON (ci.country_id=co.country_id);
```
**Inserting data in to the dimMovie table**
```
	INSERT INTO dimMovie
	(movie_key,film_id,title,description,release_year,language,original_language,
	rental_duration,length,rating,special_features)

	SELECT 
	f.film_id as movie_key,
	f.film_id,
	f.title,
	f.description,
	f.release_year,
	l.name,
	l.name,
	f.rental_duration,
	f.length,
	f.rating,
	f.special_features
	FROM film f
	JOIN language l ON (f.language_id=l.language_id);

```
**Inserting data in to the dimStore table**
```
	INSERT INTO dimStore
	(store_key,store_id,address,address2,district,city,country,postal_code,
	manager_first_name,manager_last_name,start_date,end_date)

	SELECT 
	s.store_id as store_key,
	s.store_id,
	a.address,
	a.address2,
	a.district,
	ci.city,
	co.country,
	postal_code,
	st.first_name,
	st.last_name,
	now() as start_date,
	now() as end_date
	FROM store s
	JOIN address a ON (s.address_id=a.address_id)
	JOIN city ci ON (a.city_id=ci.city_id)
	JOIN country co ON (ci.country_id=co.country_id)
	JOIN staff st ON(a.address_id=st.address_id);
```
## Creating the fact table and inserting the data from the newly created tables
**Creating fact table**
```
CREATE TABLE factsales
(
	sales_key SERIAL PRIMARY KEY,
	date_key integer REFERENCES dimDate(date_key),
	customer_key integer REFERENCES dimCustomer (customer_key),
	movie_key integer REFERENCES dimMovie(movie_key),
	store_key integer REFERENCES dimStore(store_key),
	sales_amount numeric
);
```
**Interting data to fact table**
```
INSERT INTO factsales(date_key,customer_key,movie_key,store_key,sales_amount)
SELECT 
TO_CHAR(payment_date :: DATE, 'yyyMMDD')::integer as date_key,
p.customer_id as customer_key,
i.film_id as movie_key,
p.amount as sales_amount,
s.store_id as store_key

FROM payment p
JOIN rental r ON (p.rental_id=r.rental_id)
JOIN inventory i ON(r.inventory_id=i.inventory_id)
JOIN customer c ON (p.customer_id=c.customer_id)
JOIN address ad ON (ad.address_id=c.address_id)
JOIN store s ON(s.address_id=ad.address_id);
```

**Comparing the query timings of the transactional table with the datawarehouse we created**

```
SELECT f.title, EXTRACT(month FROM p.payment_date) as month, ci.city, sum(p.amount) as revenue
FROM payment p
JOIN rental r ON (p.rental_id = r.rental_id)
JOIN inventory i ON (r.inventory_id=i.inventory_id)
JOIN film f ON (i.film_id=f.film_id)
JOIN customer c ON (p.customer_id=c.customer_id)
JOIN address a ON (c.address_id=a.address_id)
JOIN city ci ON (a.city_id=ci.city_id)
group by (f.title, month, ci.city)
order by f.title, month,ci.city, revenue desc;
```
**The above query took 311 milliseconds to execute.**
**Comparing to previous query below query only took 255 milliseconds to execute. And that why datawarehouses are important**
```
SELECT dimmovie.title, dimdate.month, dimcustomer.city,sum(sales_amount) as revenue
FROM factsales
JOIN dimmovie ON (dimmovie.movie_key=factsales.movie_key)
JOIN dimdate ON (dimdate.date_key=factsales.date_key)
JOIN dimcustomer ON (dimcustomer.customer_key=factsales.customer_key)
group by (dimmovie.title, dimdate.month, dimcustomer.city)
order by dimmovie.title, dimdate.month,dimcustomer.city, revenue desc;
```
----------------------------------------------------------------------------------

### ☕[BUY ME A COFFEE](https://www.buymeacoffee.com/thelifeimprovised)!

#### Feel the Buzz of Knowledge! If this repo added a spark to your day, why not buy me a coffee? Your support fuels the inspiration behind each project.

-----------------------------------------------------------------------------------

#### Follow me on LinkedIn and X to see all my updates related to technology and follow me on Facebook, Thread, Instagram, and YouTube to see all updates on travel and life.
#### Personal Blog: www.thelifeimprovised.com
#### Let's learn and grow together 💚
