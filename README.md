[![SDF Workspace Compiles](https://github.com/sdf-labs/utils/actions/workflows/compile-workspace.yml/badge.svg)](https://github.com/sdf-labs/utils/actions/workflows/compile-workspace.yml)

# Official SDF Utils Library

This SDF library contains the standard utilities available in the default namespace in any SDF workspace. As such, macros defined in this workspace do not need to be prefixed with the workspace name, they can be referenced directly. Here's an example:

```sql
SELECT 
  date
FROM
  dates
WHERE
  date IN {{ generate_date_strings('2020-01-01', '2020-01-10') }}
```

*Note: SDF is still < v1, as such certain scenarios may result in unexpected behavior. Please follow the [contributing guidelines](./CONTRIBUTING.md) and create an issue in this repo if you find any bugs or issues.*

For an in-depth guide on how to use Jinja macros in SDF, please see the Jinja section of [our official docs](https://docs.sdf.com/guide/macro-processing/jinja)
