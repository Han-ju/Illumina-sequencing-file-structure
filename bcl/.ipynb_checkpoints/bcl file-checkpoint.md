# MiSeq/HiSeq System

Regacy file structure of Illumina sequencing systems.  
File contains header of 4 bytes representing number of clusters containing.

* structure of bcl file

Byte | type | value
:---:|:---:|:---:
0 ~ 3 | `unsinged 4 bytes integer` |Number of clusters in the **tile**
-----|-----|-----
4 | `custom data type` | Basecalled data of 1st cluster
5| `custom data type` | Basecalled data of 2nd cluster
...||
4+N-1 | `custom data type` | Basecalled data of Nth cluster

* structure of `custom data type`

Bit | type | value
:---:|:---:|:---:
0 ~ 5 | `unsinged integer` | Phred score of basecall
6 ~ 7 | `unsinged integer` | Base [A, C, G, T] for [0, 1, 2, 3]
||All bits 0 in a byte is reserved for no-call.



You can load bcl file data with following python codes:
```python
import numpy as np
from struct import unpack

def load_bcl(path, lane, tile, cycle):
    with open(f'{path}/Data/Intensities/BaseCalls/L{lane:03d}/C{cycle}.1/s_{lane}_{tile}.bcl.gz', 'rb') as fin:
        NUMCLUSTER = unpack('<I', fin.read(4))
        data = np.frombuffer(fin.read(NUMCLUSTER), dtype=np.uint8)
    base = np.mod(data, 4)
    base[data == 0] = 0
    score = np.right_shift(reads, 2)
    return base, score
```

# NextSeq System

New file structure of Illumina sequencing systems.  
File is no more seperated in lane.  
To excess specific tile data, `bci` file is provided

* structure of bci file

Byte | type | value
:---:|:---:|:---:
0 ~ 3 | `unsinged 4 bytes integer` | Index of 1st tile
4 ~ 7 | `unsinged 4 bytes integer` | Number of clusters in the 1st tile
8 ~ 11 | `unsinged 4 bytes integer` | Index of 2nd tile
12 ~ 15 | `unsinged 4 bytes integer` | Number of clusters in the 2nd tile
...||
8N-8 ~ 8N-5 | `unsinged 4 bytes integer` | Index of Nth tile
8N-4 ~ 8N-1 | `unsinged 4 bytes integer` | Number of clusters in the Nth tile

* structure of bcl file

Byte | type | value
:---:|:---:|:---:
0 ~ 3 | `unsinged 4 bytes integer` |Number of clusters in the **lane**
-----|-----|-----
4 | `custom data type` | Basecalled data of 1st cluster of first tile
5| `custom data type` | Basecalled data of 2nd cluster of first tile
...||
4+N-1 | `custom data type` | Basecalled data of Nth cluster of last tile

* structure of `custom data type`

Bit | type | value
:---:|:---:|:---:
0 ~ 5 | `unsinged integer` | Phred score of basecall
6 ~ 7 | `unsinged integer` | Base [A, C, G, T] for [0, 1, 2, 3]
||All bits 0 in a byte is reserved for no-call.

You can load cif file data with following python codes:
```python
import numpy as np
from struct import unpack

def load_bcl(path, lane, tile, cycle):
    with open(f'{path}/Data/Intensities/BaseCalls/L{lane:03d}/s_{lane}.bci', 'rb') as fin:
        startCluster = 0
        while reads := fin.read(8):
            currentTile, numCluster = unpack('II', reads)
            if currentTile == tile:
                break
            else:
                startCluster += numCluster

    with open(f'{path}/Data/Intensities/BaseCalls/L{lane:03d}/C{cycle}.1/s_{lane}_{tile}.bcl.bgzf', 'rb') as fin:
        NUMCLUSTER = unpack('<I', fin.read(4))
        assert numCluster == NUMCLUSTER
        data = np.frombuffer(fin.read(NUMCLUSTER), dtype=np.uint8)
    base = np.mod(data, 4)
    base[data == 0] = 0
    score = np.right_shift(reads, 2)
    return base, score
```