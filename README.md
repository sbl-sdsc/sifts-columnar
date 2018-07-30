# sifts-columnar 
Benchmark for the encoding of PDB to UniProt residue-level mappings from the EBI SIFTS resource in compressed columnar dataformats.


The [SIFTS] [1](https://www.ebi.ac.uk/pdbe/docs/sifts/overview.html)(Structure Integration with Function, Taxonomy and Sequence) project provides residue-level mappings between PDB sequence, PDB residues, and UniProt residues.

SIFTS provides mapping xml files for each PDB entry (> 140,000 files), e.g.,[1xyz.xml.gz](ftp://ftp.ebi.ac.uk/pub/databases/msd/sifts/xml/1xyz.xml.gz). 

In this project we explore the use of efficient columnar dataformats to represent the residue-level mappings for the entire PDB in a single file for efficient download and processing.

Columnar dataformats can achive unprecedented levels of compression due to the columnar data respresentation and columnwise packing strategies including built-in delta- and run-length encoding, followed by entropy encoding.

## Dataset
SIFTS residue-level mappings were downloaded on July 28, 2018 and resulted in 105,594,955 residue level mappings. The encoded files where generated with the [CreatePdbToUniProtMappingFile](https://github.com/sbl-sdsc/mmtf-spark/blob/master/src/main/java/edu/sdsc/mmtf/spark/applications/CreatePdbToUniProtMappingFile.java) command line application.

## File Sizes
Data were converted to parquet[2] and orc[3] files and compressed with the available compression codecs available in Apache Spark.

Dataset name  | File format   | Compression codec | Size (MB)|
|:----------- |:------------- |:----------------- | --------:|
xml_gzip      | xml[1]        | gzip              |      tbd |
csv_gzip      | csv           | gzip              |    519.7 |
parquet_snappy| parquet       | snappy            |    145.1 |
parquet_gzip  | parquet       | gzip              | **57.9** |
orc_zlib      | orc           | zlib              |     41.9 |
orc_lzo       | orc           | lzo               | **41.7** |

The parquet files with gzip compression and the orc files with lzo compression are the best option for representing the SIFTS mapping data.

## Query Performance
In order to evaluate the performance of operating on these dataset, we setup 4 benchmarks for the optimal datasets (orc_lzo and parquet_gzip). The benchmarks were run on a MacBook Pro (Retina, 13-inch, Late 2013, 2.8 GHz Intel Core i7, 16 GB 1600 MHz DDR3, and SSD drive).

Benchmark  | orc_lzo (second) | parquet_gzip (seconds) |
|:-------- | ------------:| -------:|
 Count     |       * 3.7* |     4.1 | 
 Query     |      * 11.9* |    20.2 |
 Join      |       *12.0* |    23.3 |
 Convert   |        *6.0* |     7.9 |

Due to efficient indexing and predicate pushdown, the ORC file format  outperforms the parquet file format for this dataset.

A Jupyter Notebook of this benchmark is available QueryBenchMark

## Reading Benchmark (preliminary)
For this benchmark the entire dataset was encoded in two compressed columnar filed formats. Each file is then read completely into memory and the parsing times are reported in seconds.
--- obsolete data ---
| File format   | Compression codec | Size (MB)| Pandas[4] (s) | PySpark[5] (s) | Spark[6] (s)|
|:------------- |:----------------- | --------:| ----------:| -----------:| ---------:|
| xml (original)| gzip              |       nd |         na |          na |        na |
| csv           | gzip              |    507.9 |         nd |          nd |        nd |
| parquet[2]    | gzip              | **57.9** |     **86** |         164 |       177 |
| parquet[2]    | snappy            |    104.6 |         88 |         144 |       148 |
| orc[3]        | zlib              |     38.2 |         na |          92 |        92 |
| orc[3]        | lzo               | **37.3** |         na |      **85** |    **85** |

## References
[1] [Velankar et al., Nucleic Acids Research 41, D483 (2013)](https://doi.org/10.1093/nar/gks1258)

[2] [Apache Parquet](https://parquet.apache.org/)

[3] [Apache orc](https://orc.apache.org/)

[4] [Pandas](https://pandas.pydata.org/)

[5] [Apache Spark, Python API](https://spark.apache.org/docs/latest/index.html)

[6] [Apache Spark, Java API](https://spark.apache.org/docs/latest/index.html)
