# Schema Design Workshop

## Relational Databases
- More fundamental tech from the 1970s
- A way of organizing data
- Why do we need databases?  Why do we need persistence?

### Vocabulary
- *Tables* - Describe organization of data for models in the system (`Volunteers`, `Oranges`, `Teachers`, what have you) and store instances of models.  The math term is *Relations*
- *Rows* - Instances of models
- *Columns* - Attributes or fields of an instance.  Each field has a data type.  You can set constraints on a field's values
- *Primary key* - Unique key for each row
- *Foreign key* - A primary key used in another table.  This is how we make one-to-one and many-to-many relationships betweeen models
- *Schema* - The structure of the tables in a database, which is enforced programatically
- Like hyperlinked speadsheets
- You can reliably represent and efficiently retreive large amounts of complex data

## Connecting to the database
- Postgres runs as one server on your computer and is the interface to your databases
- Connect to postgres from the command line:
```bash
$ psql
```
- Show tables:
```sql
\dt
```
- Quit:
```sql
\q
```

## SQL (structured query language)

### Schema

#### Activerecord `schema.rb` file
```ruby
ActiveRecord::Schema.define(version: 20080906171750) do
  create_table "authors", force: true do |t|
    t.string   "name"
    t.datetime "created_at"
    t.datetime "updated_at"
  end
 
  create_table "products", force: true do |t|
    t.string   "name"
    t.text "description"
    t.datetime "created_at"
    t.datetime "updated_at"
    t.string "part_number"
  end
end
```

### 8 Ways to Get Databases to Really Listen
[Postgresql command reference](http://www.postgresql.org/docs/9.4/interactive/sql-commands.html)

#### Creating and dropping tables
[Create table reference](http://www.postgresql.org/docs/9.4/interactive/sql-createtable.html)

```sql
create table cats ( 
    name                varchar(80),
    whiskers            int,
    standoffish         boolean,
    neighborhood        varchar(80),
    date_of_birth       date,
    date_of_death       date
);
```

``` sql
drop table cats;
```

- You can also set constraints on data and modify tables (add, remove, change columns and data types)

#### Inserting rows
[Insert reference](http://www.postgresql.org/docs/9.4/interactive/sql-insert.html)

```sql
insert into cats values ('Mixture', 18, false, 'North of Ohlone Park', '2005-5-5', '2014-9-17');
```

```sql
insert into cats values ('Bigfoot', 16, false, 'North of Ohlone Park', '2010-3-27');
```

- You can also update and delete rows

#### Selecting aka retreiving data
[Select reference](http://www.postgresql.org/docs/9.4/interactive/sql-select.html)

```sql
select * from cats;
```

```sql
select name, standoffish from cats;
```

#### Additional clauses for a select statement
- `order by`: Return rows in a particular order

```sql
select * from cats order by name;
```

- `where`: Return certain rows based on if they meet a boolean condition

```sql
select * from cats where standoffish = false and whiskers > 17;
```

- `join`: Return data from multiple tables at a time

```sql
create table neighborhoods (
    name                varchar(80),
    lat                 real,
    long                real
);

insert into neighborhoods values ('North of Ohlone Park', 37.873890, -122.281152);

select cats.name, neighborhoods.lat, neighborhoods.long
    from cats, neighborhoods
    where cats.neighborhood = neighborhoods.name;
```

- That there is an inner join.  There are also left joins, right joins, outer joins, and more!
- [Visual representation of SQL joins](http://www.codeproject.com/Articles/33052/Visual-Representation-of-SQL-Joins)


## Schema Design and Relationships

- One-to-many: A human can have many cats 

```sql
create table humans (
  id                serial primary key,
  name              varchar(80),
  favorite_color    varchar(80)
);

insert into humans (name, favorite_color) values ('Masha', 'orange');

alter table cats add column human_id int default 1;
alter table cats add foreign key(human_id) references humans(id);

select cats.name, cats.whiskers,  
       humans.name, humans.favorite_color  
from cats, humans  
where  cats.human_id = humans.id
and humans.favorite_color = 'orange';
```

- Many-to-many: New information has come to light!  A cat can have many humans just as a human can have many cats

- We're going to need a join table, `cats_humans`, which captures the relationship between cats and humans by storing rows that contain both a cat's primary key and a human's primary key as `cat_id` and `human_id`
- Join tables as first-order entities: What relationship does the `cats_humans` table capture? 

```sql
create table friendships (cat_id int, human_id int);

alter table cats add column id serial primary key;

alter table friendships add foreign key(cat_id) references cats(id);
alter table friendships add foreign key(human_id) references humans(id);

insert into friendships values (1,1);
insert into friendships values (2,1);
```

```sql
select cats.name, humans.name
from cats
inner join friendships on cats.id = friendships.cat_id
inner join humans on friendships.human_id = humans.id
```

### Schema design group workshop


## Resources

- [Postgres docs](http://www.postgresql.org/docs/9.4/interactive/index.html)
- [Postgres tutorial](http://www.postgresql.org/docs/9.4/interactive/tutorial.html)
- [SQL command reference](http://www.postgresql.org/docs/9.4/interactive/sql-commands.html)
- [Visual representation of SQL joins](http://www.codeproject.com/Articles/33052/Visual-Representation-of-SQL-Joins)
