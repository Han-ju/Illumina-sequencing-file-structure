# MiSeq/HiSeq System

Regacy file structure of Illumina sequencing systems.  
File contains header of 13 bytes containing its identifier and metadata.

Byte | type | value | fixed value
:---:|:---:|:---:|:---:
0 ~  2 | `3 bytes string` |Identifier of CIF file |`CIF`
3      | `unsigned 1 byte integer` |Version of CIF file structure |`1`
4      | `unsigned 1 byte integer` |Byte size of each intensity data |`2`
5 ~  6 | `unsigned 2 bytes integer` |Start cycle index (1-based numbering)|
7 ~  8 | `unsigned 2 bytes integer` |Number of cycle|`1`
9 ~ 12 | `unsigned 4 bytes integer` |Number of clusters|
-----|----- |-----|-----
13+0 ~ 13+1 | `signed 2 bytes integer` |channel 1 intensity of 1st cluster
13+2 ~ 13+3 | `signed 2 bytes integer` |channel 1 intensity of 2nd cluster
...
13+2N-2 ~ 13+2N-1 | `signed 2 bytes integer` |channel 1 intensity of Nth cluster
13+2N   ~ 13+2N+1 | `signed 2 bytes integer` |channel 2 intensity of 1st cluster
...
13+4N-2 ~ 13+4N-1 | `signed 2 bytes integer` |channel 2 intensity of Nth cluster
13+4N   ~ 13+4N+1 | `signed 2 bytes integer` |channel 3 intensity of 1st cluster
...
13+6N-2 ~ 13+6N-1 | `signed 2 bytes integer` |channel 3 intensity of Nth cluster
13+6N   ~ 13+6N+1 | `signed 2 bytes integer` |channel 4 intensity of 1st cluster
...
13+8N-2 ~ 13+8N-1 | `signed 2 bytes integer`| channel 4 intensity of Nth cluster

You can load cif file data with following python codes:
```python
import numpy as np
from struct import unpack

def load_cif(path, lane, tile, cycle, numChannel=4):
    with open(f'{path}/Data/Intensities/L{lane:03d}/C{cycle}.1/s_{lane}_{tile}.cif', 'rb') as fin:
        CIF, VERSION, BYTESIZE, CYCLE, NUMCYCLE, NUMCLUSTER = unpack('<3sBBHHI', fin.read(13))
        assert (CIF, VERSION, BYTESIZE, CYCLE, NUMCYCLE) == (b'CIF', 1, 2, cycle, 1)
        intensities = np.frombuffer(fin.read(BYTESIZE * NUMCLUSTER * numChannel), dtype=np.int16).reshape(numChannel, NUMCLUSTER)
        # intensities.shape = (number_of_channels, number_of_clusters)
    return intensities
```

# NextSeq System

New file structure of Illumina sequencing systems.  
File doesn't contain header.
Note that intensities axeses are transposed compared to legacy structure.

Byte | type | value
:---:|:---:|:---:
0 ~ 1 | `signed 2 bytes integer` |channel 1 intensity of 1st cluster
2 ~ 3 | `signed 2 bytes integer` |channel 2 intensity of 1st cluster
4 ~ 5 | `signed 2 bytes integer` |channel 1 intensity of 2nd cluster
6 ~ 7 | `signed 2 bytes integer` |channel 2 intensity of 2nd cluster
...||
2N-3 ~ 2N-4 | `signed 2 bytes integer` |channel 1 intensity of Nth cluster
2N-2 ~ 2N-1 | `signed 2 bytes integer` |channel 2 intensity of Nth cluster
- intensitiy value -32768 (-2^15) may reserved as NULL.

You can load cif file data with following python codes:
```python
import numpy as np

def load_cif(path, lane, tile, cycle, numChannel=2):
    with open(f'{path}/Data/Intensities/L{lane:03d}/C{cycle}.1/s_{lane}_{tile}.cif', 'rb') as fin:
        intensities = np.frombuffer(fin.read(), dtype=np.int16).reshape(-1, numChannel)
        # intensities.shape = (number_of_clusters, number_of_channels)
    return intensities
```