# entrez_direct
Tutorial on using E-utilities

# Extend tutorial from NCBI

See [here](https://www.ncbi.nlm.nih.gov/books/NBK179288/)


## Install

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

`esearch -db bioproject -query PRJNA429695 | elink -target biosample | efetch -format docsum | ./xtract.Linux -pattern DocumentSummary -block Accession -element Accession | while read line; do ACC=$(echo "esearch -db nucleotide -query ${line} | efetch -format docsum | ~/xtract.Linux -pattern DocumentSummary -element Caption | head -n 1" | sh); echo -e ${line}'\t'${ACC}; done > fmicb2018_00771.txt`
`less fmicb2018_00771`
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

esearch -db bioproject -query PRJNA429695 | elink -target biosample | efetch -format docsum | ./xtract.Linux -pattern DocumentSummary -block Accession -element Accession

cat fmicb2018_00771.txt | while read samn acc; do esearch -db nucleotide -query ${set lineCount 0acc} | efetch -format docsum; done

while read -r samn acc; do echo "esearch -db nucleotide -query ${acc} | efetch -format gbwithparts" | sh > ${samn}.gbk; done < fmicb2018_00771.txt



