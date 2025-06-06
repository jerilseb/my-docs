# Supabase

### Install via CDN#

You can install @supabase/supabase-js via CDN links.

```
<script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2"></script>
//or
<script src="https://unpkg.com/@supabase/supabase-js@2"></script>
```

---

## Initializing

Create a new client for use in the browser.

You can initialize a new Supabase client using the`createClient()`method.

The Supabase client is your entrypoint to the rest of the Supabase functionality and is the easiest way to interact with everything we offer within the Supabase ecosystem.

### Parameters

- supabaseUrlRequiredstring

The unique Supabase URL which is supplied when you create a new project in your project dashboard.

- supabaseKeyRequiredstring

The unique Supabase Key which is supplied when you create a new project in your project dashboard.

- optionsOptionalSupabaseClientOptions

```
import { createClient } from '@supabase/supabase-js'
// Create a single supabase client for interacting with your database
const supabase = createClient('https://xyzcompany.supabase.co', 'public-anon-key')
```

---

## TypeScript support

`supabase-js`has TypeScript support for type inference, autocompletion, type-safe queries, and more.

With TypeScript,`supabase-js`detects things like`not null`constraints and generated columns . Nullable columns are typed as`T | null`when you select the column. Generated columns will show a type error when you insert to it.

`supabase-js`also detects relationships between tables. A referenced table with one-to-many relationship is typed as`T[]`. Likewise, a referenced table with many-to-one relationship is typed as`T | null`.

## Generating TypeScript Types#

You can use the Supabase CLI to generate the types . You can also generate the types from the dashboard .

```
supabase gen types typescript --project-id abcdefghijklmnopqrst > database.types.ts
```

These types are generated from your database schema. Given a table`public.movies`, the generated types will look like:

```
create table public.movies (
  id bigint generated always as identity primary key,
  name text not null,
  data jsonb null
);
```

```
export type Json = string | number | boolean | null | { [key: string]: Json | undefined } | Json[]
export interface Database {
  public: {
    Tables: {
      movies: {
        Row: {               // the data expected from .select()
          id: number
          name: string
          data: Json | null
        }
        Insert: {            // the data to be passed to .insert()
          id?: never         // generated columns must not be supplied
          name: string       // `not null` columns with no default must be supplied
          data?: Json | null // nullable columns can be omitted
        }
        Update: {            // the data to be passed to .update()
          id?: never
          name?: string      // `not null` columns are optional on .update()
          data?: Json | null
        }
      }
    }
  }
}
```

## Using TypeScript type definitions#

You can supply the type definitions to`supabase-js`like so:

```
import { createClient } from '@supabase/supabase-js'
import { Database } from './database.types'
const supabase = createClient<Database>(
  process.env.SUPABASE_URL,
  process.env.SUPABASE_ANON_KEY
)
```

## Helper types for Tables and Joins#

You can use the following helper types to make the generated TypeScript types easier to use.

Sometimes the generated types are not what you expect. For example, a view's column may show up as nullable when you expect it to be`not null`. Using type-fest , you can override the types like so:

```
export type Json = // ...
export interface Database {
  // ...
}
```

```
import { MergeDeep } from 'type-fest'
import { Database as DatabaseGenerated } from './database-generated.types'
export { Json } from './database-generated.types'
// Override the type for a specific column in a view:
export type Database = MergeDeep<
  DatabaseGenerated,
  {
    public: {
      Views: {
        movies_view: {
          Row: {
            // id is a primary key in public.movies, so it must be `not null`
            id: number
          }
        }
      }
    }
  }
>
```

You can also override the type of an individual successful response if needed:

```
// Partial type override allows you to only override some of the properties in your results
const { data } = await supabase.from('countries').select().overrideTypes<Array<{ id: string }>>()
// For a full replacement of the original return type use the `{ merge: false }` property as second argument
const { data } = await supabase
  .from('countries')
  .select()
  .overrideTypes<Array<{ id: string }>, { merge: false }>()
// Use it with `maybeSingle` or `single`
const { data } = await supabase.from('countries').select().single().overrideTypes<{ id: string }>()
```

The generated types provide shorthands for accessing tables and enums.

