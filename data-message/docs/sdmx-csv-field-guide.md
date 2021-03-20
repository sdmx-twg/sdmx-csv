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
- After the mandatory header row, each row contains the information related to one specific observation or to one or more attributes attached to partial keys. For `Delete` actions a row can also concern several observations if dimensions are wildcarded.
- In [RFC 4180](https://tools.ietf.org/html/rfc4180), csv stands for "comma-separated values". However, while SDMX-CSV uses indeed the "comma" (%x2C) as the default field separator, it adopts the wider interpretation of csv as "character-separated values". It is recommended for implementers to provide SDMX-CSV messages according to the locale of the user (e.g. as indicated in the http Accept-Language header). It means that e.g. the semi-colon ‘;’ (as used typically in specific regions or countries) is acceptable as separator. See also the fourth example below. Note that the separator used in a message can be determined by retrieving the character that follows the header field of the first column which extended by a squared bracket term (see below). Implementers need here to be careful since the squared bracket term can itself contain escaped (doubled) bracket characters.

## Columns

- The first column is always used for the structure type: dataflow, data structure definition or data provision agreement.
- The second column is always used for the structure's identification.
- The third column is always used for the action to be performed.
- The next up to two columns are used for the series and/or observation key.
- Each Data Structure Definition (DSD) component (dimensions, attributes (including those defined through a referenced Metadata Structure Definition (MSD)), measures) included in the message is represented in one column. SDMX web services should return the columns in the order of components as defined in the Data Structure Definition. However, any order of these columns is valid for data uploads to SDMX-consuming systems.
- Only all those dimension columns have to be present, that are required to uniquely identify the concerned attributes and/or measures.
- Attributes can but do not need to be included even if they have a mandatory status.
- Measures can but do not have to be included.
- When an SDMX RESTful web service implements streaming, then it might not know, while generating the csv header row, which measures and attributes actually have values. Therefore, it can happen that all values presented for an attribute or measure are left empty.
- Implementers have the possibility to add any other custom columns as required, e.g. updated, prepared, etc.

## Column headers (first row)

- The header field of the first column always starts with one of the term `STRUCTURE`.
  - This field must be extended with a sub-field delimiter encapsulated in squared brackets "[]", e.g. `STRUCTURE[;]`, in case the message contains multi-valued or multi-language measure or attribute values.
- The header field of the second column always contains the term `STRUCTURE_ID`.
- If option `labels=name` (see *[here](#optional-parameters)*): An additional column is added right after the artefact identification column containing the term `STRUCTURE_NAME`.
- The header field of the next column should contain the term `ACTION`. For convenience, if this column is not present, a default action ("Information") is assumed for the whole message.
- The next up to two columns contain, if option `key=series|obs|both` (see *[here](#optional-parameters)*), in this order the terms `SERIES_KEY` and/or `OBS_KEY`.
- The other columns for components contain:
  - Default: The ID of the component reported in that column, e.g. `DIM1`.
  - If option `labels=both` (see *[here](#optional-parameters)*): The ID and the localised name of the component reported in that column separated by the term ": ", e.g. `DIM1: Dimension 1`.
  - If option `labels=name` (see *[here](#optional-parameters)*): An additional column is added right after the component identification column containing the localised name of the component reported in the previous column.
- Any other custom column contains a custom but unique term, e.g. `UPDATED`.
- In case all contained sub-sequent rows contain strictly the same information, it is possible to condense the message by extending the related column header field with the equal character "=" and the content of the subsequent rows for this column, e.g. `STRUCTURE[;]=DATAFLOW`, `STRUCTURE_ID=ESTAT:NA_MAIN(1.6.0)`, `STRUCTURE_ID=ESTAT:NA_MAIN(1.6.0): National Accounts Main Aggregates`, `ACTION=I`. In this case, the content of the subsequent rows for this column can be left empty.

## Column content (all rows after header)

- The first column contains: `DATAFLOW`, `DATASTRUCTURE` or `DATAPROVISION`, depending on type of artefact for which the data contained in the message are defined: dataflow, data structure definition or data provision agreement.
- The second column contains:
  - Default: The artefact identification information in the form *AGENCY:ARTEFACT_ID(VERSION)*(1), e.g. `ESTAT:NA_MAIN(1.6.0)`.
  - If option `labels=both` (see *[here](#optional-parameters)*): The artefact identification information and its localised name separated by the term ": ", e.g. `ESTAT:NA_MAIN(1.6.0): National Accounts Main Aggregates`.
- If option `labels=name` (see *[here](#optional-parameters)*): An additional column is added right after the artefact identification column with the artefact's localised name, e.g. `National Accounts Main Aggregates`.
- The next column contains one character representing one of the current 4 action types:
  - "I": Information - Data is for information purposes.
  - "A": Append -  Data is an incremental update for an existing dataset or the provision of new data or documentation (attribute values) formerly absent. If any of the supplied data is already present, it will not replace that data. This corresponds to the "Update" value found in version 1.0 of the SDMX Technical Standards. However, according to SDMX STANDARDS: SECTION 3A PART IV: "_Append is assumed to be an incremental update. This means that one the information provided explicitly in the message should be altered. Any data attribute or observation value that is to be changed must be provided. However, the absence of an observation value or a data attribute at any level does not imply deletion; instead it is simply implied that the value is to remain unchanged. Therefore, it is valid and acceptable to send a data message with an action of Append which contains only a Series elements with attribute values. In this case, the values for the attributes will be updated. Note that it is not permissible to update data attributes using partial keys (outside of those associated with defined groups). In order to update an attribute, a full key must always be provided even if the message format does not require this._" 
  - "R": Replace - Data is to be replaced, and may also include additional data to be appended.
  - "D": Delete - Data is to be deleted. According to SDMX STANDARDS: SECTION 3A PART IV: "_Delete is assumed to be an incremental deletion. The deletion is assumed to take place of the lowest level of detail provided in the message. For example, if a delete message is sent with only a data set element, the entire data set will be deleted. On the other hand, if that data set contains a data attribute, only that data attribute value will be deleted. This same dynamic continues through the data set hierarchy. A data set containing only a series with no data attributes or observations will result in that entire series (all observations and data attributes) being deleted. If the series contains data attributes, only the supplied data attributes for that series will be deleted. Finally, if a series contains observations, then only the specified observations will be deleted. If an entire observation is to be deleted (value and data attributes), only the observation dimension should be provided. If only the observation value or particular data attributes are to be deleted, then these should be specified for the observation. Note that a group can only be used to delete the data attributes associated with it. Although the format might not require it, a full key must be provided to delete a series or observation. It is not permissible to wild card a key in order to delete more than one series or observation. Finally, to delete a data attribute or observation value it is recommended that the value to be deleted be supplied; however, it is only required that any valid value be provided._" 
  - For convenience, if this column is absent then the action "Information" is assumed.
- The next up to two columns contain, if option `key=series|obs|both`, in this order the series keys and/or the observation keys (see *[here](#optional-parameters)*). 
- The other columns for components contain:
  - Default: The ID(s) (if coded) or value(s) (if non-coded) for the component values reported in that column for the corresponding observation, e.g. `A`.
  - If option `labels=both` (see *[here](#optional-parameters)*): The ID(s) and the localised name separated by the term ": " (if coded) or the value(s) (if non-coded) for the component values reported in that column for the corresponding observation, e.g. `A: A value name`.
  - If option `labels=name` (see *[here](#optional-parameters)*): An additional column is added right after the component identification column containing the localised name, e.g. `A value name`, of the component value reported in the previous column.
  - For rows containing the information related to one specific observation, the related values for attributes attached to partial keys need to be replicated.
  - For rows containing the information related to one or more attributes attached to partial keys, in addition to these attributes only the components that are part of the partial key need to be filled, all others can be left empty. 
  - For rows containing information to be deleted, the deletion is assumed to take place of the lowest level of detail provided in the message. For that purpose, to be deleted measure or attribute values are marked with the dash character "-". Delete operations allow wildcarding dimensions by leaving the corresponding dimension field empty.
- The other custom columns contain any potentially localised custom content.

## Localisation 

- HTTP content negotiation, see [RFC 2616 - HTTP 1.1 Header Field Definitions](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html)
  - Always use this mime-type in the Accept header: `application/vnd.sdmx.data+csv; version=2.0.0`.
  - The client can indicate preferred languages through the Accept-Language header, e.g. `fr, en-gb;q=0.8, en;q=0.7`.
- Always localise all artefact names according to the preferred language. The first best language match according to the user’s preferred language choices in the http Accept-Language header (or if that is not available than according to the system's default language order) is to be used for each localisable name element. The message does however not indicate the returned language per localisable name element. In case that there is no such language match for a particular localisable name element, it is optional to return the element in a system-default language or alternatively to not return the element.
**It is recommended to indicate all languages used anywhere in the message for localised name elements through http Content-Language response header (languages of the intended audience).**  
Note: For multi-language values, all language versions are provided independently from the preferred language (see below).   

## Multi-valued components

- Some components (measures or attributes) allow for multiple values. Those multiple values are separated by a special sub-field separation character, e.g. `;`.
- This sub-field separation character has to be defined as first character in the squared bracket term of the header field of the first column, e.g. `STRUCTURE[;]`.
- Such components are indicated by having its ID followed by empty squared brackets "[]", e.g. `ATTR4[]`.
- For coded multi-valued components, if option `labels=both` (see *[here](#optional-parameters)*) then each individual value is to be prefixed with its ID and the term ": ", e.g. `A: Value A;B: Value B`.

## Non-coded multi-lingual components

- Some non-coded components (measures or attributes) allow for multi-lingual values. Those values are separated by a special sub-field separation character, e.g. `;`.
- This sub-field separation character has to be defined as first character in the squared bracket term of the header field of the first column, e.g. `STRUCTURE[;]`.
- Such components are indicated by having its ID followed by the list of possible 2-letter ISO language codes separated by the sub-field separator and encapsulated squared brackets "[]", e.g. `ATTR2[en;fr]`.
- Each individual language value is to be prefixed with its 2-letter ISO language code and a colon character ":", e.g. `en:Value;fr:Valeur`. Thus, in distinction to the ID prefix for coded values when using the HTTP accept header `labels=both` (see *[here](#optional-parameters)*), the language prefix `xx:` doesn't have an extra space character.
- Note that multi-lingual components are always non-coded and therefore do not interfere with value IDs.

## Non-coded multi-lingual multi-valued components

- Some non-coded components (measures or attributes) allow for multiple multi-lingual values. All individual values are separated by a special sub-field separation character, e.g. `;`.
- This sub-field separation character has to be defined as first character in the squared bracket term of the header field of the first column, e.g. `STRUCTURE[;]`.
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
  - If the parameter value is `id` then only the id/value of the artefacts is displayed.
  - If the parameter value is `both` then the concatenated id/value and localised name of the artefacts (see the section on [localised names](#localised-names) on how the message deals with languages) separated by `": "` are displayed. Note that the character combination `": "` could also be part of the artefact name and could therefore occur several times within the concatenated string.
  - If the parameter value is `name` then the id/value and the name of the artefacts are displayed in separate columns (see *[here](#columns)*), the ID/value column always directly preceeding its related localised name column.
- timeFormat (original|normalized; default=original):
  - If the parameter value is `original` then the time dimension (*TIME-PERIOD*) values are displayed in the SDMX *TIME_PERIOD* format as originally recorded.
  - If the parameter value is `normalized` then the time dimension (*TIME_PERIOD*) values are converted to the most granular [ISO 8601](https://www.iso.org/iso-8601-date-and-time-format.html) representation taking into account the highest frequency of the data in the message and the moment in time when the lower-frequency values were collected (which, e.g. at the ECB, is typically either at the beginning, middle or end of the reporting period). This eases comparisons and business analysis of multi-frequency values, e.g. in pivot tables. As an example, if annual and daily data are available in the message and the annual data were collected at the end of the reporting period, the formatted value for the annual period 2014 becomes 2014-12-31.
- keys (none|obs|series|comp; default=none): Request the addition of column(s) for keys. 
  - If the value is `none` (the default), no related column will be added.
  - If the value is `series`, a new column SERIES_KEY will be added after the ACTION column. The column will contain the combination of IDs/values for all the dimensions except the one(s) attached to the observation, ordered by their order in the data structure definition and separated by a dot character (.), e.g. M.USD.EUR.SP00
  - If the value is `obs`, a new column OBS_KEY will be added after the ACTION column. The column will contain the combination of IDs/values for all the dimensions, order by their order in the data structure definition and separated by a dot character (.), e.g. M.USD.EUR.SP00.2020-01
  - If the value is `both`, both a SERIES_KEY and an OBS_KEY columns must be added after the ACTION column, starting with the SERIES_KEY column.

# Examples

Note: All examples assume the minimal HTTP Accept header: `application/vnd.sdmx.data+csv; version=1.0.0`

#### 1) Ordinary case

	STRUCTURE,STRUCTURE_ID,ACTION,DIM_1,DIM_2,DIM_3,OBS_VALUE,ATTR_2,ATTR_3,ATTR_1,UPDATED
	DATAFLOW,ESTAT:NA_MAIN(1.6.0),I,A,B,2014-01,12.4,Y,"Normal, special and other values",N,2021-01-22T13:15:41Z
	DATAFLOW,ESTAT:NA_MAIN(1.6.0),I,A,B,2014-02,10.8,Y,"Normal, special and other values",Y,2021-01-22T13:15:41Z

Note: The following default parameter settings are automatically applied:
- labels=id
- timeFormat=original
- *UPDATED* is a custom column

#### 2) Components in any order, missing component(s), component with multiple values

	STRUCTURE[;],STRUCTURE_ID,ACTION,OBS_VALUE1,OBS_VALUE2,ATTR_3,ATTR_1[],DIM_2,DIM_1,DIM_3
	DATAFLOW,ESTAT:NA_MAIN(1.6.0),I,12.4,12.5,"Normal, special and other values",X;Y,B,A,2014-01
	DATAFLOW,ESTAT:NA_MAIN(1.6.0),I,10.8,10.9,"Normal, special and other values",X;Z,B,A,2014-02

#### 3) Components in any order and missing component, HTTP Accept header: `application/vnd.sdmx.data+csv; version=1.0.0; key=series`, condenced flavour

	STRUCTURE[;]=DATAFLOW,STRUCTURE_ID=ESTAT:NA_MAIN(1.6.0),ACTION=I,SERIES_KEY,OBS_VALUE1,OBS_VALUE2,ATTR_3,ATTR_1,DIM_2,DIM_1,DIM_3
	,,,A.B,12.4,12.5,"Normal, special and other values",N,B,A,2014-01
	,,,A.B,10.8,10.9,"Normal, special and other values",Y,B,A,2014-02

#### 4) Localisation: HTTP Accept header: `application/vnd.sdmx.data+csv; version=1.0.0; labels=both; key=both`, HTTP Accept-Language header: `fr-FR, en;q=0.7`, condensed flavour

	STRUCTURE[|]=DATAFLOW;STRUCTURE_ID=ESTAT:NA_MAIN(1.6.0): Principaux agrégats des comptes nationaux;ACTION=I;SERIES_KEY;OBS_KEY;DIM_1: Dimension 1;DIM_2: Dimension 2;DIM_3: Dimension 3;OBS_VALUE: Observation value;ATTR_2: Attribut 2;ATTR_3: Attribut 3;ATTR_1: Attribut 1
	;;;A.B;A.B.2014-01;A: Value A;B: Value B;2014-01: 2014-01;12,4;Y: Oui;Normal, special and other values;N: Non
	;;;A.B;A.B.2014-02;A: Value A;B: Value B;2014-02: 2014-02;10,8;Y: Oui;Normal, special and other values;Y: Oui

Note that in this example the client prefers French (fr) language with the France (FR) locale, but will also accept any type of English. Therefore, in the message the French language with the France locale is applied, transforming also the field separator from comma (,) to semicolon (;), and the decimal separator from dot (.) to comma (,).

#### 5) HTTP Accept header: `application/vnd.sdmx.data+csv; version=1.0.0; labels=both; timeFormat=normalized`, condensed flavour

	STRUCTURE[;]=DATAFLOW,STRUCTURE_ID=ESTAT:NA_MAIN(1.6.0): National Accounts Main Aggregates,ACTION=I,DIM_1: Dimension 1,DIM_2: Dimension 2,DIM_3: Dimension 3,OBS_VALUE: Observation value,ATTR_2: Attribute 2,ATTR_3: Attribute 3,ATTR_1: Attribute 1
	,,,A: Value A,B: Value B,2014-01-01,12.4,Y: Yes,"Normal, special and other values",N: No
	,,,A: Value A,B: Value B,2014-02-01,10.8,Y: Yes,"Normal, special and other values",Y: Yes

#### 6) HTTP Accept header: `application/vnd.sdmx.data+csv; version=1.0.0; labels=name`, condensed flavour

	STRUCTURE=DATAFLOW,STRUCTURE_ID=ESTAT:NA_MAIN(1.6.0),STRUCTURE_NAME=National Accounts Main Aggregates,ACTION=I,DIM_1=A,Dimension 1=Value A,DIM_2=B,Dimension 2=Value B,DIM_3,Dimension 3,OBS_VALUE,Observation value=,ATTR_1=Y,Attribute 1=Yes,"ATTR_2=""Normal, special and other values""",Attribute 2=,ATTR_3,Attribute 3
	,,,,,,,,2014-01,2014-01,12.4,,,,,,N,No
	,,,,,,,,2014-02,2014-02,10.8,,,,,,Y,Yes

#### 7) Multi-valued components, condensed flavour

	STRUCTURE[;]=DATAFLOW,STRUCTURE_ID=ESTAT:NA_MAIN(1.6.0),ACTION=I,DIM_1,DIM_2,DIM_3,OBS_VALUE,ATTR_1[],ATTR_2[],ATTR_3[]
	,,,A,B,2014-01,12.4,Value X;Value Y,"M, N & O;P & Q",A;B;C
	,,,A,B,2014-02,10.8,Value X;Value Y,"M, N & O;P & Q",A;C

#### 8) Non-coded multi-lingual components, varying dataflows, condensed flavour

	STRUCTURE[;]=DATAFLOW,STRUCTURE_ID,ACTION,DIM_1,DIM_2,DIM_3,OBS_VALUE,ATTR_1[en;fr]
	,ESTAT:NA_MAIN(1.6.0),I,A,B,2014-01,12.4,en:Any Value;fr:N'importe quelle Valeur
	,ESTAT:NA_MAIN(1.7.0),I,A,B,2014-02,10.8,"en:Value ""X"";fr:Valeur ""X"""

#### 9) Non-coded multi-lingual components, varying structures, condensed flavour

	STRUCTURE[;],STRUCTURE_ID,ACTION=I,DIM_1,DIM_2,DIM_3,OBS_VALUE,ATTR_1[en;fr]
	DATAFLOW,ESTAT:DF_NA_MAIN(1.6.0),,A,B,2014-01,12.4,en:Any Value;fr:N'importe quelle Valeur
	DATASTRUCTURE,ESTAT:DSD_NA_MAIN(1.7.0),,A,B,2014-02,10.8,"en:Value ""X"";fr:Valeur ""X"""
	DATAPROVISION,ESTAT:DPA_NA_MAIN(1.8.0),,A,B,2014-03,11.2,"en:Value ""Y"";fr:Valeur ""Y"""

#### 10) Varying actions, condensed flavour

	STRUCTURE=DATAFLOW,STRUCTURE_ID=ESTAT:NA_MAIN(1.6.0),ACTION,DIM_1,DIM_2,DIM_3,OBS_VALUE,ATTR_1
	,,A,A,B,2014-01,12.4,X
	,,R,A,B,2014-02,10.8,Y

#### 11) Data for a non-versioned(1) data structure definition

	STRUCTURE,STRUCTURE_ID,ACTION,DIM_1,DIM_2,DIM_3,OBS_VALUE,ATTR_1
	DATASTRUCTURE,AGENCY:DF_ID,I,A,B,2014-01,12.4,N
	DATASTRUCTURE,AGENCY:DF_ID,I,A,B,2014-02,10.8,Y

#### 12) Attributes attached to partial keys for a data provision agreement, condensed flavour

	STRUCTURE=DATAPROVISION,STRUCTURE_ID=AGENCY:DPA_ID(1.0.0),ACTION,DIM_2,DIM_3,ATTR_1
	,,I,B,2014-01,N
	,,I,B,2014-02,Y

#### 13) Mixing rows for attributes attached to partial keys with rows for observations 

	STRUCTURE,STRUCTURE_ID,ACTION,DIM_1,DIM_2,DIM_3,MEAS_1,ATTR_1,ATTR_2
	DATAFLOW,AGENCY:DF_ID(1.0.0),I,A,B,2014-01,12.4,N,
	DATAFLOW,AGENCY:DF_ID(1.0.0),I,,B,,,,Y

#### 14) Non-coded XHTML-formatted values with line-breaks 

	STRUCTURE,STRUCTURE_ID,ACTION,DIM_1,DIM_2,DIM_3,OBS_VALUE,ATTR_1
	DATAFLOW,ESTAT:NA_MAIN(1.6.0),I,A,B,2014-01,12.4,"<p>This is some ""xhtml"" with a line
	break</p>"
	DATAFLOW,ESTAT:NA_MAIN(1.6.0),I,A,B,2014-02,10.8,"<p>This is some other ""xhtml""</p>"

#### 15) Deleting specific measure and attribute values: all values marked with "-" are deleted

	STRUCTURE,STRUCTURE_ID,ACTION,DIM_1,DIM_2,DIM_3,OBS_VALUE,ATTR_2,ATTR_3,ATTR_1
	DATAFLOW,ESTAT:NA_MAIN(1.6.0),D,A,B,2014-01,-,,,
	DATAFLOW,ESTAT:NA_MAIN(1.6.0),D,A,B,2014-02,,,-,

#### 16) Deleting specific measure and attribute values with wildcarded dimensions: all values marked with "-" are deleted for all dimension combinations where:
   - row 2: DIM2=A
   - row 3: DIM2=B

	STRUCTURE,STRUCTURE_ID,ACTION,DIM_1,DIM_2,DIM_3,OBS_VALUE,ATTR_2,ATTR_3,ATTR_1
	DATAFLOW,ESTAT:NA_MAIN(1.6.0),D,,A,,-,,,
	DATAFLOW,ESTAT:NA_MAIN(1.6.0),D,,B,,,,-,

#### 17) Deleting whole observations with wildcarded dimensions: all observations are deleted for all dimension combinations where:
   - row 2: DIM2=A
   - row 3: DIM2=B and DIM3=C

	STRUCTURE,STRUCTURE_ID,ACTION,DIM_2,DIM_3
	DATAFLOW,ESTAT:NA_MAIN(1.6.0),D,A,,
	DATAFLOW,ESTAT:NA_MAIN(1.6.0),D,B,C,

#### 18) Deleting all data for a data structure definition:

	STRUCTURE,STRUCTURE_ID,ACTION
	DATASTRUCTURE,ESTAT:DSD_NA_MAIN(1.6.0),D
or

	STRUCTURE,STRUCTURE_ID,ACTION,DIM_1,DIM_2,DIM_3
	DATASTRUCTURE,ESTAT:DSD_NA_MAIN(1.6.0),D,,,

------------------------

**(1)** Note that since SDMX 3.0.0 the syntax *AGENCY:ARTEFACT_ID(VERSION)* allows omitting the version for non-versioned artefacts. In this case using *AGENCY:ARTEFACT_ID* is sufficient, e.g. `AGENCY:DF_ID`
