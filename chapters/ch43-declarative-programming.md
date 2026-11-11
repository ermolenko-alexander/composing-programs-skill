# Section 4.3: Declarative Programming

## Core Idea
Data values are often stored in large repositories called databases: a data store plus an interface for retrieving and transforming values. Each stored value is a **record**; records with similar structure are grouped into **tables**. Records are retrieved and transformed using queries in a query language — by far the most ubiquitous is SQL (pronounced "sequel").

SQL is a **declarative programming language**: statements do not describe computations directly, but describe the *desired result* of some computation. It is the role of the **query interpreter** of the database system to design and perform a computational process to produce that result. This differs substantially from the procedural paradigm of Python or Scheme, where computational processes are described directly by the programmer. The text covers a small subset of SQL as implemented in SQLite, and shows that under the hood a select statement is just composed sequence operations: join (product) → filter → sort → project.

## Formal Rules

### Select statement syntax and execution
Full form (not including unions):
```
with [tables] select [columns] from [names] where [condition] order by [order]
```
- `[columns]`: comma-separated expressions, each evaluated **for each row** of the input; column-description expressions share many properties with Python (infix `+`, `%`, built-ins `abs`, `round`, parentheses); a name evaluates to that column's value in the row being projected. `as` names a column; simple names are named automatically.
- `[condition]` (where clause): a filtering expression; only rows for which it evaluates to a true value produce a result row.
- `[order]` (order by): an ordering expression evaluated for each unfiltered row; its values are the sorting criterion.
- `[tables]` (with clause): comma-separated descriptions `[table name]([column names]) as ([select statement])`; any select statement may describe such a table, and a table **may have a single recursive case** defining output rows in terms of other output rows.
- All SQL statements end in a semicolon; output is one line per row, columns separated by `|`.

### Join rule
Joins are expressed by separating table names with commas in the `from` clause. If the left table has *m* rows and the right has *n*, the joined table has **m·n rows** — one for every combination. A `where` clause expressing the cross-table relationship recovers the meaningful rows. Numeric equality is tested with a single `=`. Aliases (`from temps as a, temps as b`) plus dot expressions (`a.city`) disambiguate overlapping column names and enable self-joins.

### Interpreter execution order
The text's interpreter always computes: **join → filter → sort → map(make_row)** — but this is only one **query plan**. Declarative languages describe the form of a result, not how to compute it; query interpreters are free to choose among procedures. E.g. instead of joining `cities` and `distances` then filtering, sort both tables by `name` and merge equal-name rows in one linear pass. When tables are large, efficiency gains from query plan selection can be substantial; choosing efficient procedures is a core feature of database systems.

### Aggregation rules
- Aggregate functions `max`, `min`, `count`, `sum` return the maximum, minimum, number, and sum of a column's values; multiple aggregates can apply to the same rows via multiple columns. Only rows passing the `where` clause are considered.
- `distinct` inside an aggregate excludes repeated values; `count(*)` counts rows.
- `group by` partitions rows into groups — one per unique combination of the grouping expressions' values; aggregates in the column description or `having` clause apply **per group**.
- `having` filters groups and may call aggregate functions. For fastest execution and clearest use: row-content conditions go in `where`; use `having` only when the condition requires aggregation (e.g. a minimum `count` per group).

## Code Examples

