# Introduction
SDMX-CSV Data Message is a SDMX data exchange format based on the RFC4180. CSV is a widely used standardised simple format used to exchange data and is supported by many tools.

SDMX-CSV integrates with other specifications, i.e.: 
- The SDMX API RESTful specification (e.g. content negotiation with mime-type to get SDMX-CSV representations, specific formats for responses, language selection through HTTP content negotiation)
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

- There is no SDMX-specific header. The SDMX-CSV format is mainly intended and more suitable for purposes of general public dissemination.
- Columns: First all dimensions (always one column per dimension), then the measure (one column) and then one or more attributes (always one column per attribute) ideally all following the order defined in the DSD.
- Possibility (see options below) to add:
  - A column at the end with the reference to the dataflow.
  - A column at the end with the series key.
  - Any other custom columns as required, e.g. sender, prepared, etc.
- It is also worth noting that, in case the SDMX RESTful 2.1 web service implementation supports a streaming mechanism, columns for all attributes defined in the DSD are always present in the output, regardless of whether these attributes are used (unless of course the client makes use of the SDMX 2.1 RESTful detail parameter to disable the display of attributes).
- Comma Separator for columns is used by default, but it is recommended for implementers to provide the response according to the locale of the client (which means that in some cases the semi-colon ‘;’ is acceptable as separator).
- HTTP content negotiation (HTTP Accept header) with mime-type:
    application/vnd.sdmx.data+csv;version=1.0.0,header=present|absent,display=id|name|both,additionalColumns=none|all|{[+]dataflow, [+]serieskey, [+]custom},timePeriodFormat=sdmx|iso8601

#	Optional parameters

- header (present|absent; default=present): If the parameter value is "present" then the first row is the header containing the IDs (and/or labels) of the components as they are defined in the DSD. If the parameter value is "absent" then in addition to all dimensions and the measure columns also all attribute columns need to be included in the message in the order defined in the DSD, but custom columns cannot be added. The parameter values are defined in the RFC 4180 standard.
- display (id|name|both; default=id): This parameter applies to all Nameable SDMX Artefacts contained in the header and the body of the message: 
  - If the parameter value is "id" then only the id of the Artefacts is displayed.
  - If the parameter value is "name" then only the name of the Artefacts (according to the language specified in the http negotiation) is displayed.
  - If the parameter value is "both" then the concatenated id and name of the Artefacts (according to the language specified in the http negotiation) separated by ": " are displayed. Note that the character combination ": " could also be part of the Artefact name and could therefore occur several times within the concatenated string.
- timePeriodFormat (sdmx|iso8601; default=sdmx) - If the parameter value is "iso8601" then the TIME_PERIOD values are converted to the most granular ISO8601 representation possible (depending on the frequency) and take into account the moment in time when the values were collected (which, e.g. at the ECB, is typically either at the beginning, middle or end of the reporting period). This eases comparisons and business analysis such as in pivot tables. As an example, if annual and daily data are available in the message and the annual data were collected at the end of the reporting period, the formatted value for 2014 becomes 2014-12-31. If the parameter is "sdmx" then the TIME-PERIOD values are displayed in an appropriate SDMX TIME_PERIOD format.
- additionalColumns (none|all|{[+]dataflow, [+]serieskey, [+]custom}; default=none): While the parameter options "all" and "none" cannot be combined with any other option, all other options can be combined but need to be separated by "+", e.g. "additionalColumns=dataflow+serieskey". Note that any additionalColumns parameter option other than "none" is incompatible with the option "header=absent". If additional columns are included then the header row must also be included.
  - If the parameter value is "all" then all currently implemented additional columns are added to the message.
  - If the parameter value is "none" then only columns for dimensions, measure and attributes are included in the message.
  - If the parameter value contains "dataflow" then a column is added as last column showing in all rows the full reference to the dataflow (same as URN). The corresponding header row contains the term DATAFLOW as ID. 
  - If the parameter value contains "serieskey" then a column is added as last column (after DATAFLOW if this was added) containing the serieskey, a string compliant with the KeyType defined in the SDMX WADL, e.g. "D.USD.EUR.SP00.A". The corresponding header row contains the term SERIESKEY.
  - If the parameter value contains "custom" then any custom columns, e.g. sender, prepared, etc., can be added after the attribute columns showing in all rows the custom information. The corresponding header row must contain a term that makes the column unique and identifiable.

Support of above parameters is not required by implementers.

# Examples

#### application/vnd.sdmx.data+csv;version=1.0.0,additionalColumns=dataflow+serieskey+custom

    DIM_1,DIM_2,DIM_3,VALUE,ATTR_1,ATTR_2,ATTR_3,EMBARGO_DATE,DATAFLOW,SERIESKEY
    A,B,2014-01,12.4,N,Y,"Normal, special and other values",2016-12-14T15:00:00,ESTAT+NA_MAIN+1.6,A.B
    A,B,2014-02,10.8,Y,Y,"Normal, special and other values",2016-12-15T15:00:00,ESTAT+NA_MAIN+1.6,A.B

The following parameter settings are automatically applied:
- header=present
- display=id
- timePeriodFormat=sdmx

#### application/vnd.sdmx.data+csv;version=1.0.0,display=name,additionalColumns=serieskey+custom
[French locale]

    Dimension 1;Dimension 2;Dimension 3;VALUE;Attribute 1;Attribute 2;Attribute 3;Embargo date;SERIESKEY
    Value A;Value B;2014-01;12,4;No;Yes;Normal, special and other values;2016-12-14T15:00:00;A.B
    Value A;Value B;2014-02;10,8;Yes;Yes;Normal, special and other values;2016-12-14T15:00:00;A.B

The following parameter settings are automatically applied:
- header=present
- timePeriodFormat=sdmx

#### application/vnd.sdmx.data+csv;version=1.0.0,display=both,additionalColumns=none,timePeriodFormat=iso8601
[for pivot table]

    DIM_1: Dimension 1,DIM_2: Dimension 2,DIM_3: Dimension 3,VALUE;ATTR_1: Attribute 1,ATTR_2: Attribute 2,ATTR_3: Attribute 3
    A: Value A,B: Value B,2014-01-01,12.4,N: No,Y: Yes,"Normal, special and other values"
    A: Value A,B: Value B,2014-02-01,10.8,Y: Yes,Y: Yes,"Normal, special and other values"

The following parameter settings are automatically applied:
- header=present

#### application/vnd.sdmx.data+csv;version=1.0.0,header=absent

    A,B,2014-01,12.4,N,Y,"Normal, special and other values"
    A,B,2014-02,10.8,Y,Y,"Normal, special and other values"

The following parameter settings are automatically applied:
- display=id
- additionalColumns=none
- timePeriodFormat=sdmx
