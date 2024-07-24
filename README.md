[![SDF Workspace Compiles](https://github.com/sdf-labs/utils/actions/workflows/compile-workspace.yml/badge.svg)](https://github.com/sdf-labs/utils/actions/workflows/compile-workspace.yml)

# Official SDF Utils Library

This SDF library contains the standard utilities available in the default namespace in any SDF workspace. As such, macros defined in this workspace do not need to be prefixed with the workspace name, they can be referenced directly. 

Here's an example:

```sql
WITH date_spine AS (
  {{ sdf_utils.date_spine(
          datepart = 'day',
          start_date_raw = "2019-01-01",    
          end_date_raw = "2019-12-31"
    )
  }}
)

select * from date_spine;
```

You'll find documentation for each specific macro below. If no Dialect section is specified, this means the implementation is dialect-agnostic and should work across all supported dialects.

*Note: SDF is still < v1, as such certain scenarios may result in unexpected behavior. Please follow the [contributing guidelines](./CONTRIBUTING.md) and create an issue in this repo if you find any bugs or issues.*

For an in-depth guide on how to use Jinja macros in SDF, please see the Jinja section of [our official docs](https://docs.sdf.com/guide/macro-processing/jinja)

## SDF Utils

| Test Name |
| --------- | 
| [`date_spine()`](#date-spine) |
| [`generate_date_values()`](#generate-date-values) |  
| [`generate_integer_values(condition)`](#generate-integer-values) | 
| [`group_by(condition)`](#group-by) |
| [`generate_surrogate_key()`](#generate-surrogate-key) |

#### Date Spine

[Source Code](./macros/date_spine.jinja)

This macro generates a SQL SELECT statement that produces a single column of dates. 
The dates are generated for each day in the range from the 'from' date to the 'to' date, where from is inclusive and to is not inclusive.

  _Parameters:_
    - `datepart`: The date part to increment. This can be one of the following: 'day', 'week', 'month', 'quarter', 'year'. Others like hour, minute, second are dialect-specific and supported in Snowflake for example.
    - `start_date`: The start date of the range, as a SQL Date object. The best dialect-agnostic way is to pass "CAST('2019-01-01' AS DATE)" for example.
    - `end_date`: The noninclusive end date of the range, as a SQL Date object. The best dialect-agnostic way is to pass "CAST('2019-01-01' AS DATE)" for example.
    - `start_date_raw`: The start date of the range, as a string in 'YYYY-MM-DD' format.
    - `end_date_raw`: The end date of the range, as a string in 'YYYY-MM-DD' format.

  _Returns:_
    A SQL SELECT statement that produces a table with a single column of dates.

  _Usage:_
  ```jinja
  WITH date_spine AS (
      {{ 
          sdf_utils.date_spine(
              datepart = 'day',
              start_date = "CAST('2019-01-01' AS DATE)",    
              end_date = "CAST('2019-12-31' AS DATE)"
          )
      }}
  )
  ```

  Alternatively, using the raw parameters:

  ```jinja
  WITH date_spine AS (
      {{ 
          sdf_utils.date_spine(
              datepart = 'day',
              start_date_raw = "2019-01-01",    
              end_date_raw = "2019-12-31"
          )
      }}
  )
  ```

  _Output:_
  ```sql
  WITH date_spine AS (
      WITH
      intervals_list AS (
          SELECT 
              ARRAY_GENERATE_RANGE(
                  0, 
                  DATEDIFF(day, CAST('2019-01-01' AS DATE), CAST('2019-12-31' AS DATE)) + 1
              ) AS i
      )

      SELECT 
          i.VALUE AS i,
          DATEADD(day, i.VALUE, CAST('2019-01-01' AS DATE)) AS date_spine
      FROM intervals_list l, LATERAL FLATTEN(input => l.i) i
  )
  ```

  

  _Dialects_:
  | Dialect | Supported |
  | ------- | --------- |
  | Snowflake | ✅ |
  | BigQuery | ❌ |
  | Redshift | ❌ |
  | Trino | ❌ |
  | Postgres | ❌ |


#### Generate Date Values

[Source Code](./macros/generate_date_values.jinja)

Generates a SQL VALUES clause with a list of date strings. The date strings are generated for each day in the range from the 'from' date to the 'to' date, inclusive.

  _Parameters:_
    - `from`: The start date of the range, as a string in 'YYYY-MM-DD' format.
    - `to`: The end date of the range, as a string in 'YYYY-MM-DD' format.

 _Returns:_
    A string representing a SQL VALUES clause with a list of date strings.

  _Usage:_
  ```jinja
  {{ sdf_utils.generate_date_values('2022-01-01', '2022-01-03') }}
  ```
  _Output:_
  ```sql
  (VALUES ('2022-01-01'), ('2022-01-02'), ('2022-01-03'))
  ```

#### Generate Integer Values

[Source Code](./macros/generate_integer_values.jinja)

Generates a SQL VALUES clause with a list of incrementing integers. The 'from' int to the 'to' int are inclusive.

  _Parameters:_
    - `from`: The starting integer of the range
    - `to`: The ending integer of the range

  _Returns:_
    A string representing a SQL VALUES clause with a list of incrementing integers.

  _Usage:_
  ```jinja
  {{ sdf_utils.generate_integer_values(1,5) }}
  ```
  _Output:_
  ```sql
  (VALUES 1,2,3,4,5)
  ```

#### Group By

[Source Code](./macros/group_by.jinja)

Builds a group by statement for fields 1...N

  _Parameters:_
    - `n`: The number of fields to group by

  _Returns:_
    A string representing a SQL GROUP BY clause with a list of incrementing integers.

  _Usage:_
  ```jinja
  {{ sdf_utils.group_by(5) }}
  ```

  _Output:_
  ```sql
  group by 1,2,3,4,5
  ```

#### Generate Surrogate Key

[Source Code](./macros/generate_surrogate_key.jinja)

Generates a surrogate key for a given list of fields. The surrogate key is a unique identifier for each record in a table. It is generated by concatenating the values of the given fields and applying the MD5 hash function to the result. The fields are cast to varchar type before concatenation. If a field value is null, it is replaced with the string '_jinja_surrogate_key_null_'.

  _Parameters:_
    - `fields`: A list of field names for which the surrogate key should be generated.

  _Returns:_
    A string representing a SQL GROUP BY clause with a list of incrementing integers.

  _Usage:_
  ```sql
 SELECT
      {{ sdf_utils.generate_surrogate_key(['column1', 'column2', 'column3']) }} AS surrogate_key,
  ```

  _Output:_
  ```sql
  SELECT
      md5(
          coalesce(cast(column1 as varchar),'_jinja_surrogate_key_null_')
           || '-' || coalesce(cast(column2 as varchar),'_jinja_surrogate_key_null_')
           || '-' || coalesce(cast(column3 as varchar),'_jinja_surrogate_key_null_')) AS surrogate_key,
  ```

  _Why the concatenated dash (`|| '-' ||`) in the output?_

  Without the delimiter, the inputs `firstName` = 'John Smith'  and `lastName` = '', and firstName = 'John ' and lastName = 'Smith' would generate the same hash with generate_surrogate_key(["firstName", "lastName"]). The delimiter prevents this collision

  Thanks [@sophieml](https://github.com/sophieml) from Obie for the delimiter improvement ^ <3