### Tables, select, where, order
```sql
sqlite> select 38 as latitude, 122 as longitude, "Berkeley" as name;
38|122|Berkeley

sqlite> create table cities as
   ...>    select 38 as latitude, 122 as longitude, "Berkeley" as name union
   ...>    select 42,             71,               "Cambridge"        union
   ...>    select 45,             93,               "Minneapolis";
sqlite> select * from cities;
38|122|Berkeley
42|71|Cambridge
45|93|Minneapolis

sqlite> select name, 60*abs(latitude-38) from cities;
Berkeley|0
Cambridge|240
Minneapolis|420

sqlite> create table distances as
   ...>   select name, 60*abs(latitude-38) as distance from cities;
sqlite> select distance/5, name from distances;        -- 0|Berkeley 48|Cambridge 84|Minneapolis
sqlite> create table cold as select name from cities where latitude > 43;
sqlite> select name, "is cold!" from cold;             -- Minneapolis|is cold!
sqlite> select distance, name from distances order by -distance;
84|Minneapolis
48|Cambridge
0|Berkeley
```
`union` stacks rows (left table's column names win; spacing within a line is irrelevant).

### Joins and self-joins
```sql
sqlite> create table temps as
   ...>   select "Berkeley" as city, 68 as temp union
   ...>   select "Chicago"         , 59         union
   ...>   select "Minneapolis"     , 55;

sqlite> select * from cities, temps;      -- 3·3 = 9 rows, every combination

sqlite> select name, latitude, temp from cities, temps where name = city;
Berkeley|38|68
Minneapolis|45|55

sqlite> select a.city, b.city, a.temp - b.temp
   ...>        from temps as a, temps as b where a.city < b.city;
Berkeley|Chicago|10
Berkeley|Minneapolis|15
Chicago|Minneapolis|5
```
The alphabetical constraint `a.city < b.city` ensures each pair appears only once.

### String building with `||`
```sql
sqlite> create table nouns as
  ....>   select "the dog" as phrase union
  ....>   select "the cat"           union
  ....>   select "the bird";
sqlite> select subject.phrase || " chased " || object.phrase
  ....>        from nouns as subject, nouns as object
  ....>        where subject.phrase != object.phrase;
the bird chased the cat
...  -- all 6 ordered pairs
```
(Exercise in the text: use a recursive local table to generate "the dog that chased the cat that chased the bird also chased the bird.")

### Aggregation and grouping
```sql
sqlite> create table animals as
  ....>   select "dog" as name, 4 as legs, 20 as weight union
  ....>   select "cat"        , 4        , 10           union
  ....>   select "ferret"     , 4        , 10           union
  ....>   select "t-rex"      , 2        , 12000        union
  ....>   select "penguin"    , 2        , 10           union
  ....>   select "bird"       , 2        , 6;

sqlite> select max(legs) from animals;                       -- 4
sqlite> select sum(weight) from animals;                     -- 12056
sqlite> select min(legs), max(weight) from animals where name <> "t-rex";  -- 2|20
sqlite> select count(legs) from animals;                     -- 6
sqlite> select count(distinct legs) from animals;            -- 2
sqlite> select legs, max(weight) from animals group by legs;
2|12000
4|20
sqlite> select weight from animals group by weight having count(*)>1;   -- 10
sqlite> select max(name), legs, weight from animals group by legs, weight
  ....>   having max(weight) < 100;
bird|2|6
penguin|2|10
ferret|4|10
dog|4|20
sqlite> select count(*), weight/legs from animals group by weight/legs;
2|2
1|3
2|5
1|6000
```
Non-aggregate columns under aggregation: `select name, max(weight) from animals;` gives `t-rex|12000` — SQL picks the row corresponding to the aggregate when it is clear (`max`/`min`). With `count` the corresponding row is unclear and the chosen value may be arbitrary.

## Worked Examples

### 1. The SQL interpreter in Python
Each table's rows are instances of a `namedtuple` class; a table is a list of rows:
```python
>>> from collections import namedtuple
>>> CitiesRow = namedtuple("Row", ["latitude", "longitude", "name"])
>>> cities = [CitiesRow(38, 122, "Berkeley"),
              CitiesRow(42,  71, "Cambridge"),
              CitiesRow(43,  93, "Minneapolis")]
```
A select is just `map` over rows (`DistancesRow(name, 60*abs(latitude-38))`). The general interpreter:
```python
>>> class Select:
        """select [columns] from [tables] where [condition] order by [order]."""
        def __init__(self, columns, tables, condition, order):
            self.columns = columns
            self.tables = tables
            self.condition = condition
            self.order = order
            self.make_row = create_make_row(self.columns)
        def execute(self, env):
            """Join, filter, sort, and map rows from tables to columns."""
            from_rows = join(self.tables, env)
            filtered_rows = filter(self.filter, from_rows)
            ordered_rows = self.sort(filtered_rows)
            return map(self.make_row, ordered_rows)
        def filter(self, row):
            if self.condition:
                return eval(self.condition, row)
            else:
                return True
        def sort(self, rows):
            if self.order:
                return sorted(rows, key=lambda r: eval(self.order, r))
            else:
                return rows

>>> def create_make_row(description):
        """Return a function from an input environment (dict) to an output row."""
        columns = description.split(", ")
        expressions, names = [], []
        for column in columns:
            if " as " in column:
                expression, name = column.split(" as ")
            else:
                expression, name = column, column
            expressions.append(expression)
            names.append(name)
        row = namedtuple("Row", names)
        return lambda env: row(*[eval(e, env) for e in expressions])

>>> from itertools import product
>>> def join(tables, env):
        """Return an iterator over dictionaries from names to values in a row."""
        names = tables.split(", ")
        joined_rows = product(*[env[name] for name in names])
        return map(lambda rows: make_env(rows, names), joined_rows)

>>> def make_env(rows, names):
        """Create an environment of names bound to values."""
        env = dict(zip(names, rows))
        for row in rows:
            for name in row._fields:
                env[name] = getattr(row, name)
        return env
```
`row._fields` gives the column names because each row is a namedtuple. Running it:
```python
>>> env = {"cities": cities}
>>> select = Select("name, 60*abs(latitude-38) as distance",
                    "cities", "name != 'Berkeley'", "-longitude")
>>> for row in select.execute(env): print(row)
Row(name='Minneapolis', distance=300)
Row(name='Cambridge', distance=240)
>>> env["distances"] = list(select.execute(env))
>>> joined = Select("cities.name as name, distance, longitude", "cities, distances",
                    "cities.name == distances.name", None)
>>> for row in joined.execute(env): print(row)
Row(name='Cambridge', distance=240, longitude=71)
Row(name='Minneapolis', distance=300, longitude=93)
```
Each run reproduces the equivalent sqlite statement's output — demonstrating the relationship between sequence processing operations and query languages. The full `sql` example also parses statement text and executes `create table` and `union`.

### 2. With-clause join: cities and states
```sql
sqlite> with
   ...>   states(city, state) as (
   ...>     select "Berkeley",  "California"    union
   ...>     select "Boston",    "Massachusetts" union
   ...>     select "Cambridge", "Massachusetts" union
   ...>     select "Chicago",   "Illinois"      union
   ...>     select "Pasadena",  "California"
   ...>   )
   ...> select a.city, b.city, a.state from states as a, states as b
   ...>        where a.state = b.state and a.city < b.city;
Berkeley|Pasadena|California
Boston|Cambridge|Massachusetts
```

### 3. Recursive with-clauses: ranges, Pythagorean triples, Fibonacci
Integers 5–15, odd ones squared:
```sql
sqlite> with
   ...>   ints(n) as (
   ...>     select 5 union
   ...>     select n+1 from ints where n < 15
   ...>   )
   ...> select n, n*n from ints where n % 2 = 1;
5|25
7|49
9|81
11|121
13|169
15|225
```
Multiple tables in one with clause — all Pythagorean triples (a² + b² = c²) up to 20:
```sql
sqlite> with
   ...>   ints(n) as (
   ...>     select 1 union select n+1 from ints where n < 20
   ...>   ),
   ...>   squares(x, xx) as (
   ...>     select n, n*n from ints
   ...>   ),
   ...>   sum_of_squares(a, b, sum) as (
   ...>     select a.x, b.x, a.xx + b.xx
   ...>            from squares as a, squares as b where a.x < b.x
   ...>   )
   ...> select a, b, x from squares, sum_of_squares where sum = xx;
3|4|5
6|8|10
5|12|13
9|12|15
8|15|17
12|16|20
```
Fibonacci — the key design insight: **each input row must carry all information needed to compute a result row**; computing the next Fibonacci number needs not only the current but also the previous element:
```sql
sqlite> with
   ...>   fib(previous, current) as (
   ...>     select 0, 1 union
   ...>     select current, previous+current from fib
   ...>     where current <= 100
   ...>   )
   ...> select previous from fib;
0
1
1
2
3
5
8
13
21
34
55
89
```
"These examples demonstrate that recursion is a powerful means of combination, even in declarative languages."

## Key Concepts
- **Record / table (relation)**: a table has a fixed number of named, typed columns; each row is a record with one value per column.
- **Query interpreter**: the database component that designs and performs the process producing a declared result.
- **Projection**: evaluating the column-expression list for each input row.
- **Union**: stacks rows of two tables; left table's column names win.
- **Join**: comma-separated tables in `from`; m·n combinations pruned by `where`.
- **Alias + dot expression**: table disambiguation; required for self-joins.
- **Query plan**: the concrete procedure chosen among many equivalent ones.
- **`with` clause**: generates and names local tables, optionally with one recursive case.
- **`||`**: string concatenation.
- **`group by` / `having`**: group formation per unique value combination; group-level filtering, may aggregate.
- **`distinct` / `count(*)`**: dedup inside aggregates; row count.

## Mental Models
- A select statement is `map(make_row, sort(filter(product(tables))))` — the text's interpreter literally is this.
- A join is a full Cartesian product that the `where` clause prunes.
- Declarative = you write the spec; the query plan is the interpreter's business. That separation is what enables optimization.
- Designing a recursive query = making each row self-sufficient for producing the next row (Fibonacci carries `(previous, current)`).

## Anti-patterns / Common Errors
- **Joining without a relating `where` clause**: m·n meaningless combinations.
- **Duplicate pairs in self-joins**: without an ordering constraint like `a.city < b.city`, each pair appears twice (and each row pairs with itself).
- **Aggregation conditions in `where`**: aggregates belong in `having`; `where` sees individual rows only. Conversely, plain row filters in `having` cost speed and clarity.
- **Non-aggregate columns under `group by` with unclear correspondence**: with `count` the chosen value may be arbitrary. Include non-aggregate columns only when predictable from the aggregation (e.g. alongside `max`/`min`), and include the grouping expression itself in the columns so each result row is identifiable.
- **Recursive table rows missing state**: if a row lacks the data needed to compute its successor, the recursion cannot proceed — widen the row (Fibonacci pattern).
- **Forgetting the termination condition in the recursive case** (`where n < 15`, `where current <= 100`): the recursion must stop.

## Key Takeaways
1. SQL describes results; the interpreter picks the procedure — the separation enables query-plan optimization.
2. Every select is join → filter → sort → project over row sequences; the text proves it by building the interpreter in ~60 lines of Python with `namedtuple`, `itertools.product`, `filter`, `sorted`, `map`, `eval`.
3. Use aliases + dot expressions for self-joins; add an ordering predicate to dedup pairs.
4. `with` + one recursive case gives recursion inside a declarative language (ranges, triples, Fibonacci, sentence generation).
5. Carry enough state per row to compute the next row in recursive tables.
6. `where` filters rows before aggregation; `having` filters groups after; `distinct` dedups inside aggregates; `count(*)` counts rows.

## Connects To
- **Section 4.2**: the interpreter is built from lazy `map`/`filter`/`product` iterators.
- **Chapter 3 (interpreters)**: a query interpreter is another instance of the evaluator pattern — representation, parser, evaluator.
- **Section 4.4**: logic programming is a more expressive declarative paradigm over relations.
- **Section 4.7**: MapReduce is another what/how split — the programmer supplies map/reduce, the framework supplies the plan.