```
import { Database, Tables, Enums } from "./database.types.ts";
// Before 😕
let movie: Database['public']['Tables']['movies']['Row'] = // ...
// After 😍
let movie: Tables<'movies'>
```

### Response types for complex queries#

`supabase-js`always returns a`data`object (for success), and an`error`object (for unsuccessful requests).

These helper types provide the result types from any query, including nested types for database joins.

Given the following schema with a relation between cities and countries, we can get the nested`CountriesWithCities`type:

```
create table countries (
  "id" serial primary key,
  "name" text
);
create table cities (
  "id" serial primary key,
  "name" text,
  "country_id" int references "countries"
);
```

```
import { QueryResult, QueryData, QueryError } from '@supabase/supabase-js'
const countriesWithCitiesQuery = supabase
  .from("countries")
  .select(`
    id,
    name,
    cities (
      id,
      name
    )
  `);
type CountriesWithCities = QueryData<typeof countriesWithCitiesQuery>;
const { data, error } = await countriesWithCitiesQuery;
if (error) throw error;
const countriesWithCities: CountriesWithCities = data;
```

---

## Fetch data

Perform a SELECT query on the table or view.

- By default, Supabase projects return a maximum of 1,000 rows. This setting can be changed in your project's API settings . It's recommended that you keep it low to limit the payload size of accidental or malicious requests. You can use`range()`queries to paginate through your data.
- `select()`can be combined with Filters 
- `select()`can be combined with Modifiers 
- `apikey`is a reserved keyword if you're using the Supabase Platform and should be avoided as a column name .

### Parameters

- columnsOptionalQuery

The columns to retrieve, separated by commas. Columns can be renamed when returned with`customName:columnName`

- optionsRequiredobject

Named parameters

```
const { data, error } = await supabase
  .from('characters')
  .select()
```

---

## Insert data

Perform an INSERT into the table or view.

### Parameters

- valuesRequiredOne of the following options

The values to insert. Pass an object to insert a single row or an array to insert multiple rows.

- Option 1Row

- Option 2Array<Row>

- optionsOptionalobject

Named parameters

```
const { error } = await supabase
  .from('countries')
  .insert({ id: 1, name: 'Mordor' })
```

---

## Update data

Perform an UPDATE on the table or view.

- `update()`should always be combined with Filters to target the item(s) you wish to update.

### Parameters

- valuesRequiredRow

The values to update with

- optionsRequiredobject

Named parameters

```
const { error } = await supabase
  .from('instruments')
  .update({ name: 'piano' })
  .eq('id', 1)
```

---

## Upsert data

Perform an UPSERT on the table or view. Depending on the column(s) passed to`onConflict`,`.upsert()`allows you to perform the equivalent of`.insert()`if a row with the corresponding`onConflict`columns doesn't exist, or if it does exist, perform an alternative action depending on`ignoreDuplicates`.

- Primary keys must be included in`values`to use upsert.

### Parameters

- valuesRequiredOne of the following options

The values to upsert with. Pass an object to upsert a single row or an array to upsert multiple rows.

- Option 1Row

- Option 2Array<Row>

- optionsOptionalobject

Named parameters

```
const { data, error } = await supabase
  .from('instruments')
  .upsert({ id: 1, name: 'piano' })
  .select()
```

---

## Delete data

Perform a DELETE on the table or view.

- `delete()`should always be combined with filters to target the item(s) you wish to delete.
- If you use`delete()`with filters and you have RLS enabled, only rows visible through`SELECT`policies are deleted. Note that by default no rows are visible, so you need at least one`SELECT`/`ALL`policy that makes the rows visible.
- When using`delete().in()`, specify an array of values to target multiple rows with a single query. This is particularly useful for batch deleting entries that share common criteria, such as deleting users by their IDs. Ensure that the array you provide accurately represents all records you intend to delete to avoid unintended data removal.

### Parameters

- optionsRequiredobject

Named parameters

```
const response = await supabase
  .from('countries')
  .delete()
  .eq('id', 1)
```

---

## Call a Postgres function

Perform a function call.

You can call Postgres functions as*Remote Procedure Calls*, logic in your database that you can execute from anywhere. Functions are useful when the logic rarely changes—like for password resets and updates.

