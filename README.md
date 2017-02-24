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
  select d.name, m.title, m.year, m.id as movie_id
  from movies m
  join directors d
  on m.director_id = d.id;

select * from director_titles;
```
Run the file in psql, and notice how selecting from the view returns everything from the original query. You can mess around with the view by selecting specific columns from the view e.g. name and title.

Still not impressed? Imagine we have a situation where we have this query we use all the time, but now we need to join the view to another table. Possible? 

```
select dt.*, g.name as genre_name
from director_titles dt
join genre_memberships gm
on dt.movie_id = gm.movie_id
join genres g
on gm.genre_id = g.id;
```
What's happening here? The view still contains all the columns from our original query, including movies.id aliased as movie_id. Since we have the movie id, we can join to genre_memberships and again to genres in order to pull out the genre name. 

In this example, you may think it's a little convoluted and unnecessary to use a view since you can just use the original query. However, when a subquery becomes 10-20 lines long with multiple tables joined, you don't want to rewrite that every time. 




