# Intro
This spike is going to be a little introduction to some more-than-intermediate-but-not-quite-advanced concepts in SQL. We're going to use a lesson from Mod 4 Backend to setup our database and have some data ready to go. If you would like to go through the other lesson on your own, go to the following lesson plan:

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
During this spike, I will be running sql scripts from a .sql file. If you do not like running .sql files, feel free to run the scripts in terminal. If you are curious what all the other tables are, run the following in your imdb psql instance:

```
\dt
```

Make a file in the sql_deep_dive directory (I'm calling mine spike.sql). Add the following line in your file:

```
-- sample query
select title, budget, gross from movies;
```

Then in your psql instance, run the following line:

```
\i spike.sql
```

You should see the title, budget, and gross of all movies from the table. The "\i" stands for input, if you were curious.

# Views

Say we had a query we wanted to use over and over and in different files. For example, what if I needed to know directors and their movies all the time on my project? 

```
select d.name, m.title, m.year, m.id as movie_id
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

Still not impressed? What if I needed to know directors, their films, and their film genres?. Possible using this view? 

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

Go ahead and try to make a wild view using the tables from '\dt'. Can you try to join views to views?

# Functions / Stored Procedures
What is SQL's equivalent of a method?

The answer is functions and stored procedures.

Most SQL databases have an extension for procedural programming. If you ever go into a career dealing with databases often, you will most likey deal with procedural extensions. For the sake of time, we will only go over functions, but I highly recommend those interested in databases to research procedural language. Read this for PostgreSQL's distinction:

http://dba.stackexchange.com/questions/2357/what-are-the-differences-between-stored-procedures-and-stored-functions

## PostgreSQL's Basic Function Structure
PostgreSQL's functions may look daunting at first, but when we break them down we can easily create functions of our own.

```
CREATE FUNCTION one() RETURNS integer AS $$
    SELECT 1 AS result;
$$ LANGUAGE SQL;

SELECT one();

 one
-----
   1
```
Let's look at what's going on here. 
* "CREATE FUNCTION" is the declaration of the function
* "one()" is the name of the function
* "RETURNS integer" is the return type of the function
* $$ is known as dollar quoting, and this where our "block" begins
* LANGUAGE SQL means we are going to use SQL (it's weird, we can use other languages)

Let's write a simple math SQL function.
```
CREATE OR REPLACE FUNCTION add_stuff(integer, integer) RETURNS integer AS $$
    SELECT $1 + $2;
$$ LANGUAGE SQL;
```
Notice here we reference each parameter with a dollar sign. PostgreSQL loves dollar signs. I suppose we all love dollar signs.

Try running the function with various inputs. 
```
SELECT add_stuff(30, 20);
```

Let's make a function more relative to our data. What if we wanted to make a function that could update our movie title?

```
CREATE OR REPLACE FUNCTION update_movie(id integer, new_title varchar) RETURNS void AS
$$
    UPDATE movies
    SET title = $2
    WHERE ID = $1;
$$ LANGUAGE SQL;
```

In this example, we are creating a function that will take in a integer and a string (called varchar in SQL) that will return nothing (void). If you run a query, update the movie, then run the same query, do you notice that the data has changed?

```
select title from movies where id = 1;
SELECT update_movie(1, 'new title');
select title from movies where id = 1;
```