```
create or replace function hello_world() returns text as $$
  select 'Hello world';
$$ language sql;
```

To call Postgres functions on Read Replicas , use the`get: true`option.

### Parameters

- fnRequiredFnName

The function name to call

- argsRequiredFn['Args']

The arguments to pass to the function call

- optionsRequiredobject

Named parameters

```
const { data, error } = await supabase.rpc('hello_world')
```

---

## Using filters

Filters allow you to only return rows that match certain conditions.

Filters can be used on`select()`,`update()`,`upsert()`, and`delete()`queries.

If a Postgres function returns a table response, you can also apply filters.

```
const { data, error } = await supabase
  .from('instruments')
  .select('name, section_id')
  .eq('name', 'violin')    // Correct
const { data, error } = await supabase
  .from('instruments')
  .eq('name', 'violin')    // Incorrect
  .select('name, section_id')
```

---

## Column is equal to a value

Match only rows where`column`is equal to`value`.

### Parameters

- columnRequiredColumnName

The column to filter on

- valueRequired

The value to filter with

```
const { data, error } = await supabase
  .from('characters')
  .select()
  .eq('name', 'Leia')
```

---

## Column is not equal to a value

Match only rows where`column`is not equal to`value`.

### Parameters

- columnRequiredColumnName

The column to filter on

- valueRequired

The value to filter with

```
const { data, error } = await supabase
  .from('characters')
  .select()
  .neq('name', 'Leia')
```

---

## Column is greater than a value

Match only rows where`column`is greater than`value`.

### Parameters

- columnRequiredOne of the following options

The column to filter on

- Option 1ColumnName

- Option 2string

- valueRequiredOne of the following options

The value to filter with

- Option 1Row['ColumnName']

- Option 2unknown

```
const { data, error } = await supabase
  .from('characters')
  .select()
  .gt('id', 2)
```

---

## Column is greater than or equal to a value

Match only rows where`column`is greater than or equal to`value`.

### Parameters

- columnRequiredOne of the following options

The column to filter on

- Option 1ColumnName

- Option 2string

- valueRequiredOne of the following options

The value to filter with

- Option 1Row['ColumnName']

- Option 2unknown

```
const { data, error } = await supabase
  .from('characters')
  .select()
  .gte('id', 2)
```

---

## Column is less than a value

Match only rows where`column`is less than`value`.

### Parameters

- columnRequiredOne of the following options

The column to filter on

- Option 1ColumnName

- Option 2string

- valueRequiredOne of the following options

The value to filter with

- Option 1Row['ColumnName']

- Option 2unknown

```
const { data, error } = await supabase
  .from('characters')
  .select()
  .lt('id', 2)
```

---

## Column is less than or equal to a value

Match only rows where`column`is less than or equal to`value`.

### Parameters

- columnRequiredOne of the following options

The column to filter on

- Option 1ColumnName

- Option 2string

- valueRequiredOne of the following options

The value to filter with

- Option 1Row['ColumnName']

- Option 2unknown

```
const { data, error } = await supabase
  .from('characters')
  .select()
  .lte('id', 2)
```

---

## Column matches a pattern

Match only rows where`column`matches`pattern`case-sensitively.

### Parameters

- columnRequiredOne of the following options

The column to filter on

- Option 1ColumnName

- Option 2string

- patternRequiredstring

The pattern to match with

```
const { data, error } = await supabase
  .from('characters')
  .select()
  .like('name', '%Lu%')
```

---

## Column matches a case-insensitive pattern

Match only rows where`column`matches`pattern`case-insensitively.

### Parameters

- columnRequiredOne of the following options

The column to filter on

- Option 1ColumnName

- Option 2string

- patternRequiredstring

The pattern to match with

```
const { data, error } = await supabase
  .from('characters')
  .select()
  .ilike('name', '%lu%')
```

---

## Column is a value

Match only rows where`column`IS`value`.

### Parameters

- columnRequiredOne of the following options

The column to filter on

- Option 1ColumnName

- Option 2string

- valueRequiredOne of the following options

The value to filter with

- Option 1null

- Option 2boolean

