## Auxiliary files for benchmarking Apache Parquet

| File                                      | Description                                                                                                                           |
|-------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------|
| data/_metadata_wide_schema_50k_col_100_rg | Wide schema table to benchmark metadata deserialization performance. See [note](#wide-schema-metadata) for details. |

## Wide schema metadata

`data/_metadata_wide_schema_50k_col_100_rg` contains a schema with 50k float columns and 100 row groups. It is generated with the following code: 

```python
%%time
import numpy as np
import pyarrow as pa
import pyarrow.parquet as pq

row_groups = 100
columns = 50_000
chunk_size = 1
rows = row_groups * chunk_size
parquet_path = "synthetic_large_metadata_footer_dataset"

def get_table():
    # Generate a random 2D array of floats using NumPy
    # Each column in the array represents a column in the final table
    data = np.random.rand(rows, columns)

    # Convert the NumPy array to a list of PyArrow Arrays, one for each column
    pa_arrays = [pa.array(data[:, i]) for i in range(columns)]

    # Optionally, create column names
    column_names = [f'column_{i}' for i in range(columns)]

    # Create a PyArrow Table from the Arrays
    return pa.Table.from_arrays(pa_arrays, names=column_names)

table = get_table()

metadata_collector = []
pq.write_to_dataset(table, parquet_path, metadata_collector=metadata_collector, row_group_size=chunk_size,
                    use_dictionary=False, write_statistics=False, compression=None)
pq.write_metadata(table.schema, "_metadata", metadata_collector=metadata_collector)
```
