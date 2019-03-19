# Extended tutorial from NCBI

For an in-depth tutorial on the edirect software, refer to [https://www.ncbi.nlm.nih.gov/books/NBK179288/](https://www.ncbi.nlm.nih.gov/books/NBK179288/).

# Custom entrez_direct tutorial

The tutorial below aims to give a basic overview of the Entrez Direct (edirect) Representational State Transfer (REST) Application Programming Interface (API).  This system, known as `edirect`, is used to access the National Center for Biotechnology Information (NCBI) Entrez database.  Entrez is a web-accessible molecular biology database that provides integrated access to nucleotide and protein sequence data, gene-centered and genomic mapping information, 3D structure data, PubMed MEDLINE, and more.
[https://www.ncbi.nlm.nih.gov/Web/Search/entrezfs.html](https://www.ncbi.nlm.nih.gov/Web/Search/entrezfs.html)



## Install software
### edirect

First up, we need to install the edirect suite of tools.  The software is written in the [Perl](https://www.perl.org/) programming language.  [Instructions for installation](https://www.ncbi.nlm.nih.gov/books/NBK179288/) are copied below.  Paste these commands into a terminal window and hit enter.


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


Succesful results of any edirect query are returned to stdout in human readable text as [xml](https://www.sitepoint.com/really-good-introduction-xml/), [json](https://en.wikipedia.org/wiki/JSON) and [asn.1](https://www.ncbi.nlm.nih.gov/Structure/asn1.html) formats.  Errors are returned to standard error (stderr).  We can use the tool `xtract` to parse xml output.

### xtract

To install `xtract`, run the following commands in a terminal window but choose the appropriate version for your operating environment.  See [here](http://ftp.ncbi.nlm.nih.gov/entrez/entrezdirect/) for available versions:

```
#Mac use xtract.Darwin, Linux use xtract.Linux, windows I think use the cygwin version.
ftp-cp ftp.ncbi.nlm.nih.gov /entrez/entrezdirect xtract.Linux.gz
gunzip -f xtract.Linux.gz
#make it executable
chmod +x xtract.Linux

#Add it to your path, or put it somewhere that is in your path, for example in ~/.local/bin/ so that you can get help by doing:
xtract.Linux -help
```

## edirect Functions

The edirect functions allow you to query the NCBI entrez database from the command line.  This approach is powerful in that customised queries can be automated and reproduced by writing them into scripts.

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

stdout of the functions can be piped to standard in (stdin) of another function, allowing creativty on the part of the end user.  To get help on any function do `functionname -help`, example `efetch -help`.


## Example 1: Get RefSeq assemblies from a BioProject

### Check existence of BioProject and return a DocumentSummary of this record
In this example we will examine bioproject PRJNA429695.  We will build up the command to connect the stdout of `esearch`, to the stdin of `efetch`, from the which the stdout goes to stdin of `elink`, from which the returned stdout is passed to `xtract` to grab desired fields.  Some knowledge of bash programming (shell scripting) will help out here.

First, check that the target bioproject exists by searching for the PRJ accession using `esearch`:

```
esearch -db bioproject -query PRJNA429695
```

The stdout shows a count of `1`, indicating a single hit, after going in `1` step:
```
<ENTREZ_DIRECT>
  <Db>bioproject</Db>
  <WebEnv>NCID_1_34836435_130.14.22.76_9001_1552959566_1290483952_0MetA0_S_MegaStore</WebEnv>
  <QueryKey>1</QueryKey>
  <Count>1</Count>
  <Step>1</Step>
</ENTREZ_DIRECT>
```

To fetch the document summary from this source, pipe the stdout of `esearch` to stdin of `efetch` using the pipe character
```
esearch -db bioproject -query PRJNA429695 | efetch -format docsum
```

The result should be

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE DocumentSummarySet PUBLIC "-//NLM//DTD esummary bioproject 20140903//EN" "https://eutils.ncbi.nlm.nih.gov/eutils/dtd/20140903/esummary_bioproject.dtd">

<DocumentSummarySet status="OK">
<DbBuild>Build190318-0700.1</DbBuild>

<DocumentSummary><Id>429695</Id>
	<TaxId>0</TaxId>
	<Project_Id>429695</Project_Id>
	<Project_Acc>PRJNA429695</Project_Acc>
	<Project_Type>Primary submission</Project_Type>
	<Project_Data_Type>Genome sequencing</Project_Data_Type>
	<Sort_By_ProjectType>78095</Sort_By_ProjectType>
	<Sort_By_DataType>99665</Sort_By_DataType>
	<Sort_By_Organism>311278</Sort_By_Organism>
	<Project_Subtype></Project_Subtype>
	<Project_Target_Scope>Multispecies</Project_Target_Scope>
	<Project_Target_Material>Genome</Project_Target_Material>
	<Project_Target_Capture>Whole</Project_Target_Capture>
	<Project_MethodType>Sequencing</Project_MethodType>
	<Project_Method></Project_Method>
	<Project_Objectives_List>
		<Project_Objectives_Struct>
			<Project_ObjectivesType>Sequence</Project_ObjectivesType>
			<Project_Objectives></Project_Objectives>
		</Project_Objectives_Struct>
	</Project_Objectives_List>
	<Registration_Date>2018/01/12 00:00</Registration_Date>
	<Project_Name></Project_Name>
	<Project_Title>Stenotrophomonas maltophilia complex genospecies 1 and genospecies 2</Project_Title>
	<Project_Description>Complete genome sequences of ten Mexican environmental isolates of the Stenotrophomonas maltophilia complex classified as genospecies 1 and genospecies 2 by multilocus sequence analysis.</Project_Description>
	<Keyword></Keyword>
	<Relevance_Agricultural></Relevance_Agricultural>
	<Relevance_Medical></Relevance_Medical>
	<Relevance_Industrial></Relevance_Industrial>
	<Relevance_Environmental>yes</Relevance_Environmental>
	<Relevance_Evolution></Relevance_Evolution>
	<Relevance_Model></Relevance_Model>
	<Relevance_Other></Relevance_Other>
	<Organism_Name></Organism_Name>
	<Organism_Strain></Organism_Strain>
	<Organism_Label></Organism_Label>
	<Sequencing_Status>Chromosome(s)</Sequencing_Status>
	<Submitter_Organization>Centro de Ciencias Genomicas - UNAM</Submitter_Organization>
	<Submitter_Organization_List>
		<string>Centro de Ciencias Genomicas - UNAM</string>
	</Submitter_Organization_List>
	<Supergroup></Supergroup>
</DocumentSummary>

</DocumentSummarySet>
```

Optionally format the result in `json` (to be parsed using json parsers instead of xml parsers as described in this tutorial).

```
esearch -db bioproject -query PRJNA42969 | efetch -format docsum -mode json
```

### Find accession numbers of BioSamples in the Bioproject

Now that we know the BioProject accession is valid, let's get the BioSample accession numbers from the BioProject to work with in our downstream searches. To this end, we will link the results of the `esearch` on BioProject to the BioSample database using `elink`.

```
esearch -db bioproject -query PRJNA429695 | elink -target biosample
```

The result shows a count of `10` hits accessed by going in `2` steps.

```
<ENTREZ_DIRECT>
  <Db>biosample</Db>
  <WebEnv>NCID_1_34881429_130.14.18.97_9001_1552960223_1142337136_0MetA0_S_MegaStore</WebEnv>
  <QueryKey>3</QueryKey>
  <Count>10</Count>
  <Step>2</Step>
</ENTREZ_DIRECT>
```

We will now parse the document summaries of the above 10 hits to get the accessions using `xtract`.  Note, substitute the `xtract.Linux` command to whatever is required to get `xtract` to run on your system.
```
esearch -db bioproject -query PRJNA429695 | elink -target biosample | efetch -format docsum | xtract.Linux -pattern DocumentSummary -block Accession -element Accession
```

The result is
```
SAMN08357826
SAMN08357825
SAMN08357824
SAMN08357823
SAMN08357822
SAMN08357821
SAMN08357820
SAMN08357819
SAMN08357818
SAMN08357817
```

Great, we now have the BioSample accessions.  Our next problem is getting and keeping a record of which RefSeq assembly accessions and strains align with these BioSamples.  Let's store the BioSample accessions from the search in the variable BISOAMPLES using a bash subshell (`$(dostuff)`).

`BIOSAMPLES=$(esearch -db bioproject -query PRJNA429695 | elink -target biosample | efetch -format docsum | xtract.Linux -pattern DocumentSummary -block Accession -element Accession | xargs)`

Look at the variable with `echo ${BIOSAMPLES}`.

Now iterate through the variable BIOSAMPLES, query entrez for each BIOSAMPLE using the edirect functions.  At each iteration,capture the DocumentSummary in the variable DOCSUM (so that we can just extract from this variable rather than having to use the slower method of re-quering entrez).  Using `xtract` we will parse DOCSUM and extract `STRAIN` and `ASSEMBLY` info, storing the metadata in the file `MDATA`.

```
MDATA="mdata.tab"
echo -e "BioSample\tStrain\tAssembly" >> ${MDATA} #Put a header in the file
for BIOSAMPLE in ${BIOSAMPLES[@]}
do
    DOCSUM=$(esearch -db assembly -query ${BIOSAMPLE} | efetch -format docsum)
    STRAIN=$(echo ${DOCSUM} | xtract.Linux -pattern DocumentSummary -block Infraspecie -element Sub_value)
    ASSEMBLY=$(echo ${DOCSUM} | xtract.Linux -pattern DocumentSummary -block Synonym -element RefSeq)
    echo -e ${BIOSAMPLE}'\t'${STRAIN}'\t'${ASSEMBLY} >> ${MDATA}
done
```

Finally, look in and iterate through the lines in the MDATA file, `efetch` a genbank `ASSEMBLY` and for each iteration and save each result in `[ASSEMBLY].gbk`.  This operation will be split up into three parallel operations using [GNU Parallel](https://www.gnu.org/software/parallel/).
```
cat ${MDATA} | while read BS ST AS
do
    echo "esearch -db nucleotide -query ${ASSEMBLY} | efetch -format gbwithparts > ${ASSEMBLY}.gbk"
done | parallel -j 3 --bar {}
```

Putting it all together, we would run the following block of commands:

```
BIOSAMPLES=$(esearch -db bioproject -query PRJNA429695 | elink -target biosample | efetch -format docsum | xtract.Linux -pattern DocumentSummary -block Accession -element Accession | xargs)
MDATA="mdata.tab"
echo -e "BioSample\tStrain\tAssembly" >> ${MDATA}
for BIOSAMPLE in ${BIOSAMPLES[@]}
do
    DOCSUM=$(esearch -db assembly -query ${BIOSAMPLE} | efetch -format docsum)
    STRAIN=$(echo ${DOCSUM} | xtract.Linux -pattern DocumentSummary -block Infraspecie -element Sub_value)
    ASSEMBLY=$(echo ${DOCSUM} | xtract.Linux -pattern DocumentSummary -block Synonym -element RefSeq)
    echo -e ${BIOSAMPLE}'\t'${STRAIN}'\t'${ASSEMBLY} >> ${MDATA}
done

cat ${MDATA} | while read BS ST AS
do
    echo "esearch -db nucleotide -query ${ASSEMBLY} | efetch -format gbwithparts > ${ASSEMBLY}.gbk"
done | parallel -j 3 --bar {}
```

## Example 2:

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

