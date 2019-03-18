# entrez_direct
Tutorial on using E-utilities
REST API called the Entrez Utilities
Representational State Transfer (REST) Application Programming Interface (API)

# Extend tutorial from NCBI

See [here](https://www.ncbi.nlm.nih.gov/books/NBK179288/)


## Install e-utilities

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
  <!-- cpan
  > o conf make_install_make_command 'sudo make'
  > o conf commit
  > quit
  mount
  mount -n -o remount,suid / -->
  ./edirect/setup.sh

  ```

## Install xtract

```
Unable to locate xtract executable. Please execute the following:

  ftp-cp ftp.ncbi.nlm.nih.gov /entrez/entrezdirect xtract.Darwin.gz
  gunzip -f xtract.Darwin.gz
  chmod +x xtract.Darwin


~/xtract.Linux -help
```

## Intro to XML

[XML](https://www.sitepoint.com/really-good-introduction-xml/)

## Entrez Direct Functions

Navigation functions support exploration within the Entrez databases:

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



## Get assemblies from bioproject

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

