# Introduction
SDMX-CSV Data Message is a SDMX data exchange format based on the RFC4180. CSV is a widely used standardised simple format used to exchange data and is supported by many tools.

SDMX-CSV integrates with other specifications, i.e.: 
- The SDMX RESTful specification (e.g. content negotiation with mime-type to get SDMX-CSV representations, specific formats for responses, language selection through HTTP content negotiation)
- The RFC 4180 specification (determined column number, comma separated)

SDMX-CSV is flexible enough in its representation to support the needs of different target audiences:
- A representation optimised for public data dissemination and similar, and for usage in common statistical software
- A representation optimised for creating pivot tables in spreadsheets applications

All representations offered by SDMX-CSV are based on the RFC 4180, which defines a common format and MIME Type for CSV files. 

##	RFC 4180: A common format for CSV files
In order to benefit from best practices, SDMX-CSV is based on the rules defined in the [RFC 4180](https://tools.ietf.org/html/rfc4180). It is advised to read the (very short) RFC for a full list of requirements but, in a nutshell, the RFC defines rules such as:
- How the CSV file should be structured (the RFC specifies that all records must have an identical structure, like when using an SDMX "flat" representation for data);
- When double-quotes should be used and how to escape them when needed;
- How spaces should be handled;
- Which separator should be used (comma);
- Which mime type should be used;
- How to handle / request column headers;
- What is the default character set, etc.

#	Design principles

- There is no SDMX-specific header. The SDMX-CSV format is mainly intended and more suitable for purposes of public dissemination.
- Columns: First dimensions (always one column per dimension), then the measure (one column) and then the attributes (always one column per attribute all following the order defined in the DSD).
-	Possibility (see options below) to add:
  - a column at the end with the reference to the dataflow
  - a column at the end with the series key
  - any other custom columns as required.
-	Whenever appropriate (e.g. in case the updatedAfter or includeHistory parameters of the SDMX RESTful API were used by the client), additional columns such as the ones for the action flags (Delete, Replace, etc.) or the validFrom and validTo “technical” attributes can be displayed.
- It is also worth noting that, in case the SDMX RESTful 2.1 web service implementation supports a streaming mechanism, columns for all attributes defined in the DSD are always present in the output, regardless of whether these attributes are used (unless of course the client makes use of the SDMX 2.1 RESTful detail parameter to disable to display of attributes).
- Comma Separator for columns is used by default, but it is recommended for implementers to provide the response according to the locale of the client (which means that in some cases the semi-colon ‘;’ is acceptable as separator).
- HTTP content negotiation (HTTP Accept header) with mime-type:
    application/vnd.sdmx.data+csv;version=1.0.0,header=true|false,display=id|name|both,dataflow=false|true,serieskey=false|true,periodFormatting=false|true

#	Optional parameters

- Header (default=true): If this is true the first row is the header containing the IDs (and/or labels) of the components as they are defined in the DSD. If the parameter is false then the order defined in the DSD should be followed (display option applies here). 
- Display(default=id): 
  - If the option is ID, displays only the id of the dimension or attribute (for coded components).
  - If it is NAME it displays the label according to the language specified in the http negotiation.
  - If it is BOTH then display concatenated id and label (according to the language specified in the http negotiation) matching this regular expression ID(:| |-)( )[0..1]LABEL
- Dataflow (default=false): If this is true a column is added at the end of the file showing in all rows the full reference to the dataflow (Agency + ID + Version). The corresponding header row contains the term DATAFLOW as ID.
- Serieskey (default=false): If this is true a column is added at the end of the file (after DATAFLOW if this was added) containing the serieskey prefixed with the dataflow id. The corresponding header row contains the term SERIESKEY.
- periodFormatting (default=false) - In order to ease comparisons, the TIME_PERIOD values are converted to the most granular ISO8601 representation possible (depending on the frequency) and take into account the moment in time when the values were collected (which, e.g. at the ECB, is typically either at the beginning, middle or end of the reporting period). As an example, if annual and daily data are available in the CSV output and the annual data were collected at the end of the period, the formatted value for 2014 becomes 2014-12-31.

Support of above options is not required by implementers.

# Examples

#### application/vnd.sdmx.data+csv;version=1.0.0,header=true,display=id,dataflow=true,serieskey=false,periodFormatting=false

    DIM_1,DIM_2,DIM_3,VALUE,ATTR_1,ATTR_2,ATTR_3,EMBARGO_DATE,DATAFLOW
    A,B,2014-01,12.4,N,Y,"Normal, special and other values",2016-12-14T15:00:00,ESTAT+NA_MAIN+1.6
    A,B,2014-02,10.8,Y,Y,"Normal, special and other values",2016-12-15T15:00:00,ESTAT+NA_MAIN+1.6

#### application/vnd.sdmx.data+csv;version=1.0.0,header=true,display=label,dataflow=false,serieskey=true,periodFormatting=false
[French locale]

    Dimension 1;Dimension 2;Dimension 3;VALUE;Attribute 1;Attribute 2;Attribute 3;Embargo date;SERIESKEY
    Value A;Value B;2014-01;12,4;No;Yes;Normal, special and other values;2016-12-14T15:00:00;NA_MAIN.A.B.C
    Value A;Value B;2014-02;10,8;Yes;Yes;Normal, special and other values;2016-12-14T15:00:00;NA_MAIN.A.B.D

#### application/vnd.sdmx.data+csv;version=1.0.0,header=true,display=both,dataflow=false,serieskey=false,periodFormatting=true
[for pivot table]

    DIM_1: Dimension 1,DIM_2: Dimension 2,DIM_3: Dimension 3,VALUE;ATTR_1: Attribute 1,ATTR_2: Attribute 2,ATTR_3: Attribute 3,EMBARGO_DATE: Embargo date
    A: Value A,B: Value B,2014-01-01,12.4,N: No,Y: Yes,"Normal, special and other values",2016-12-14T15:00:00
    A: Value A,B: Value B,2014-02-01,10.8,Y: Yes,Y: Yes,"Normal, special and other values",2016-12-14T15:00:00

#### application/vnd.sdmx.data+csv;version=1.0.0,header=false

    A,B,2014-01,12.4,N,Y,"Normal, special and other values",2016-12-14T15:00:00
    A,B,2014-02,10.8,Y,Y,"Normal, special and other values",2016-12-15T15:00:00
