# Introduction

SDMX-CSV Data Message is an SDMX data exchange format based on the [RFC 4180](https://tools.ietf.org/html/rfc4180). CSV is a widely used standardised and simple format to exchange data supported by many tools.

SDMX-CSV integrates with other specifications, i.e.: 
- The SDMX API RESTful specification (e.g. content negotiation with mime-type to get SDMX-CSV representations, specific formats for responses, language selection through HTTP content negotiation)
- The [RFC 4180](https://tools.ietf.org/html/rfc4180) specification

##	RFC 4180: A common format for CSV files
In order to benefit from best practices, SDMX-CSV is based on the rules defined in the [RFC 4180](https://tools.ietf.org/html/rfc4180), which defines a common format and MIME Type for CSV files. It is advised to read the (very short) RFC for a full list of requirements but, in a nutshell, the RFC defines rules such as:
- How the CSV file should be structured (the RFC specifies that all records must have an identical structure (determined column number), like when using an SDMX "flat" representation for data);
- When double-quotes should be used and how to escape them when needed;
- How spaces should be handled: Spaces are considered part of a field and should not be ignored;
- Which mime type should be used;
- What is the default character set, etc.

The SDMX-CSV format is flexible enough in its representation to support the needs of different target audiences:
- It is designed and optimised for the purpose of general public data dissemination of statistical data, and for usage in common statistical software.
- It allows using the messages to create pivot tables in spreadsheets applications.

# Design principles for SDMX-CSV 2.0 Data Messages (aligned with SDMX 3.0.0)

- In order to ensure the identifiability of the data contained in the message, the header row containing the column headers is mandatory and its content is well-defined. 
- After the mandatory header row, each row contains the information related to one specific observation or to one or more attributes attached to partial keys. 
- In [RFC 4180](https://tools.ietf.org/html/rfc4180), csv stands for "comma-separated values". However, while SDMX-CSV uses indeed the "comma" (%x2C) as the default field separator, it adopts the wider interpretation of csv as "character-separated values". It is recommended for implementers to provide SDMX-CSV messages according to the locale of the user (e.g. as indicated in the http Accept-Language header). It means that e.g. the semi-colon ‘;’ (as used typically in specific regions or countries) is acceptable as separator. See also the fourth example below. Note that the separator used in a message can be determined by retrieving the character that follows the header field of the first column which extended by a squared bracket term (see below). Implementers need here to be careful since the squared bracket term can itself contain escaped (doubled) bracket characters.

## Columns

- The first column is always used for the dataflow, data structure definition or data provision agreement identification and other optional pieces of identification information.
- Each Data Structure Definition (DSD) component (dimensions, attributes (including those defined through a referenced Metadata Structure Definition (MSD)), measures) included in the message is represented in one column, in any order.
- Only all those dimensions are to be included, that are required to uniquely identify the included attributes and/or measures.
- Attributes can but do not need to be included even if they have a mandatory status.
- Measures can but do not have to be included.
- All values presented for an attribute or measure can be left empty. This allows SDMX RESTful web service implementations supporting a streaming mechanism.
- Implementers have the possibility to add any other custom columns as required, e.g. serieskey, prepared, etc.

## Column headers (first row)

- The header field of the first column always starts with one of the terms `DATAFLOW`, `DATASTRUCTURE` or `DATAPROVISION`, depending on type of artefact for which the data contained in the message are defined: dataflow, data structure definition or data provision agreement.
- Optionally, the header of the first column can be extended with the following pieces of information encapsulated in squared brackets "[]":
  - **Variant A**: a sub-field delimiter, e.g. `DATAFLOW[;]`.
  - **Variant B** (only if all data contained in the message belong to the same artefact): a sub-field delimiter and the artefact identification information in the form *AGENCY:ARTEFACT_ID(VERSION)*(1), e.g. `DATAFLOW[;ESTAT:NA_MAIN(1.6.0)]`.
  - **Variant C1** (only if all data contained in the message belong to the same artefact and if option `labels=both` (see *[here](#optional-parameters)*)): a sub-field delimiter, the artefact identification information in the form *AGENCY:ARTEFACT_ID(VERSION)*(1), a separation term ": " and the artefact's localised name, e.g. `DATAFLOW[;ESTAT:NA_MAIN(1.6.0): National Accounts Main Aggregates]`.
  - **Variant C2** (only if all data contained in the message belong to the same artefact and if option `labels=name` (see *[here](#optional-parameters)*)): a sub-field delimiter, the artefact identification information in the form *AGENCY:ARTEFACT_ID(VERSION)*(1), a colon character ":", the IDs of the components included in the message (in any order) separated by a dot ".", a separation term ": " and the artefact's localised name, e.g. `DATAFLOW[;ESTAT:NA_MAIN(1.6.0):DIM1.DIM2.DIM3.MEAS1.MEAS3.ATTR2.ATTR4: National Accounts Main Aggregates]`.
  - For the variants C1 and C2, if the artefact name contains squared brackets then they must be doubled, e.g. `DATAFLOW[;AGENCY:ARTEFACT_1(1.0.0): Artefact [[1]]]`.
- The other columns for components contain either:
  - Default: The ID of the component reported in that column, e.g. `DIM1`.
  - If option `labels=both` (see *[here](#optional-parameters)*): The ID and the localised name of the component reported in that column separated by the term ": ", e.g. `DIM1: Dimension 1`.
  - If option `labels=name` (see *[here](#optional-parameters)*) and variant C2: The localised name of the component reported in that column, e.g. `Dimension 1`. In this case only, the column order must respect the order of components listed in the header of the first column, and additional columns not representing DSD components must be listed at the end.
- Any other custom column contains a custom but unique term, e.g. `SERIESKEY`.

## Column content (all rows after header)

- The first column contains:
  - Default: The artefact identification information in the form *AGENCY:ARTEFACT_ID(VERSION)*(1), e.g. `ESTAT:NA_MAIN(1.6.0)`.
  - Default if option `labels=both` (see *[here](#optional-parameters)*): The artefact identification information in the form *AGENCY:ARTEFACT_ID(VERSION)* and its localised name separated by the term ": ", e.g.  `ESTAT:NA_MAIN(1.6.0): National Accounts Main Aggregates`.
  - If variant B or if option `labels=both` (see *[here](#optional-parameters)*) with variant C1: The fields are being left empty.
  - If option `labels=name` (see *[here](#optional-parameters)*) with variant C2: The IDs of the component values included in the message (respecting the order of components listed in the header of the first column) separated by a dot ".", leaving non-enumerated component values empty, e.g. `A.B.C...AA..`.
- The other columns for components contain either:
  - Default: The ID(s) (if coded) or value(s) (if non-coded) for the component values reported in that column for the corresponding observation, e.g. `A`.
  - If option `labels=both` (see *[here](#optional-parameters)*): The ID(s) and the localised name separated by the term ": " (if coded) or the value(s) (if non-coded) for the component values reported in that column for the corresponding observation, e.g. `A: A value name`.
  - If option `labels=name` (see *[here](#optional-parameters)*) with variant C2: The localised name separated by the term ": " (if coded) or the value(s) (if non-coded) for the component values reported in that column for the corresponding observation, e.g. `A value name`. In this case only, the column order must respect the order of components listed in the header of the first column, and additional columns not representing DSD components must be listed at the end.
  - For rows containing the information related to one specific observation, the related values for attributes attached to partial keys need to be replicated.
  - For rows containing the information related to one or more attributes attached to partial keys, in addition to these attributes only the components that are part of the partial key need to be filled, all others can be left empty. 
- The other custom columns contain any potentially localised custom content.

## Localisation 

- HTTP content negotiation, see [RFC 2616 - HTTP 1.1 Header Field Definitions](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html)
  - Always use this mime-type in the Accept header: `application/vnd.sdmx.data+csv; version=1.0.0`.
  - The client can indicate preferred languages through the Accept-Language header, e.g. `fr, en-gb;q=0.8, en;q=0.7`.
- Always localise all artefact names according to the preferred language. The first best language match according to the user’s preferred language choices in the http Accept-Language header (or if that is not available than according to the system's default language order) is to be used for each localisable name element. The message does however not indicate the returned language per localisable name element. In case that there is no such language match for a particular localisable name element, it is optional to return the element in a system-default language or alternatively to not return the element.
**It is recommended to indicate all languages used anywhere in the message for localised name elements through http Content-Language response header (languages of the intended audience).**  
Note: For multi-language values, all language versions are provided independently from the preferred language (see below).   

## Multi-valued components

- Some components (measures or attributes) allow for multiple values. Those multiple values are separated by a special sub-field separation character, e.g. `;`.
- This sub-field separation character has to be defined as first character in the squared bracket term of the header field of the first column, e.g. `DATAFLOW[;]`.
- Such components are indicated by having its ID followed by empty squared brackets "[]", e.g. `ATTR4[]`.
- For coded multi-valued components, if option `labels=both` (see *[here](#optional-parameters)*) then each individual value is to be prefixed with its ID and the term ": ", e.g. `A: Value A;B: Value B`.

## Non-coded multi-lingual components

- Some non-coded components (measures or attributes) allow for multi-lingual values. Those values are separated by a special sub-field separation character, e.g. `;`.
- This sub-field separation character has to be defined as first character in the squared bracket term of the header field of the first column, e.g. `DATAFLOW[;]`.
- Such components are indicated by having its ID followed by the list of possible 2-letter ISO language codes separated by the sub-field separator and encapsulated squared brackets "[]", e.g. `ATTR2[en;fr]`.
- Each individual language value is to be prefixed with its 2-letter ISO language code and a colon character ":", e.g. `en:Value;fr:Valeur`. Thus, in distinction to the ID prefix for coded values when using the HTTP accept header `labels=both` (see *[here](#optional-parameters)*), the language prefix `xx:` doesn't have an extra space character.
- Note that multi-valued components are always non-coded and therefore do not interfere with value IDs.

## Non-coded multi-lingual multi-valued components

- Some non-coded components (measures or attributes) allow for multiple multi-lingual values. All individual values are separated by a special sub-field separation character, e.g. `;`.
- This sub-field separation character has to be defined as first character in the squared bracket term of the header field of the first column, e.g. `DATAFLOW[;]`.
- Such components are indicated by having its ID followed by the list of possible language codes separated by the sub-field separator and encapsulated squared brackets "[]", e.g. `ATTR2[en;fr;de]`.
- Each individual language value is to prefixed with its 2-letter ISO language code and a colon character ":", e.g. `en:Value1`.
- Each multi-lingual value set is to be encapsulated in double quotes, e.g. `"en:Value1;fr:Valeur1";"en:Value2;de:Wert2"`. However, mote that fields with quotes must themselves be encapsulated in double quotes and that the inner quotes need to be doubled, thus the fully complete example is `"""en:Value1;fr:Valeur1"";""en:Value2;de:Wert2"""`.

## Non-coded XHTML-valued components

- Some non-coded components (measures or attributes) allow for XHTML values.
- Each XHTML value is to be encapsulated in double quotes, e.g. `"<p>This is some ""metadata html""</p>"`. Remember that the inner quotes need to be doubled.
- The CSV format allows fields to contain line breaks if those fields are enclosed in double quotes. Thus XHTML values can also contain line breaks.

# Optional parameters

Optional parameters can be added to the HTTP Accept header. They need to be separated by the character combination `"; "`.
- labels (id|name|both; default=id): This parameter applies to all Nameable SDMX Artefacts contained in the header and the body of the message: 
  - If the parameter value is `id` then only the id of the Artefacts is displayed.
  - If the parameter value is `name` then the id and the name of the Artefacts are displayed, but in separate places (see *[here](#columns)*).
  - If the parameter value is `both` then the concatenated id and localised name of the Artefacts (see the section on [localised names](#localised-names) on how the message deals with languages) separated by `": "` are displayed. Note that the character combination `": "` could also be part of the Artefact name and could therefore occur several times within the concatenated string.
- timeFormat (original|normalized; default=original):
  - If the parameter value is `original` then the *TIME-PERIOD* values are displayed in the SDMX *TIME_PERIOD* format as originally recorded.
  - If the parameter value is `normalized` then the *TIME_PERIOD* values are converted to the most granular [ISO 8601](https://www.iso.org/iso-8601-date-and-time-format.html) representation taking into account the highest frequency of the data in the message and the moment in time when the lower-frequency values were collected (which, e.g. at the ECB, is typically either at the beginning, middle or end of the reporting period). This eases comparisons and business analysis of multi-frequency values, e.g. in pivot tables. As an example, if annual and daily data are available in the message and the annual data were collected at the end of the reporting period, the formatted value for the annual period 2014 becomes 2014-12-31.

Support of above non-default parameters is not required by implementers.

# Examples

Note: All examples assume the minimal HTTP Accept header: `application/vnd.sdmx.data+csv; version=1.0.0`

#### 1) Ordinary case

	DATAFLOW,DIM_1,DIM_2,DIM_3,OBS_VALUE,ATTR_2,ATTR_3,ATTR_1,SERIESKEY
	ESTAT:NA_MAIN(1.6.0),A,B,2014-01,12.4,Y,"Normal, special and other values",N,A.B
	ESTAT:NA_MAIN(1.6.0),A,B,2014-02,10.8,Y,"Normal, special and other values",Y,A.B

Note: The following default parameter settings are automatically applied:
- labels=id
- timeFormat=original
- *SERIESKEY* is a custom column.

#### 2) Components in any order and missing component, Variant A

	DATAFLOW[;],SERIESKEY,OBS_VALUE1,OBS_VALUE2,ATTR_3,ATTR_1,DIM_2,DIM_1,DIM_3
	ESTAT:NA_MAIN(1.6.0),A.B,12.4,12.5,"Normal, special and other values",N,B,A,2014-01
	ESTAT:NA_MAIN(1.6.0),A.B,10.8,10.9,"Normal, special and other values",Y,B,A,2014-02

#### 3) Components in any order and missing component, Variant B

	DATAFLOW[;ESTAT:NA_MAIN(1.6.0)],SERIESKEY,OBS_VALUE1,OBS_VALUE2,ATTR_3,ATTR_1,DIM_2,DIM_1,DIM_3
	,A.B,12.4,12.5,"Normal, special and other values",N,B,A,2014-01
	,A.B,10.8,10.9,"Normal, special and other values",Y,B,A,2014-02

#### 4) Localisation: HTTP Accept header: `application/vnd.sdmx.data+csv; version=1.0.0; labels=both`, HTTP Accept-Language header: `fr-FR, en;q=0.7`, Variant C1

	DATAFLOW[|ESTAT:NA_MAIN(1.6.0): Principaux agrégats des comptes nationaux];DIM_1: Dimension 1;DIM_2: Dimension 2;DIM_3: Dimension 3;OBS_VALUE: Observation value;ATTR_2: Attribut 2;ATTR_3: Attribut 3;ATTR_1: Attribut 1;SERIESKEY
	;A: Value A;B: Value B;2014-01: 2014-01;12,4;Y: Oui;Normal, special and other values;N: Non;A.B
	;A: Value A;B: Value B;2014-01: 2014-02;10,8;Y: Oui;Normal, special and other values;Y: Oui;A.B

Note that in this example the client prefers French (fr) language with the France (FR) locale, but will also accept any type of English. Therefore, in the message the French language with the France locale is applied, transforming also the field separator from comma (,) to semicolon (;), and the decimal separator from dot (.) to comma (,).

#### 5) HTTP Accept header: `application/vnd.sdmx.data+csv; version=1.0.0; labels=both; timeFormat=normalized`, Variant C1

	DATAFLOW[;ESTAT:NA_MAIN(1.6.0): National Accounts Main Aggregates],DIM_1: Dimension 1,DIM_2: Dimension 2,DIM_3: Dimension 3,OBS_VALUE: Observation value,ATTR_2: Attribute 2,ATTR_3: Attribute 3,ATTR_1: Attribute 1,SERIESKEY
	,A: Value A,B: Value B,2014-01-01,12.4,Y: Yes,"Normal, special and other values",N: No,A.B
	,A: Value A,B: Value B,2014-02-01,10.8,Y: Yes,"Normal, special and other values",Y: Yes,A.B

#### 6) HTTP Accept header: `application/vnd.sdmx.data+csv; version=1.0.0; labels=name` = Variant C2

	DATAFLOW[;ESTAT:NA_MAIN(1.6.0):DIM_2.DIM_1.DIM_3.OBS_VALUE.ATTR_2.ATTR_3.ATTR_1: National Accounts Main Aggregates],Dimension 1,Dimension 2,Dimension 3,Observation value,Attribute 2,Attribute 3,Attribute 1,SERIESKEY
	B.A.2014-01..Y..N,Value B,Value A,2014-01,12.4,Yes,"Normal, special and other values",No,A.B
	B.A.2014-02..Y..Y,Value B,Value A,2014-01,10.8,Yes,"Normal, special and other values",Yes,A.B

#### 7) Multi-valued components, Variant B

	DATAFLOW[;ESTAT:NA_MAIN(1.6.0)],DIM_1,DIM_2,DIM_3,OBS_VALUE,ATTR_1[],ATTR_2[],ATTR_3[]
	,A,B,2014-01,12.4,Value X;Value Y,"M, N & O;P & Q",A;B;C
	,A,B,2014-02,10.8,Value X;Value Y,"M, N & O;P & Q",A;C

#### 8) Non-coded multi-lingual components, varying dataflows, Variant A

	DATAFLOW[;],DIM_1,DIM_2,DIM_3,OBS_VALUE,ATTR_1[en;fr]
	ESTAT:NA_MAIN(1.6.0),A,B,2014-01,12.4,en:Any Value;fr:N'importe quelle Valeur
	ESTAT:NA_MAIN(1.7.0),A,B,2014-02,10.8,"en:Value ""X"";fr:Valeur ""X"""

#### 9) Non-coded multi-lingual multi-valued components, varying dataflows, Variant A

	DATAFLOW[;],DIM_1,DIM_2,DIM_3,OBS_VALUE,ATTR_1[en;fr;de]
	ESTAT:NA_MAIN(1.6.0),A,B,2014-01,12.4,"""en:Value1;fr:Valeur1"";""en:Value2;de:Wert2"""
	ESTAT:NA_MAIN(1.7.0),A,B,2014-02,10.8,"""en:Value1;fr:Valeur1"";""en:Value2;de:Wert2"""

#### 10) Data for a non-versioned(1) data structure definition, Variant B

	DATASTRUCTURE[;AGENCY:DF_ID],DIM_1,DIM_2,DIM_3,OBS_VALUE,ATTR_1,SERIESKEY
	,A,B,2014-01,12.4,N,A.B
	,A,B,2014-02,10.8,Y,A.B

#### 11) Attributes attached to partial keys for a data provision agreement, Variant B

	DATAPROVISION[;AGENCY:DPA_ID(1.0.0)],DIM_2,DIM_3,ATTR_1
	,B,2014-01,N
	,B,2014-02,Y

#### 12) Mixing rows for attributes attached to partial keys with rows for observations 

	DATAFLOW,DIM_1,DIM_2,DIM_3,MEAS_1,ATTR_1,ATTR_2
	AGENCY:DF_ID(1.0.0),A,B,2014-01,12.4,N,
	AGENCY:DF_ID(1.0.0),,B,,,,Y

#### 13) Non-coded XHTML-formatted values with line-breaks 

	DATAFLOW,DIM_1,DIM_2,DIM_3,OBS_VALUE,ATTR_1
	ESTAT:NA_MAIN(1.6.0),A,B,2014-01,12.4,"<p>This is some ""xhtml"" with a line
	break</p>"
	ESTAT:NA_MAIN(1.6.0),A,B,2014-02,10.8,"<p>This is some other ""xhtml""</p>"

------------------------

**(1)** Note that since SDMX 3.0.0 the syntax *AGENCY:ARTEFACT_ID(VERSION)* allows omitting the version for non-versioned artefacts. In this case using *AGENCY:ARTEFACT_ID* is sufficient, e.g. `AGENCY:DF_ID`
