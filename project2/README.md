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



----------------------------------------------------------------------------------

### ☕[BUY ME A COFFEE](https://www.buymeacoffee.com/thelifeimprovised)!

#### Feel the Buzz of Knowledge! If this repo added a spark to your day, why not buy me a coffee? Your support fuels the inspiration behind each project.

-----------------------------------------------------------------------------------

#### Follow me on LinkedIn and X to see all my updates related to technology and follow me on Facebook, Thread, Instagram, and YouTube to see all updates on travel and life.
#### Personal Blog: www.thelifeimprovised.com
#### Let's learn and grow together 💚