```
const { data, error } = await supabase
  .from('countries')
  .select()
  .is('name', null)
```

---

## Column is in an array

Match only rows where`column`is included in the`values`array.

### Parameters

- columnRequiredColumnName

The column to filter on

- valuesRequiredArray

The values array to filter with

```
const { data, error } = await supabase
  .from('characters')
  .select()
  .in('name', ['Leia', 'Han'])
```

---

## Column contains every element in a value

Only relevant for jsonb, array, and range columns. Match only rows where`column`contains every element appearing in`value`.

### Parameters

- columnRequiredOne of the following options

The jsonb, array, or range column to filter on

- Option 1ColumnName

- Option 2string

- valueRequiredOne of the following options

The jsonb, array, or range value to filter with

- Option 1string

- Option 2Record<string, unknown>

- Option 3Array<Row['ColumnName']>

- Option 4Array<unknown>

```
const { data, error } = await supabase
  .from('issues')
  .select()
  .contains('tags', ['is:open', 'priority:low'])
```

---

## Contained by value

Only relevant for jsonb, array, and range columns. Match only rows where every element appearing in`column`is contained by`value`.

### Parameters

- columnRequiredOne of the following options

The jsonb, array, or range column to filter on

- Option 1ColumnName

- Option 2string

- valueRequiredOne of the following options

The jsonb, array, or range value to filter with

- Option 1string

- Option 2Record<string, unknown>

- Option 3Array<Row['ColumnName']>

- Option 4Array<unknown>

```
const { data, error } = await supabase
  .from('classes')
  .select('name')
  .containedBy('days', ['monday', 'tuesday', 'wednesday', 'friday'])
```

---

## Greater than a range

Only relevant for range columns. Match only rows where every element in`column`is greater than any element in`range`.

### Parameters

- columnRequiredOne of the following options

The range column to filter on

- Option 1ColumnName

- Option 2string

- rangeRequiredstring

The range to filter with

```
const { data, error } = await supabase
  .from('reservations')
  .select()
  .rangeGt('during', '[2000-01-02 08:00, 2000-01-02 09:00)')
```

---

## Greater than or equal to a range

Only relevant for range columns. Match only rows where every element in`column`is either contained in`range`or greater than any element in`range`.

### Parameters

- columnRequiredOne of the following options

The range column to filter on

- Option 1ColumnName

- Option 2string

- rangeRequiredstring

The range to filter with

```
const { data, error } = await supabase
  .from('reservations')
  .select()
  .rangeGte('during', '[2000-01-02 08:30, 2000-01-02 09:30)')
```

---

## Less than a range

Only relevant for range columns. Match only rows where every element in`column`is less than any element in`range`.

### Parameters

- columnRequiredOne of the following options

The range column to filter on

- Option 1ColumnName

- Option 2string

- rangeRequiredstring

The range to filter with

```
const { data, error } = await supabase
  .from('reservations')
  .select()
  .rangeLt('during', '[2000-01-01 15:00, 2000-01-01 16:00)')
```

---

## Less than or equal to a range

Only relevant for range columns. Match only rows where every element in`column`is either contained in`range`or less than any element in`range`.

### Parameters

- columnRequiredOne of the following options

The range column to filter on

- Option 1ColumnName

- Option 2string

- rangeRequiredstring

The range to filter with

```
const { data, error } = await supabase
  .from('reservations')
  .select()
  .rangeLte('during', '[2000-01-01 14:00, 2000-01-01 16:00)')
```

---

## Mutually exclusive to a range

Only relevant for range columns. Match only rows where`column`is mutually exclusive to`range`and there can be no element between the two ranges.

### Parameters

- columnRequiredOne of the following options

The range column to filter on

- Option 1ColumnName

- Option 2string

- rangeRequiredstring

The range to filter with

```
const { data, error } = await supabase
  .from('reservations')
  .select()
  .rangeAdjacent('during', '[2000-01-01 12:00, 2000-01-01 13:00)')
```

---

## With a common element

Only relevant for array and range columns. Match only rows where`column`and`value`have an element in common.

### Parameters

- columnRequiredOne of the following options

The array or range column to filter on

- Option 1ColumnName

- Option 2string

