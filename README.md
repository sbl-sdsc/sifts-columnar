# sifts-columnar 
Benchmark for the encoding of PDB to UniProt residue-level mappings from the EBI SIFTS resource in compressed columnar dataformats.


The [SIFTS] [1](https://www.ebi.ac.uk/pdbe/docs/sifts/overview.html)(Structure Integration with Function, Taxonomy and Sequence) project provides residue-level mappings between PDB sequence, PDB residues, and UniProt residues.

SIFTS provides mapping xml files for each PDB entry (> 140,000 files), e.g.,[1xyz.xml.gz](ftp://ftp.ebi.ac.uk/pub/databases/msd/sifts/xml/1xyz.xml.gz). 

In this project we explore the use of efficient columnar dataformats to represent the residue-level mappings for the entire PDB in a single file for efficient download and processing.

Columnar dataformats can achive unprecedented levels of compression due to the columnar data respresentation and columnwise packing strategies including built-in delta- and run-length encoding, followed by entropy encoding.

## Dataset
SIFTS residue-level mappings were downloaded (incremental update) on July 27, 2018 and resulted in 105,594,971 residue level mappings. The encoded files where generated with the [CreatePdbToUniProtMappingFile](https://github.com/sbl-sdsc/mmtf-spark/blob/master/src/main/java/edu/sdsc/mmtf/spark/applications/CreatePdbToUniProtMappingFile.java) command line application.

## Reading Benchmark (preliminary)
For this benchmark the entire dataset was encoded in two compressed columnar filed formats. Each file is then read completely into memory and the parsing times are reported in seconds.

| File format   | Compression codec | Size (MB)| Pandas[4] (s) | PySpark[5] (s) | Spark[6] (s)|
|:------------- |:----------------- | --------:| ----------:| -----------:| ---------:|
| xml (original)| gzip              |       nd |         na |          na |        na |
| csv           | gzip              |    507.9 |         nd |          nd |        nd |
| parquet[2]    | gzip              | **58.9** |     **86** |         164 |       177 |
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
