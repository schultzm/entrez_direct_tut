# Extended tutorial from NCBI

For an in-depth tutorial on the edirect software, refer to [https://www.ncbi.nlm.nih.gov/books/NBK179288/](https://www.ncbi.nlm.nih.gov/books/NBK179288/).

# Custom entrez_direct tutorial

The tutorial below aims to give a basic overview of the Entrez Direct (edirect) Representational State Transfer (REST) Application Programming Interface (API).  This system, known as `edirect`, is used to access the National Center for Biotechnology Information (NCBI) Entrez database.  Entrez is a web-accessible molecular biology database that provides integrated access to nucleotide and protein sequence data, gene-centered and genomic mapping information, 3D structure data, PubMed MEDLINE, and more.
[https://www.ncbi.nlm.nih.gov/Web/Search/entrezfs.html](https://www.ncbi.nlm.nih.gov/Web/Search/entrezfs.html)



## Install software
### edirect

First up, we need to install the edirect suite of tools.  The software is written in the [Perl](https://www.perl.org/) programming language.  [Instructions for installation](https://www.ncbi.nlm.nih.gov/books/NBK179288/) are copied below.  Paste these commands into a terminal window.


```
cd ~
/bin/bash
perl -MNet::FTP -e \
  '$ftp = new Net::FTP("ftp.ncbi.nlm.nih.gov", Passive => 1);
   $ftp->login; $ftp->binary;
   $ftp->get("/entrez/entrezdirect/edirect.tar.gz");'
gunzip -c edirect.tar.gz | tar xf -
rm edirect.tar.gz
builtin exit
export PATH=${PATH}:$HOME/edirect >& /dev/null || setenv PATH "${PATH}:$HOME/edirect"
./edirect/setup.sh
```


Results of an edirect query are returned to stdout in human readable text as [xml](https://www.sitepoint.com/really-good-introduction-xml/), [json](https://en.wikipedia.org/wiki/JSON) and (asn.1)[https://www.ncbi.nlm.nih.gov/Structure/asn1.html] formats.
### xtract

To parse xml output, we can use the tool `xtract`.
To install `xtract`, run the following commands in a terminal window but choose the appropriate version for your operating environment (see ftp://ftp.ncbi.nlm.nih.gov/entrez/entrezdirect/ for available versions):

```
#Mac use xtract.Darwin, Linux use xtract.Linux, windows I think use the
ftp-cp ftp.ncbi.nlm.nih.gov /entrez/entrezdirect xtract.Linux.gz
gunzip -f xtract.Linux.gz
chmod +x xtract.Linux

#To get help on xtract do:
~/xtract.Linux -help
#or
./pathtoxtract/xtract.version -help
#for linux this would be
./xtract.Linux -help
#or if you add it to your path, or put it somewhere that is in your path, for example in ~/.local/bin/ just do:
xtract.Linux -help
```

## edirect Functions

The edirect functions allow you to query the NCBI entrez database from the command line.  This approach is powerful in that customised, advanced or simple queries can be built into scripts, automated, and executed in parallel (be careful not to exceed three queries per second).

The following navigation functions support exploration within the Entrez databases:

`esearch` performs a new Entrez search using terms in indexed fields.

`elink` looks up neighbors (within a database) or links (between databases).

`efilter` filters or restricts the results of a previous query.

Records can be retrieved in specified formats or as document summaries:

`efetch` downloads records or reports in a designated format.

Desired fields from XML results can be extracted without writing a program:

`xtract` converts EDirect XML output into a table of data values.

Several additional functions are also provided:

`einfo` obtains information on indexed fields in an Entrez database.

`epost` uploads unique identifiers (UIDs) or sequence accession numbers.

`nquire` sends a URL request to a web page or CGI service.

The above functions can be piped to one another to allow creativty on the part of the end user.  To get help on any function do `functionname -help`, example `efetch -help`.


## An example to get assemblies from bioproject
In this example we will examine bioproject PRJNA429695.  We will build up the command using pipe characters to eventually download the assemblies in genbank format.

First, search for the bioproject on entrez using `esearch`:

```
esearch -db bioproject -query PRJNA429695
```

Read the result by piping the hit to `efetch`
```
esearch -db bioproject -query PRJNA429695 | efetch -format docsum
```

Optionally format the result in `json` (which can be parsed using json parsers)

```
esearch -db bioproject -query PRJNA42969 | efetch -format docsum -mode json
```
In the above, the results returned from the running of one function (i.e., the standard out or `stdout`) are passed to the standard in (or `stdin`) of the next function using the pipe `|` character.



to `elink`, which uses the ouput of esearch to link into a search in the biosample database.  `efetch` fetches the hits in DocumentSummary format. `xtract.Linux` is used to extract the `SAMN` accessions. We then iterate through each accession and run `esearch`, `efetch` and `xtract` to get the caption (i.e., the NCBI refseq assembly accession).  For each record there are two captions (the refseq and the user submission), so we use `head` to retain only the first caption.  The first caption is stored inside a variable `ACC`, which captures the results of the previous search run inside a subshell `$()`.  The final part prints the `SAMN` (stored in the variable `line`) and the NCBI accession (stored in the variable `ACC`) in tab-delimited format.  the output is stored in the variable `ACC`.


```
esearch -db bioproject -query PRJNA429695 | elink -target biosample | efetch -format docsum | ./xtract.Linux -pattern DocumentSummary -block Accession -element Accession | while read line; do ACC=$(echo "esearch -db nucleotide -query ${line} | efetch -format docsum | ~/xtract.Linux -pattern DocumentSummary -element Caption | head -n 1" | sh); echo -e ${line}'\t'${ACC}; done > fmicb2018_00771.txt`
less fmicb2018_00771
```

```
SAMN08357826    NZ_CP025995
SAMN08357825    NZ_CP025996
SAMN08357824    NZ_CP025997
SAMN08357823    NZ_CP025998
SAMN08357822    NZ_CP025999
SAMN08357821    NZ_CP026000
SAMN08357820    NZ_CP026001
SAMN08357819    NZ_CP026002
SAMN08357818    NZ_CP026003
SAMN08357817    NZ_CP026004
```

```
esearch -db bioproject -query PRJNA429695 | elink -target biosample | efetch -format docsum | ./xtract.Linux -pattern DocumentSummary -block Accession -element Accession

cat fmicb2018_00771.txt | while read samn acc; do esearch -db nucleotide -query ${set lineCount 0acc} | efetch -format docsum; done

while read -r samn acc; do echo "esearch -db nucleotide -query ${acc} | efetch -format gbwithparts" | sh > ${samn}.gbk; done < fmicb2018_00771.txt
```


## Get reads from bioproject SRA

### Understanding SRA

[help!](https://www.ncbi.nlm.nih.gov/books/NBK56913/)


```
esearch -db bioproject -query PRJNA383436 | elink -target biosample | efetch -format docsum | ./xtract.Linux -pattern DocumentSummary -block Ids -element Id -group SRA > accessions_demo.txt
#BIOSAMPLE    Sample_name    SRA_sample_accession
SAMN06765483	Enterobacter cloacae MS7884A	SRS2169477
SAMN07173918	Enterobacter cloacae MS8079	SRS2350264
SAMN07173917	Enterobacter cloacae MS8078	SRS2350262
SAMN07173916	Enterobacter cloacae MS8077	SRS2350261
SAMN07173915	Enterobacter cloacae MS7926	SRS2350260
SAMN07173914	Escherichia coli MS7925	SRS2350259
SAMN07173913	Enterobacter cloacae MS7924	SRS2350258
SAMN07173912	Enterobacter cloacae MS7923	SRS2350257
SAMN07173911	Enterobacter cloacae MS7893	SRS2350273
SAMN07173910	Enterobacter cloacae MS7892	SRS2350272
SAMN07173909	Enterobacter cloacae MS7891	SRS2350268
SAMN07173908	Enterobacter cloacae MS7890	SRS2350269
SAMN07173907	Enterobacter cloacae MS7889	SRS2350271
SAMN07173906	Enterobacter cloacae MS7888	SRS2350270
SAMN07173905	Enterobacter cloacae MS7887	SRS2350265
SAMN07173904	Enterobacter cloacae MS7886	SRS2350263
SAMN07173903	Enterobacter cloacae MS7885	SRS2350267
SAMN07173902	Enterobacter cloacae MS7884	SRS2350266
SAMN06831386	Enterobacter cloacae MS7884B	SRS2169478


```