- valueRequiredOne of the following options

The array or range value to filter with

- Option 1string

- Option 2Array<Row['ColumnName']>

- Option 3Array<unknown>

```
const { data, error } = await supabase
  .from('issues')
  .select('title')
  .overlaps('tags', ['is:closed', 'severity:high'])
```

---

## Match a string

Only relevant for text and tsvector columns. Match only rows where`column`matches the query string in`query`.

- For more information, see Postgres full text search .

### Parameters

- columnRequiredOne of the following options

The text or tsvector column to filter on

- Option 1ColumnName

- Option 2string

- queryRequiredstring

The query text to match with

- optionsOptionalobject

Named parameters

```
const result = await supabase
  .from("texts")
  .select("content")
  .textSearch("content", `'eggs' & 'ham'`, {
    config: "english",
  });
```

---

## Match an associated value

Match only rows where each column in`query`keys is equal to its associated value. Shorthand for multiple`.eq()`s.

### Parameters

- queryRequiredOne of the following options

The object to filter with, with column names as keys mapped to their filter values

- Option 1Record<ColumnName, Row['ColumnName']>

- Option 2Record<string, unknown>

```
const { data, error } = await supabase
  .from('characters')
  .select('name')
  .match({ id: 2, name: 'Leia' })
```

---

## Don't match the filter

Match only rows which doesn't satisfy the filter.

not() expects you to use the raw PostgREST syntax for the filter values.

```
.not('id', 'in', '(5,6,7)')  // Use `()` for `in` filter
.not('arraycol', 'cs', '{"a","b"}')  // Use `cs` for `contains()`, `{}` for array values
```

### Parameters

- columnRequiredOne of the following options

The column to filter on

- Option 1ColumnName

- Option 2string

- operatorRequiredOne of the following options

The operator to be negated to filter with, following PostgREST syntax

- Option 1FilterOperator

- Option 2string

- valueRequiredOne of the following options

The value to filter with, following PostgREST syntax

- Option 1Row['ColumnName']

- Option 2unknown

```
const { data, error } = await supabase
  .from('countries')
  .select()
  .not('name', 'is', null)
```

---

## Match at least one filter

Match only rows which satisfy at least one of the filters.

or() expects you to use the raw PostgREST syntax for the filter names and values.

```
.or('id.in.(5,6,7), arraycol.cs.{"a","b"}')  // Use `()` for `in` filter, `{}` for array values and `cs` for `contains()`.
.or('id.in.(5,6,7), arraycol.cd.{"a","b"}')  // Use `cd` for `containedBy()`
```

### Parameters

- filtersRequiredstring

The filters to use, following PostgREST syntax

- optionsRequiredobject

Named parameters

```
const { data, error } = await supabase
  .from('characters')
  .select('name')
  .or('id.eq.2,name.eq.Han')
```

---

## Match the filter

Match only rows which satisfy the filter. This is an escape hatch - you should use the specific filter methods wherever possible.

filter() expects you to use the raw PostgREST syntax for the filter values.

```
.filter('id', 'in', '(5,6,7)')  // Use `()` for `in` filter
.filter('arraycol', 'cs', '{"a","b"}')  // Use `cs` for `contains()`, `{}` for array values
```

### Parameters

- columnRequiredOne of the following options

The column to filter on

- Option 1ColumnName

- Option 2string

- operatorRequiredOne of the following options

The operator to filter with, following PostgREST syntax

- Option 1FilterOperator

- Option 2"not.eq"

- Option 3"not.neq"

- Option 4"not.gt"

- Option 5"not.gte"

- Option 6"not.lt"

- Option 7"not.lte"

- Option 8"not.like"

- Option 9"not.ilike"

- Option 10"not.is"

- Option 11"not.in"

- Option 12"not.cs"

- Option 13"not.cd"

- Option 14"not.sl"

- Option 15"not.sr"

- Option 16"not.nxl"

- Option 17"not.nxr"

- Option 18"not.adj"

- Option 19"not.ov"

- Option 20"not.fts"

- Option 21"not.plfts"

- Option 22"not.phfts"

- Option 23"not.wfts"

- Option 24string

- valueRequiredunknown

