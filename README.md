# Tiled

*Disclaimer: This is very early work, still in the process of defining scope.*

Data analysis is easier and better when we load and operate on data in simple,
self-describing structures that keep our mind on the science rather the
book-keeping of filenames and file formats.

Tiled is a **data access** tool that enables **search** and **structured,
chunkwise access** to data in a **variety of formats**, regardless of the
format the data happens to be stored in at rest. Like Jupyter, Tiled can be
used solo or deployed as a shared resource. Tiled provides data, locally or
over the web, in a choice of formats, spanning slow but widespread
interchange formats (e.g. CSV, JSON, TIFF) and fast, efficient ones (e.g. C
buffers, Apache Arrow DataFrames). Tiled enables slicing and sub-selection
for accessing only the data of interest, and it enables parallelized download
of many chunks at once. Users can access data with very light software
dependencies and fast partial downloads.

Serve a "Catalog", a Python object backed by some generated data, directory
of files, network resource, or database

```
tiled serve pyobject tiled.examples.generated:demo
```

And then access the data efficiently via the Python client, a web browser, or
any HTTP client.

```python
>>> from tiled.client.catalog import Catalog

>>> catalog = Catalog.from_uri("http://localhost:8000")

>>> catalog
<Catalog {'arrays', 'dataframes', 'xarrays', 'nested', ...} ~5 entries>

>>> catalog['arrays']
<Catalog {'large', 'medium', 'small', 'tiny'}>

>>> catalog['arrays']['medium']
<ClientDaskArrayAdapter>

>>> catalog['arrays']['medium'][:]
array([[0.21267816, 0.59685753, 0.12483017, ..., 0.74891246, 0.43889019,
        0.27761903],
       [0.95434218, 0.31376234, 0.05776443, ..., 0.53886856, 0.92855426,
        0.32506382],
       [0.0458622 , 0.0561961 , 0.3893611 , ..., 0.23124064, 0.40311252,
        0.22488572],
       ...,
       [0.91990991, 0.98361972, 0.26394368, ..., 0.86427576, 0.00436757,
        0.03021872],
       [0.26595236, 0.18207517, 0.18989639, ..., 0.16221733, 0.59052007,
        0.94255651],
       [0.4721781 , 0.01424852, 0.57294198, ..., 0.70392867, 0.69371454,
        0.228491  ]])

>>> catalog['dataframes']
<Catalog {'df'}>

>>> catalog['dataframes']['df']
<ClientDaskDataFrameAdapter ['A', 'B', 'C']>

>>> catalog['dataframes']['df'][['A', 'B']]
              A         B
index                    
0      0.748885  0.769644
1      0.071319  0.364743
2      0.322665  0.897854
3      0.328785  0.810159
4      0.158253  0.822505
...         ...       ...
95     0.913758  0.488304
96     0.969652  0.287850
97     0.769774  0.941785
98     0.350033  0.052412
99     0.356245  0.683540

[100 rows x 2 columns]
```

Using an Internet browser or a commandline HTTP client like
[curl](https://curl.se/) or [httpie](https://httpie.io/) you can download the
data in whole or in efficiently-chunked parts in the format of your choice:

```
# Download tabular data as CSV
http://localhost:8000/dataframe/full/dataframes/df?format=csv

# or XLSX (Excel)
http://localhost:8000/dataframe/full/dataframes/df?format=xslx

# and subselect columns.
http://localhost:8000/dataframe/full/dataframes/df?format=xslx&column=A&column=B

# View or download (2D) array data as PNG
http://localhost:8000/array/full/arrays/medium?format=png

# and slice regions of interest.
http://localhost:8000/array/full/arrays/medium?format=png&slice=:50,100:200
```

Web-based data access usually involves downloading complete files, in the
manner of [Globus](https://www.globus.org/); or using modern chunk-based
storage formats, such as [TileDB](https://tiledb.com/) and
[Zarr](https://zarr.readthedocs.io/en/stable/) in local or cloud storage; or
using custom solutions tailored to a particular large dataset. Waiting for an
entire file to download when only the first frame of an image stack or a
certain column of a table are of interest is wasteful and can be prohibitive
for large longitudinal analyses. Yet, it is not always practical to transcode
the data into a chunk-friendly format or build a custom tile-based-access
solution. (Though if you can do either of those things, you should consider
them instead!)

In more technical language, Tiled is a service providing "tiled" chunk-based
access to strided arrays, tablular datasets ("dataframes"), and nested
structures thereof. It is centered on these *structures* (backed by numpy,
pandas, xarray, various mappings in the server) rather than particular
formats. The structures may be read from an extensible range of formats; the
web client receives them as one of an extensible range of MIME types, and it
can pose requests that sub-select and slice the data before it is read or
served. The server incorporates search capability, which may be backed by a
proper database solution at large scales or a simple in-memory index at small
scales, as well as access controls. A Python client provides a friendly
h5py-like interface and supports offline caching.