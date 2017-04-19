# Introduction

SDMX-CSV Data Message is an SDMX data exchange format based on the [RFC 4180](https://tools.ietf.org/html/rfc4180). CSV is a widely used standardised and simple format to exchange data supported by many tools.

SDMX-CSV integrates with other specifications, i.e.: 
- The SDMX API RESTful specification (e.g. content negotiation with mime-type to get SDMX-CSV representations, specific formats for responses, language selection through HTTP content negotiation)
- The [RFC 4180](https://tools.ietf.org/html/rfc4180) specification (determined column number, comma separated)

SDMX-CSV is flexible enough in its representation to support the needs of different target audiences:
- A representation optimised for public data dissemination and similar, and for usage in common statistical software
- A representation optimised for creating pivot tables in spreadsheets applications

##	RFC 4180: A common format for CSV files
In order to benefit from best practices, SDMX-CSV is based on the rules defined in the [RFC 4180](https://tools.ietf.org/html/rfc4180), which defines a common format and MIME Type for CSV files. It is advised to read the (very short) RFC for a full list of requirements but, in a nutshell, the RFC defines rules such as:
- How the CSV file should be structured (the RFC specifies that all records must have an identical structure, like when using an SDMX "flat" representation for data);
- When double-quotes should be used and how to escape them when needed;
- How spaces should be handled;
- Which separator should be used (comma);
- Which mime type should be used;
- How to handle / request column headers;
- What is the default character set, etc.

#	Design principles

- There is no SDMX-specific header. The SDMX-CSV format is designed for the purpose of general public dissemination of statistical data.
- After an optional header row, each row contains the information related to one specific observation. 
- Columns: There must be one column for the dataflow, one column per dimension, one column for the measure and one column per attribute. All dimensions defined in the related Data Structure Definition (DSD) are to be included. In case the SDMX RESTful 2.1 web service implementation supports a streaming mechanism, columns for all attributes defined in the DSD are present in the output, regardless of whether these attributes are used. Implementers have the possibility to add any other custom columns as required, e.g. serieskey, sender, prepared, etc.
- Column headers (first row, if present - see option below): 
  - For the dataflow column, always is the term *DATAFLOW*.
  - For a dimension column, is the dimension's ID, name or both (see option below).
  - For the measure column, always is the term *OBS_VALUE*.
  - For an attribute column, is the attribute's ID, name or both (see option below).
  - For any custom column, is any custom but unique term.
- Column content (all rows after header):
  - For the dataflow column, is the reference to the *dataflow* in the following form: AGENCY:DATAFLOW_ID(VERSION), its name or both (see option below).
  - For a dimension column, is the ID, name or both (see option below) of the observation's code in the corresponding dimension.
  - For the measure column, is the value of the observation.
  - For a coded attribute column, is the ID, name or both (see option below) of the code in the corresponding attribute. For attributes defined at series, group or dataset level, the codes are replicated for all observations concerned.
  - For an uncoded attribute column, is the value of the corresponding attribute. For attributes defined at series, group or dataset level, the values are replicated for all observations concerned.
  - For any custom column, contains any custom content.
- Comma Separator for columns is used by default, but it is recommended for implementers to provide the response according to the locale of the client (which means that in some cases the semi-colon ‘;’ is acceptable as separator).
- HTTP content negotiation (HTTP Accept header) with mime-type (see [RFC 7231](https://tools.ietf.org/html/rfc7231#section-5.3.2)):

      application/vnd.sdmx.data+csv; version=1.0.0
    
#	Optional parameters

Optional parameters can be added to the HTTP Accept header. They need to be separated by `"; "`.
- header (present|absent; default=present): This parameter is defined in the [RFC 4180](https://tools.ietf.org/html/rfc4180) standard.
  - If the parameter value is `present` then the first row is the header as described above. 
  - If the parameter value is `absent` then in addition to the columns for the dataflow, for all dimensions in the order defined in the DSD and for the measure also all attribute columns in the order defined in the DSD need to be included in the message, but custom columns cannot be added.
- labels (id|name|both; default=id): This parameter applies to all Nameable SDMX Artefacts contained in the header and the body of the message: 
  - If the parameter value is `id` then only the id of the Artefacts is displayed.
  - If the parameter value is `name` then only the name of the Artefacts (according to the language specified in the http negotiation) is displayed.
  - If the parameter value is `both` then the concatenated id and name of the Artefacts (according to the language specified in the http negotiation) separated by `": "` are displayed. Note that the character combination `": "` could also be part of the Artefact name and could therefore occur several times within the concatenated string.
- timeFormat (original|normalized; default=original):
  - If the parameter value is `original` then the *TIME-PERIOD* values are displayed in the SDMX *TIME_PERIOD* format as originally recorded.
  - If the parameter value is `normalized` then the *TIME_PERIOD* values are converted to the most granular [ISO 8601](https://www.iso.org/iso-8601-date-and-time-format.html) representation taking into account the highest frequency of the data in the message and the moment in time when the lower-frequency values were collected (which, e.g. at the ECB, is typically either at the beginning, middle or end of the reporting period). This eases comparisons and business analysis of multi-frequency values, e.g. in pivot tables. As an example, if annual and daily data are available in the message and the annual data were collected at the end of the reporting period, the formatted value for the annual period 2014 becomes 2014-12-31.

Support of above parameters is not required by implementers.

# Examples

#### 1) HTTP Accept header: application/vnd.sdmx.data+csv; version=1.0.0

    DATAFLOW,DIM_1,DIM_2,DIM_3,OBS_VALUE,ATTR_2,ATTR_3,ATTR_1,SERIESKEY
    ESTAT:NA_MAIN(1.6),A,B,2014-01,12.4,Y,"Normal, special and other values",N,A.B
    ESTAT:NA_MAIN(1.6),A,B,2014-02,10.8,Y,"Normal, special and other values",Y,A.B

The following default parameter settings are automatically applied:
- header=present
- labels=id
- timeFormat=original
- *SERIESKEY* is a custom column.

#### 2) HTTP Accept header: application/vnd.sdmx.data+csv; version=1.0.0; header=absent

    ESTAT:NA_MAIN(1.6),A,B,2014-01,12.4,N,Y,"Normal, special and other values"
    ESTAT:NA_MAIN(1.6),A,B,2014-02,10.8,Y,Y,"Normal, special and other values"

The following default parameter settings are automatically applied:
- labels=id
- timeFormat=original
- Custom columns are suppressed.
- Dimension and attribute columns are ordered in order defined in related Data Structure Definition.

#### 3) HTTP Accept header: application/vnd.sdmx.data+csv; version=1.0.0; labels=name
####    HTTP Accept-Language header: fr-FR, en;q=0.7

    DATAFLOW;Dimension 1;Dimension 2;Dimension 3;OBS_VALUE;Attribut 2;Attribut 3;Attribut 1;SERIESKEY
    Principaux agrégats des comptes nationaux;Valeur A;Valeur B;2014-01;12,4;Oui;Normal, special and other values;Non;A.B
    Principaux agrégats des comptes nationaux;Valeur A;Valeur B;2014-02;10,8;Oui;Normal, special and other values;Oui;A.B

The following default parameter settings are automatically applied:
- header=present
- timeFormat=original
- *SERIESKEY* is a custom column.

Note that in this example the client prefers French (fr) language with the France (FR) locale, but will also accept any type of English. See [RFC 2616 - HTTP 1.1 Header Field Definitions](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html). Therefore, in the message the French language with the France locale is realized, transforming also the field separator from comma (,) to semicolon (;), and the decimal separator from dot (.) to comma (,).

#### 4) HTTP Accept header: application/vnd.sdmx.data+csv; version=1.0.0; labels=both; timeFormat=normalized

    DATAFLOW,DIM_1: Dimension 1,DIM_2: Dimension 2,DIM_3: Dimension 3,OBS_VALUE,ATTR_2: Attribute 2,ATTR_3: Attribute 3,ATTR_1: Attribute 1,SERIESKEY
    ESTAT:NA_MAIN(1.6): National Accounts Main Aggregates,A: Value A,B: Value B,2014-01-01,12.4,Y: Yes,"Normal, special and other values",N: No,A.B
    ESTAT:NA_MAIN(1.6): National Accounts Main Aggregates,A: Value A,B: Value B,2014-02-01,10.8,Y: Yes,"Normal, special and other values",Y: Yes,A.B

The following default parameter settings are automatically applied:
- header=present
- *SERIESKEY* is a custom column.