The value to filter with, following PostgREST syntax

```
const { data, error } = await supabase
  .from('characters')
  .select()
  .filter('name', 'in', '("Han","Yoda")')
```

---

## Using modifiers

Filters work on the row level—they allow you to return rows that only match certain conditions without changing the shape of the rows. Modifiers are everything that don't fit that definition—allowing you to change the format of the response (e.g., returning a CSV string).

Modifiers must be specified after filters. Some modifiers only apply for queries that return rows (e.g.,`select()`or`rpc()`on a function that returns a table response).

---

## Return data after inserting

Perform a SELECT on the query result.

### Parameters

- columnsOptionalQuery

The columns to retrieve, separated by commas

```
const { data, error } = await supabase
  .from('characters')
  .upsert({ id: 1, name: 'Han Solo' })
  .select()
```

---

## Order the results

Order the query result by`column`.

### Parameters

- columnRequiredOne of the following options

The column to order by

- Option 1ColumnName

- Option 2string

- optionsOptionalobject

Named parameters

```
const { data, error } = await supabase
  .from('characters')
  .select('id, name')
  .order('id', { ascending: false })
```

---

## Limit the number of rows returned

Limit the query result by`count`.

### Parameters

- countRequirednumber

The maximum number of rows to return

- optionsRequiredobject

Named parameters

```
const { data, error } = await supabase
  .from('characters')
  .select('name')
  .limit(1)
```

---

## Limit the query to a range

Limit the query result by starting at an offset`from`and ending at the offset`to`. Only records within this range are returned. This respects the query order and if there is no order clause the range could behave unexpectedly. The`from`and`to`values are 0-based and inclusive:`range(1, 3)`will include the second, third and fourth rows of the query.

### Parameters

- fromRequirednumber

The starting index from which to limit the result

- toRequirednumber

The last index to which to limit the result

- optionsRequiredobject

Named parameters

```
const { data, error } = await supabase
  .from('countries')
  .select('name')
  .range(0, 1)
```

---

## Set an abort signal

Set the AbortSignal for the fetch request.

You can use this to set a timeout for the request.

### Parameters

- signalRequiredAbortSignal

The AbortSignal to use for the fetch request

```
const ac = new AbortController()
ac.abort()
const { data, error } = await supabase
  .from('very_big_table')
  .select()
  .abortSignal(ac.signal)
```

---

## Retrieve one row of data

Return`data`as a single object instead of an array of objects.

```
const { data, error } = await supabase
  .from('characters')
  .select('name')
  .limit(1)
  .single()
```

---

## Retrieve zero or one row of data

Return`data`as a single object instead of an array of objects.

### Return Type

One of the following options
- Option 1null

- Option 2ResultOne

```
const { data, error } = await supabase
  .from('characters')
  .select()
  .eq('name', 'Katniss')
  .maybeSingle()
```

---

## Retrieve as a CSV

Return`data`as a string in CSV format.

### Return Type

string

```
const { data, error } = await supabase
  .from('characters')
  .select()
  .csv()
```

---

## Override type of successful response

Override the type of the returned`data`.

- Deprecated: use overrideTypes method instead

```
const { data } = await supabase
  .from('countries')
  .select()
  .returns<Array<MyType>>()
```

---

## Partially override or replace type of successful response

Override the type of the returned`data`field in the response.

```
const { data } = await supabase
  .from('countries')
  .select()
  .overrideTypes<Array<MyType>, { merge: false }>()
```

---

## Using explain

Return`data`as the EXPLAIN plan for the query.

For debugging slow queries, you can get the Postgres`EXPLAIN`execution plan of a query using the`explain()`method. This works on any query, even for`rpc()`or writes.

Explain is not enabled by default as it can reveal sensitive information about your database. It's best to only enable this for testing environments but if you wish to enable it for production you can provide additional protection by using a`pre-request`function.

Follow the Performance Debugging Guide to enable the functionality on your project.

### Parameters

- optionsRequiredobject

Named parameters

### Return Type

One of the following options
- Option 1string

- Option 2Array<Record<string, unknown>>

```
const { data, error } = await supabase
  .from('characters')
  .select()
  .explain()
```