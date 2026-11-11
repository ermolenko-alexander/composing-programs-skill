# Section 4.3: Declarative Programming

## Core Idea
A declarative language (SQL) describes the desired *form of a result*, not the computation; the database's query interpreter chooses how to compute it. Under the hood, select statements are sequence operations: join (product), filter, sort, map.

## Frameworks Introduced
- **Declarative vs procedural programming**: in Python/Scheme the programmer describes the computational process; in SQL the statement describes only the result's form and the query interpreter designs the process.
  - When to use: retrieving/transforming records from large repositories (databases).
- **SQL select statements**: `select [columns] from [tables] where [condition] order by [order]` — project each input row through column expressions; `where` filters rows; `order by` sorts by an expression; `as` names columns; `union` stacks rows; `create table ... as` names a result.
- **Joins**: comma-separated tables in `from` produce every combination of rows (m·n rows); a `where` clause expressing the cross-table relationship (e.g. `name = city`) recovers the meaningful rows; `as` aliases plus dot expressions disambiguate columns and enable self-joins.
- **Interpreting SQL (query interpreter)**: represent rows as `namedtuple`s, a table as a list of rows; a `Select.execute` = join → filter → sort → map(make_row). Query plans: the interpreter is free to compute the same result more efficiently (e.g. sort both tables by the join key and merge linearly instead of filtering a full product).
- **Recursive select statements (`with` clause)**: `with name(cols) as (select ... union select ... from name where ...)` — a local table may have a recursive case defining output rows in terms of other output rows; enables integer ranges, Pythagorean triples, Fibonacci.
- **Aggregation and grouping**: aggregate functions `max`, `min`, `count`, `sum` (plus `distinct`, `count(*)`); `group by` partitions rows into groups per unique value combination; `having` filters groups and may call aggregates.

## Key Concepts
- **Record / table (relation)**: a stored value; records of similar structure grouped into tables with fixed named, typed columns.
- **Query interpreter**: the database component that designs and performs the computation producing a declared result.
- **Projection**: evaluating a comma-separated list of column expressions for each input row.
- **Alias**: `from temps as a, temps as b` — needed for self-joins and overlapping column names.
- **Query plan**: the concrete procedure a database chooses among many that produce the same declared result.
- **`||` operator**: string concatenation in SQL.
- **`group by` / `having`**: group formation and group-level filtering; aggregates apply per group.
- **`with` clause**: generates and names additional (possibly recursive) tables used by the final select.

## Mental Models
- Think of a select statement as map(project, sort(filter(product(tables)))) — the SQL interpreter in the text literally does this.
- Think of a join as a full Cartesian product that the `where` clause prunes.
- Designing a recursive query = ensuring each row carries all the information needed to compute the next row (Fibonacci needs `(previous, current)`, not just `current`).
- Use `where` to filter rows, `having` to filter groups — put row-level conditions in `where` for speed and clarity.

## Anti-patterns
- **Joining without a relating `where` clause**: yields m·n meaningless row combinations.
- **Duplicate pairs in self-joins**: without an ordering constraint like `a.city < b.city`, each pair appears twice.
- **Non-aggregate columns under `group by` whose value is unpredictable**: when the corresponding row is unclear (e.g. with `count`), the chosen value may be arbitrary — include non-aggregate columns only when predictable from the aggregation.
- **Aggregation conditions in `where`**: aggregates belong in `having`; `where` sees individual rows only.

## Code Examples
```sql
create table cities as
   select 38 as latitude, 122 as longitude, "Berkeley" union
   select 42,             71,               "Cambridge" union
   select 45,             93,               "Minneapolis";

select name, latitude, temp from cities, temps where name = city;
-- Berkeley|38|68
-- Minneapolis|45|55

select a.city, b.city, a.temp - b.temp
       from temps as a, temps as b where a.city < b.city;
```
- **What it demonstrates**: table construction via union, a join related by `where`, and a self-join with aliases and a dedup ordering constraint.

```sql
with
  fib(previous, current) as (
    select 0, 1 union
    select current, previous+current from fib
    where current <= 100
  )
select previous from fib;
```
- **What it demonstrates**: a recursive local table — rows defined in terms of prior output rows.

## Worked Example
The interpreter example. Rows are `namedtuple("Row", [...])` instances; `Select(columns, tables, condition, order)` stores clause strings and builds `make_row` via `create_make_row` (splits "expr as name" pairs, returns `lambda env: row(*[eval(e, env) for e in expressions])`). `execute(env)` does `join` (itertools `product` over the named tables, each combination turned into a name→value dict by `make_env` using `row._fields`), then `filter` by `eval(condition, row)`, then `sorted` by `eval(order, r)`, then `map(make_row, ...)`. Running `Select("name, 60*abs(latitude-38) as distance", "cities", "name != 'Berkeley'", "-longitude")` reproduces exactly the sqlite output — showing a declarative query is just composed sequence processing, and that the fixed join→filter→sort→project order is only one possible query plan.

## Key Takeaways
1. SQL describes results; the interpreter picks the procedure — that separation is what makes query-plan optimization possible.
2. Every select is join → filter → order → project over row sequences.
3. Use aliases + dot expressions for self-joins; add an ordering predicate to avoid duplicate pairs.
4. `with` + one recursive case gives recursion inside a declarative language (ranges, triples, Fibonacci).
5. Carry enough state per row to compute the next row in recursive tables.
6. `where` filters rows before aggregation; `having` filters groups after; `distinct` dedups inside aggregates.

## Connects To
- **Section 4.2**: the interpreter is built from lazy `map`/`filter`/`product` iterators.
- **Section 4.4**: logic programming is a more expressive declarative paradigm over relations.
- **Section 4.7**: MapReduce is another declarative-flavored split between what (map/reduce functions) and how (the framework).
