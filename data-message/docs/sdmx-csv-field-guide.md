# Introduction
SDMX-CSV Data Message is a SDMX data exchange format based on the RFC4180. CSV is a widely used standardised simple format used to exchange data and is supported by many tools.

SDMX-CSV integrates with other specifications, i.e.: 
- The SDMX RESTful specification (e.g. content negotiation with mime-type to get SDMX-CSV representations, specific formats for responses, language selection)
- The RFC 4180 specification (determined column number, comma separated)

#	Design principles
SDMX-CSV is flexible enough in its representation, to support the needs of different target audiences. Multiple representations are supported:
- A representation optimised for data exchanges and similar, in principles, to the “flat” representation of the SDMX-ML 2.1 Structure Specific Data format. 
- A representation optimised for creating pivot tables in spreadsheets applications such as Microsoft Excel. 

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

##	CSV for data exchanges
The CSV flavour for data exchanges is very close to the principles adopted for the flat representation of the SDMX-ML 2.1 Structure Specific Data format:
-	Concept and code IDs are used, instead of the (user-friendlier but more verbose) concept and code names;
-	The TIME_PERIOD is returned as-is.

##	CSV for pivot tables
The CSV flavour for pivot tables is considered user-friendly but is also more verbose:
-	Concept names, as well as both code IDs and names are used;
-	In order to ease comparisons, the TIME_PERIOD values are converted to the most granular ISO8601 representation possible (depending on the frequency) and take into account the moment in time when the values were collected (which, at the ECB is typically either at the beginning, middle or end of the reporting period). To give an example, if annual and daily data are available in the CSV output and the annual data were collected at the end of the period, the formatted value for 2014 becomes 2014-12-31.

##	Additional commonalities between the 2 formats
In addition to the rules defined in the RFC 4180 and to the dimensions and attributes defined in the DSDs, both SDMX-CSV flavours offer:
-	The possibility to add a column with the series key (including the dataflow id as prefix);
-	Whenever appropriate (e.g. in case the updatedAfter or includeHistory parameters of the SDMX RESTful API were used by the client), additional columns such as the ones for the action flags (Delete, Replace, etc.) or the validFrom and validTo “technical” attributes can be displayed;
It is also worth noting that, in case the SDMX RESTful 2.1 web service implementation supports a streaming mechanism, columns for all attributes defined in the DSD are always present in the output, regardless of whether these attributes are used (unless of course the client makes use of the SDMX 2.1 RESTful detail parameter to disable to display of attributes).
