# Intro
This spike is going to be a little introduction to some more intermediate concepts in SQL. We're going to use a lesson from Mod 4 Backend to setup our database and have some data ready to go. If you would like to go through the other lesson on your own, go to the following lesson plan:

http://backend.turing.io/module4/lessons/sql_deep_dive

# Setup
Create a folder named sql_deep_dive in your terminal and get the DB:
```
$ mkdir sql_deep_dive
$ cd sql_deep_dive
$ brew install wget
$ wget https://cl.ly/2k1q0J070W2S/download/imdb.pgsql
$ createdb imdb
$ psql imdb < imdb.pgsql
```
Then connect to the database and run a sample query:
```
$ psql imdb
psql> select * from movies where id=100 limit 1;
100|The Hobbit: An Unexpected Journey |182|303001229|http://www.imdb.com/title/tt0903624/?ref_=fn_tt_tt_1|PG-13|180000000|2012|English|1|2016-12-11 20:52:25.675416|2016-12-11 20:52:25.675416
```
Since are using PostsgreSQL, there aren't really SQL editors we can use where have complete control of our SQL. Therefore, we are going to make some .sql files and run them in our psql instance. During this spike, if you are curious what all the other tables are, run the following in your imdb psql instance:
```
\dt
```

Make a file in the sql_deep_dive directory (I'm just calling mine spike.sql). If you do not like running .sql files, feel free to run the scripts in terminal.

# Views

Add the following line in your file:

```
-- sample query
select title, budget, gross from movies;
```
Then in your psql instance, run the following line:
```
\i spike.sql
```
You should see the title, budget, and gross of all movies from the table. The "\i" stands for input, if you were curious.

Say we had a query we wanted to use over and over and in different files. For example, what if I needed to know directors and their movies all the time on my project? 
```
select d.name, m.title, m.year
from movies m
join directors d
on m.director_id = d.id;
```

One way is to make use a subquery and reference that every time, but another way is to make a view.

```
create or replace view director_titles as
  select d.id, d.name, m.title, m.year
  from movies m
  join directors d
  on m.director_id = d.id;

select * from director_titles;
```
Run the file in psql, and notice how selecting from the view returns everything from the original query. You can mess around with the view by selecting specific columns from the view e.g. name and title.

