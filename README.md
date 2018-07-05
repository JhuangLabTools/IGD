# iGD: Reshape and integrate large-scale data sources for highly efficient genome analysis

## Summary
Genome analysis usually requires comparing one set of genomic loci (region set) to many other annotated region sets that may come from one or more large-scale data sources. A wide variety of computational tools, such as LOLA, BART, Bedtools, Tabix, GenomeRunner and Giggle, have been developed for this purpose. Among these tools, Giggle claims to be over three orders of magnitude faster than others in searching millions to billions of genomic intervals. **iGD**, an integrated genome database, takes a database approach that directly integrates large-scale data sources into an easily searchable database by reshaping the data records. As a database, iGD takes not only the genomic regions, but also the signal levels and/or strand info, which makes the parameterized query (dynamic searching) an easy task. The searching speed of iGD is one to two orders of magnitude faster than Giggle while the database size is several times smaller than that of the Giggle index files. 


## Introduction
Genome data sources, such as Roadmap, UCSC, and Cistrome, contain thousand of annotated data sets and each dataset has thousands to millions of genomic regions. 
The enrichment analysis is to find overlaps between regions of all data sets in the data source and all regions in the query data set. This becomes challenging when the size of data source regions and the size of query regions are large. For example, UCSC human data source contains ~7 billion of regions or intervals, if the query set has 100,000 interavls, then a total of 100 trillion comparisons may be involved. To effectively search the overlaps, different indexing techniques have been developed. For example, Giggle employs a B+ tree to build a bunch of pure indexing files and then a qurey is carried out on these indexing files instead of the original data. With the indexing, the actual searching space is significantly reduced.
 
The goal of iGD is to build a database that integrates all genomic data sets in one or more data sources and minimizes the actual searching space for a general query. To achieve this, iGD reshapes the data sources by dividing the genome into a large number of equal-size bins (~200,000 bins), and arranging data records of the data sources into the bin or bins they intersect. Data in each bin will be saved as a file (mode 0) or as a block of a single file (mode 1). Mode 0 iGD database  can be extended easily while using mode 1 database is slightly faster for searching.  For mode 0, each file is named according to the index of the bin and for mode 1 the file head contains the location of the bins and the number of iGD data elements in each bin. Each iGD data element contains the index of the original dataset, the original genomic region (start and end coordinates), and a value (signal level and/or strand info). To find overlaps of a query, one only needs to load one or a few bin files (mode 0) or bin block data (mode 1) instead of all data sets, which minimizes the data loading time; and more importantly, the comparisons are carried out only in the loaded one or a few bins, which minimizes the actual searching space. Details about the implementation (a link) will be provided later. 
 

## How to run iGD
Clone the site including the subfolders. All `.c` programs can be compiled and run on their own -- **no dependency among them**. These programs are tested on Linux systems and the installation of some librays like zlib may be needed.


### 1. Create iGD database from a genome data source
To compile `igd_create.c` on a linux terminal: 
```
gcc -o igd_create igd_create.c -lm -lz
```

To run the executable `igd_create`: 
```
./igd_create "/path/to/data_source_folder/*" "/path/to/igd_folder/" "databaseName"
``` 

where:

- `"path/to/data_source_folder"` is the path of the folder that contains `.bed.gz` data files (function to process non-gz bed files will be added later)

- `"path/to/igd_folder"` is the path to the output igd folder: this folder should be made first with mkdir and it should contain a subfolder named as data0, where data0 should contain 24 subfolders: chr1, chr2, ..., chr22, chrX and chrY.

- `"databaseName"` is the name you give to the database, for eaxmple, `"roadmap"`

An example: `"rme"` is an example folder containing `.bed.gz` files, `"rme_igd"` is an example igd database folder
```
./igd_create "rme/*" "rme_igd/" "roadmap"
```

This will generate a total of ~200,000 igd bin files (mode 0) in the subfolders chr1,...chrY; a single igd database file (mode 1) `roadmap.igd` and dataset index file `roadmap_index.tsv` in the igd folder.


### 2. Search iGD for overlaps
To compile `igd_search.c` on a linux terminal:
```
gcc -o igd_search igd_search.c -lm -lz
```

To run the executable `igd_search`: 
```
./igd_search "path/to/query_file" "path/to/igd_data_file"
```

where:

- `path/to/query_file` is the path to the query file

- `path/to/igd_data_file` is the path to the igd data (mode 1)

An example: `query100.bed` is an example query file
```
./igd_search query100.bed rme_igd/roadmap.igd
```

