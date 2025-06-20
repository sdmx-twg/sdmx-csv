# Introduction

SDMX-CSV Data Message is an SDMX data exchange format based on the [RFC 4180](https://tools.ietf.org/html/rfc4180). CSV is a widely used standardised and simple format to exchange data supported by many tools.

SDMX-CSV integrates with other specifications, i.e.: 
- The SDMX API RESTful specification (e.g. content negotiation with mime-type to get SDMX-CSV representations, specific formats for responses, language selection through HTTP content negotiation)
- The [RFC 4180](https://tools.ietf.org/html/rfc4180) specification

## RFC 4180: A common format for CSV files
In order to benefit from best practices, SDMX-CSV is based on the rules defined in the [RFC 4180](https://tools.ietf.org/html/rfc4180), which defines a common format and MIME Type for CSV files. It is advised to read the (very short) RFC for a full list of requirements but, in a nutshell, the RFC defines rules such as:
- How the CSV file should be structured (the RFC specifies that all records must have an identical structure (determined column number), like when using an SDMX "flat" representation for data);
- When double-quotes should be used and how to escape them when needed;
- How spaces should be handled: Spaces are considered part of a field and should not be ignored;
- Which mime type should be used;
- What is the default character set, etc.

The SDMX-CSV format is flexible enough in its representation to support the needs of different target audiences:
- It is designed and optimised for the purpose of general public data dissemination of statistical data, and for usage in common statistical software.
- It allows using the messages to create pivot tables in spreadsheets applications.

# Design principles for SDMX-CSV 2.1 Data Messages (aligned with SDMX 3.1)

- In order to ensure the identifiability of the data contained in the message, the header row containing the column headers is mandatory and its content is well-defined. 
- After the mandatory header row, each row contains the information related to one specific observation or to one or more attributes attached to partial keys. For `Delete` actions a row can also concern several observations if dimensions are wildcarded.
- In [RFC 4180](https://tools.ietf.org/html/rfc4180), csv stands for "comma-separated values". However, while SDMX-CSV uses indeed the "comma" (%x2C) as the default field separator, it adopts the wider interpretation of csv as "character-separated values". It is recommended for implementers to provide SDMX-CSV messages according to the locale of the user (e.g. as indicated in the http Accept-Language header). It means that e.g. the semi-colon ‘;’ (as used typically in specific regions or countries) is acceptable as separator. See also the related example below. Note that the separator used in a message can be determined by retrieving the character that follows the fixed first column header term *STRUCTURE* (which may be extended by a squared bracket term).

## Columns

- The first column is always used for the structure type: dataflow, data structure definition or data provision agreement.
- The next one or two columns are always used for the structure's identification.
- The next column is always used for the action to be performed.
- The next up to two columns are used for the series and/or observation key.
- Each Data Structure Definition (DSD) component (dimensions, measures, attributes including those defined through a referenced Metadata Structure Definition) included in the message is represented in one or two columns. SDMX web services should return the columns in the order of components as defined in (each of) the underlying Data Structure Definition(s), grouped by type of component, thus in case of data defined by different data structures: first the dimensions of the first data structure, then the remaining dimensions of the second data structure and so forth, then the measures of the first data structure, then the remaining measures of the second data structure and so forth, then the attributes of the first data structure, then the remaining attributes of the second data structure and so forth. However, any order of these columns is valid for data uploads to SDMX-consuming systems.
- Only all those dimension columns have to be present, that are required to uniquely identify the concerned attributes and/or measures.
- Attributes can but do not need to be included even if they have a mandatory status.
- Measures can but do not have to be included.
- When an SDMX RESTful web service implements streaming, then it might not know, while generating the csv header row, which measures and attributes actually have values. Therefore, it can happen that all values presented in an attribute or measure column are left empty.
- Implementers have the possibility to add any other custom columns as required, e.g. updated, prepared, etc.

## Column headers (first row)

- The header field of the first column always contains the term `STRUCTURE`.
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

## Column content (all rows after header)

- The first column contains: `dataflow`, `datastructure` or `dataprovision`, depending on type of artefact for which the data contained in the row are defined: dataflow, data structure definition or data provision agreement.
- The second column contains:
  - Default: The artefact identification information for the data in the row in the form *AGENCY:ARTEFACT_ID(VERSION)*(1), e.g. `ESTAT:NA_MAIN(1.6.0)`.
  - If option `labels=both` (see *[here](#optional-parameters)*): The artefact identification information and its localised name separated by the term ": ", e.g. `ESTAT:NA_MAIN(1.6.0): National Accounts Main Aggregates`.
- If option `labels=name` (see *[here](#optional-parameters)*): An additional column is added right after the artefact identification column with the artefact's localised name, e.g. `National Accounts Main Aggregates`.
- The next column contains one character representing one of the current 4 action types:
  - "I": Information - Deprecated. When used to update an SDMX storage system, the *Merge* action is assumed. 
  - "A": Append -  Deprecated. When used to update an SDMX storage system, the *Merge* action is assumed.
  - "M": Merge - Data or data-related reference metadata is to be merged, through either update or insertion depending on already existing information. This operation does not allow deleting any component values. Updating individual values in multi-valued measure, attribute or data-related reference metadata values is not supported either. The complete multi-valued value is to be provided.  Only non-dimensional components (measure, attribute or data-related reference metadata values) can be **omitted** (\<empty\> cell or column is absent) as long as at least one of those components is present. Bulk merges are thus not supported. Only the provided values are merged.  Dimension values for higher-level (data-related reference metadata) attributes can be **switched-off** (using `~`) when those are not attached to these dimensions.  All observations as well as the sets of data-related reference metadata attributes at specific dimension combinations impacted by the *Merge* action change their time stamp when used to update an SDMX storage system.
  - "R": Replace - Data or data-related reference metadata is to be replaced, through either update, insert or delete depending on already existing information. A full replacement is hereby assumed to take place at specific “replacement levels”: for entire observations and for any specific dimension combination for data-related reference metadata attributes. Within these “replacement levels” the provided values are inserted or updated, and omitted values are deleted. Values provided for the other attributes (those above the observation level) are merged (see *Merge* action).  Only non-dimensional components (measure, attribute or reference metadata values) can be **omitted** (\<empty\> cell or column is absent). Bulk replacing is thus not supported.  Dimension values for higher-level (data-related reference metadata) attributes can be **switched-off** (using `~`) when those are not attached to these dimensions.  Replacing non-existing elements is not resulting in an error.  All observations as well as the sets of data-related reference metadata attributes at specific dimension combinations impacted by the *Replace* action change their time stamp when used to update an SDMX storage system.  Because the *replace* action always takes place at specific levels, it cannot be used to replace a whole dataset or a whole series. However, a “*replace all*” effect can be achieved by combining a *Delete* row containing a completely wildcarded key (where all dimension values are omitted) with *Merge* or *Replace* rows within the same data message. Similarly, to replace a whole series, a message can combine a *delete* row containing only the partial key of the series (where the not used dimension values are omitted) with *Merge* or *Replace* rows for that series.
  - "D": Delete - Data or data-related reference metadata is to be deleted. Deletion is hereby assumed to take place at the lowest level of detail provided in the message.  Any component (including dimensions) can be **omitted** (\<empty\> cell or column is absent). Omitting dimension values allows for bulk deletions. Partially omitting non-dimension component values allows restricting the deletion of measure, attribute or data-related reference metadata values to the ones being present. Instead of real values for non-dimensional components, it is sufficient to use any valid value, e.g. the dash character `-`.  With this, all dataflow data, any slices of observations for dimension groups such as time series, observations or individual measure, attribute and data-related reference metadata attributes values can be deleted.  Dimension values for higher-level (data-related reference metadata) attributes can be **switched-off** (using `~`) when those are not attached to these dimensions.  Deleting non-existing elements or values is not resulting in an error.  All observations as well as the sets of attributes and data-related reference metadata at higher partial keys impacted by the *Delete* action change their time stamp when used to update an SDMX storage system.
  - For convenience, if this column is absent then the *Merge* action is assumed.  
    For more details see [here](#further-details-for-data-actions).
- The next up to two columns contain, if option `key=series|obs|both`, in this order the series keys and/or the observation keys (see *[here](#optional-parameters)*). 
- The other columns for components contain:
  - Default: The ID(s) (if coded) or value(s) (if non-coded) for the component values reported in that column for the corresponding observation, e.g. `A`.
  - If option `labels=both` (see *[here](#optional-parameters)*): The ID(s) and the localised name separated by the term ": " (if coded) or the value(s) (if non-coded) for the component values reported in that column for the corresponding observation, e.g. `A: A value name`.
  - If option `labels=name` (see *[here](#optional-parameters)*): An additional column is added right after the component identification column containing the localised name, e.g. `A value name`, of the component value reported in the previous column. It is empty if the value has no localised name.
  - For rows containing the information related to one specific observation, the related values for attributes attached to partial keys may have to be replicated.
  - For rows containing the information related to one or more attributes attached to partial keys, in addition to these attributes only the components that are part of the partial key need to be filled, all other components can be left empty. Also the columns not related to the attribute's data structure (when data from different data structures are present) are to be left empty.
  - For rows containing information to be deleted, the deletion is assumed to take place at the lowest level of detail provided in the message. For that purpose, to be deleted measure or attribute values are non-empty, e.g. marked with the dash character "-". Delete operations allow wildcarding dimensions by leaving the corresponding dimension field empty.
- The other custom columns contain any potentially localised custom content.

## Further details for data actions

The following convention is used to indicate the state of components in data messages:

|  |  | **Dimension value is** | | **Measure, attribute or reference metadata value is** | |
| --- | --- | --- | --- | --- | --- |
|  |  | **Omitted** | **switched off** | **Omitted** | **Present** |
| Action | Delete | bulk deletion: dimension value doesn't matter | only for irrelevant dimensions:1) higher-level (reference metadata) attributes not attached to this dimension(incl. TIME\_PERIOD)2) measures and attributes not attached to this dimension if the DSD allows for an ‘evolving structure’ (excl. TIME\_PERIOD) | to be deleted only if **all** non-dimension components are omitted | to be deleted |
| | Merge | *bulk merge is not permitted* | (see above) | not to be changed | to be updated/inserted |
| | Replace | *bulk replace is not permitted* | (see above) | at permitted replacement levels: to be deleted, otherwise not to be changed | to be updated/inserted |
| Format | CSV | \<empty\> cell or column is absent | ~ | \<empty\> cell or column is absent | any valid or intentionally missing value |

**Important notes:**

The terms “*delete*”, “*merge*” and “*replace*” do **not** imply a physical replacement or deletion of values in the underlying database. To minimize the physical resource requirements, SDMX web service implementations that do not support the *includeHistory* and *asOf* URL parameters might physically replace the existing values in the database. SDMX web services that neither support the *updatedAfter* URL parameter might also implement physical deletions. However, SDMX web services that support these parameters (or other time-machine features), would not overwrite or delete the physical values.

SDMX web services that support the *includeHistory* or *asOf* URL parameters should never allow deleting their **historic** data content because this would interfere with the interests of data consumers, such as data aggregators. Therefore, a specific feature to physically delete previous (outdated) content is intentionally not added to the SDMX standard syntax. If such a feature is required by an organisation, then it needs to be implemented as a custom feature outside the SDMX standard.

Likewise, all SDMX-compliant systems that do (or are configured to) support the *updatedAfter* URL parameter need to systematically retain the information about deleted data (or data-related reference metadata).

All datasets – even with varying actions – within a single data message have always to be treated as **ACID transaction** to guarantee “transactional safety” (full data consistency and validity despite errors, power failures, and other mishaps). These datasets are to be processed in the order of appearance in the message. The advantage of such data messages is thus the ability to bundle separate *delete* and *replace* or *merge* actions into one transactional data message.

**Recommended[^1] dataset actions in SDMX web service responses to GET data queries:**

1. Without the *updatedAfter*, *includeHistory*, *detail*, *attributes* or *measures* URL parameters:  
   
   The response message should contain the retrieved data in a *Replace* dataset (instead of the previous *information* dataset).

1. Without the *updatedAfter* and *includeHistory*, but with *detail*, *attributes* or *measures* URL parameters:  
   
   The response message should contain the retrieved data in a *Merge* dataset (instead of the previous *Information* dataset).

1. With the *updatedAfter* URL parameter:  
   
   The response must include the information of all previously updated, inserted and deleted data or data-related reference metadata, even if bulk deletions have been used. One of the two approaches are possible:

   * a *Delete* dataset for entirely deleted observations and for entirely deleted sets of (data-related reference metadata) attribute values attached to specific dimension combinations and  
   a *Replace* dataset for all other changed observations and changed attribute and data-related reference metadata values attached to specific dimension combinations, or  
   * a *Delete* dataset for entirely deleted observations, for entirely deleted sets of (data-related reference metadata) attribute values attached to specific dimension combinations and for individually deleted mesure, attribute and reference metadata values and  
   a *Merge* dataset for all other updated or inserted observation, attribute and data-related reference metadata values.
   
   The DB synchronization use case requires that the generated response must always allow achieving to replicate the exact same punctual data content as currently stored in the queried data source.

1. With the *includeHistory* URL parameter:  
   
   Using a number of datasets with *Delete*, *Replace* or *Merge* actions and limited in their validity time span that allow achieving to replicate the exact same punctual data contents as previously stored in the queried data source.

1. With the *asOf* URL parameter:  
   
   The recommendations of 1 and 2 apply depending on the other parameters. In addition, the returned dataset should have its validity time span limited to the point in time requested in the *asOf* parameter.

[^1]: So far this is recommended for systems that do not require backward-compatibility. Later, with SDMX 4.0, this may generally be made mandatory.

## Intentionally missing values

To indicate **intentionally missing** observations, attributes and reference metadata values, even if mandatory, the following special values are to be used in SDMX-CSV:
- Numeric data types float and double: `NaN`
- All other data types: `#N/A`

## Localisation 

- HTTP content negotiation, see [RFC 2616 - HTTP 1.1 Header Field Definitions](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html)
  - Always use this mime-type in the Accept header: `application/vnd.sdmx.data+csv; version=2.0.0`.
  - The client can indicate preferred languages through the Accept-Language header, e.g. `fr, en-gb;q=0.8, en;q=0.7`.
- Always localise all artefact names according to the preferred language. The first best language match according to the user’s preferred language choices in the http Accept-Language header (or if that is not available than according to the system's default language order) is to be used for each localisable name element. The message does however not indicate the returned language per localisable name element. In case that there is no such language match for a particular localisable name element, it is optional to return the element in a system-default language or alternatively to not return the element.
**It is recommended to indicate all languages used anywhere in the message for localised name elements through http Content-Language response header (languages of the intended audience).**  
Note: For multi-language values, all language versions are provided independently from the preferred language (see below).   

## Multi-valued components and nested metadata attributes

- Some components (measures or attributes) allow for multiple values. Those multiple values are separated by a special sub-field separation character, e.g. `;`.
- This sub-field separation character has to be defined as first character in the squared bracket term of the header field of the first column, e.g. `STRUCTURE[;]`.
- Such components are indicated by having their IDs followed by empty squared brackets "[]", e.g. `ATTR4[]`.
- For coded multi-valued components, if option `labels=both` (see *[here](#optional-parameters)*) then each individual value is to be prefixed with its ID and the term ": ", e.g. `A: Value A;B: Value B`.
- Each metadata attribute is also to be presented in its own column(s), even if the metadata attributes are nested. In that case, the attribute IDs in the column headers are pre-fixed with the IDs of the related parent attribute(s) separated by a dot `.`, e.g. `CONTACT[].NAME[]`. All the values corresponding to one attribute are presented like a multi-valued component by respecting their position in the nested attribute tree, e.g. `name for contact 1;name for contact 2`. Parent branches in that attribute tree without a value for a specific attribute need to be indicated by leaving the corresponding multi-value sub-field empty, e.g. `name for contact 1;;name for contact 3`. Nested parent branches are indicated by using nested double quotes. Note that fields containing double quotes must themselves be encapsulated in double quotes and that nested inner double quotes need to be doubled recoursively, e.g. `"""name 1 for contact 1;name 2 for contact 1"";""name 1 for contact 2;name 2 for contact 2"""`.

## Non-coded multi-lingual components

- Some non-coded components (measures or attributes) allow for multi-lingual values. Those values are separated by a special sub-field separation character, e.g. `;`.
- This sub-field separation character has to be defined as first character in the squared bracket term of the header field of the first column, e.g. `STRUCTURE[;]`.
- Such components are indicated by having their IDs followed by the list of possible 2-letter ISO language codes separated by the sub-field separator and encapsulated squared brackets "[]", e.g. `ATTR2[en;fr]`.
- Each individual language value is to be prefixed with its 2-letter ISO language code and a colon character ":", e.g. `en:Value;fr:Valeur`. Thus, in distinction to the ID prefix for coded values when using the HTTP accept header `labels=both` (see *[here](#optional-parameters)*), the language prefix `xx:` doesn't have an extra space character.
- Note that multi-lingual components are always non-coded and therefore do not interfere with value IDs.

## Non-coded multi-lingual multi-valued components

- Some non-coded components (measures or attributes) allow for multiple multi-lingual values. All individual values are separated by a special sub-field separation character, e.g. `;`.
- This sub-field separation character has to be defined as first character in the squared bracket term of the header field of the first column, e.g. `STRUCTURE[;]`.
- Such components are indicated by having its ID followed by the list of possible language codes separated by the sub-field separator and encapsulated squared brackets "[]", e.g. `ATTR2[en;fr;de]`.
- Each individual language value is to prefixed with its 2-letter ISO language code and a colon character ":", e.g. `en:Value1`.
- Each multi-lingual value set is to be encapsulated in double quotes, e.g. `"en:Value1;fr:Valeur1";"en:Value2;de:Wert2"`. However, note that fields containing double quotes must themselves be encapsulated in double quotes and that the inner double quotes need to be doubled, thus the complete example is `"""en:Value1;fr:Valeur1"";""en:Value2;de:Wert2"""`.

## Non-coded XHTML-valued components

- Some non-coded components (measures or attributes) allow for XHTML values.
- Each XHTML value is to be encapsulated in double quotes, e.g. `"<p>This is some ""metadata html""</p>"`. Remember that the inner quotes need to be doubled.
- The CSV format allows fields to contain line breaks if those fields are enclosed in double quotes. Thus XHTML values can also contain line breaks.

# Optional parameters

Optional parameters can be added to the HTTP Accept header. They need to be separated by the character combination `"; "`.
- labels (id|name|both; default=id): This parameter applies to all Nameable SDMX Artefacts contained in the header and the body of the message: 
  - If the parameter value is `id` then only the id/value of the artefacts is displayed.
  - If the parameter value is `name` then the id/value and the name of the artefacts are displayed in separate columns (see *[here](#columns)*), the ID/value column always directly preceding its related localised name column.
  - If the parameter value is `both` then the concatenated id/value and localised name of the artefacts (see the section on [localised names](#localised-names) on how the message deals with languages) separated by `": "` are displayed. Note that the character combination `": "` could also be part of the artefact name and could therefore occur several times within the concatenated string.
- timeFormat (original|normalized; default=original):
  - If the parameter value is `original` then the time dimension (*TIME-PERIOD*) values are displayed in the SDMX *TIME_PERIOD* format as originally recorded.
  - If the parameter value is `normalized` then the time dimension (*TIME_PERIOD*) values are converted to the most granular [ISO 8601](https://www.iso.org/iso-8601-date-and-time-format.html) representation taking into account the highest frequency of the data in the message and the moment in time when the lower-frequency values were collected (which, e.g. at the ECB, is typically either at the beginning, middle or end of the reporting period). This eases comparisons and business analysis of multi-frequency values, e.g. in pivot tables. As an example, if annual and daily data are available in the message and the annual data were collected at the end of the reporting period, the formatted value for the annual period 2014 becomes 2014-12-31.
- keys (none|obs|series|both; default=none): Request the addition of column(s) for keys. 
  - If the value is `none` (the default), no related column will be added.
  - If the value is `obs`, a new column OBS_KEY will be added after the ACTION column. The column will contain the combination of IDs/values for all the dimensions, order by their order in the data structure definition and separated by a dot character (.), e.g. M.USD.EUR.SP00.2020-01
  - If the value is `series`, a new column SERIES_KEY will be added after the ACTION column. The column will contain the combination of IDs/values for all the dimensions except the one(s) attached to the observation, ordered by their order in the data structure definition and separated by a dot character (.), e.g. M.USD.EUR.SP00
  - If the value is `both`, both a SERIES_KEY and an OBS_KEY columns must be added after the ACTION column, starting with the SERIES_KEY column.

# Examples

Note: All examples assume the minimal HTTP Accept header: `application/vnd.sdmx.data+csv; version=2.1.0`

#### 1) Ordinary case

	STRUCTURE,STRUCTURE_ID,ACTION,DIM_1,DIM_2,DIM_3,OBS_VALUE,ATTR_2,ATTR_3,ATTR_1,UPDATED
	dataflow,ESTAT:NA_MAIN(1.6.0),M,A,B,2014-01,12.4,Y,"Normal, special and other values",N,2021-01-22T13:15:41Z
	dataflow,ESTAT:NA_MAIN(1.6.0),M,A,B,2014-02,10.8,Y,"Normal, special and other values",Y,2021-01-22T13:15:41Z

Notes:  
- The following default parameter settings are automatically applied:
  - labels=id
  - timeFormat=original
- *UPDATED* is a custom column

#### 2) Components in any order, missing component(s), component with multiple values

	STRUCTURE[;],STRUCTURE_ID,ACTION,OBS_VALUE1,OBS_VALUE2,ATTR_3,ATTR_1[],DIM_2,DIM_1,DIM_3
	dataflow,ESTAT:NA_MAIN(1.6.0),M,12.4,12.5,"Normal, special and other values",X;Y,B,A,2014-01
	dataflow,ESTAT:NA_MAIN(1.6.0),M,10.8,10.9,"Normal, special and other values",X;Z,B,A,2014-02

#### 3) Components in any order and missing component, HTTP Accept header: `application/vnd.sdmx.data+csv; version=1.0.0; key=series`

	STRUCTURE[;],STRUCTURE_ID,ACTION,SERIES_KEY,OBS_VALUE1,OBS_VALUE2,ATTR_3,ATTR_1,DIM_2,DIM_1,DIM_3
	dataflow,ESTAT:NA_MAIN(1.6.0),M,A.B,12.4,12.5,"Normal, special and other values",N,B,A,2014-01
	dataflow,ESTAT:NA_MAIN(1.6.0),M,A.B,10.8,10.9,"Normal, special and other values",Y,B,A,2014-02

#### 4) Localisation: HTTP Accept header: `application/vnd.sdmx.data+csv; version=1.0.0; labels=both; key=both`, HTTP Accept-Language header: `fr-FR, en;q=0.7`

	STRUCTURE[|];STRUCTURE_ID;ACTION;SERIES_KEY;OBS_KEY;DIM_1: Dimension 1;DIM_2: Dimension 2;DIM_3: Dimension 3;OBS_VALUE: Observation value;ATTR_2: Attribut 2;ATTR_3: Attribut 3;ATTR_1: Attribut 1
	dataflow;ESTAT:NA_MAIN(1.6.0): Principaux agrégats des comptes nationaux;M;A.B;A.B.2014-01;A: Value A;B: Value B;2014-01: 2014-01;12,4;Y: Oui;Normal, special and other values;N: Non
	dataflow;ESTAT:NA_MAIN(1.6.0): Principaux agrégats des comptes nationaux;M;A.B;A.B.2014-02;A: Value A;B: Value B;2014-02: 2014-02;10,8;Y: Oui;Normal, special and other values;Y: Oui

Note that in this example the client prefers French (fr) language with the France (FR) locale, but will also accept any type of English. Therefore, in the message the French language with the France locale is applied, transforming also the field separator from comma (,) to semicolon (;), and the decimal separator from dot (.) to comma (,).

#### 5) HTTP Accept header: `application/vnd.sdmx.data+csv; version=1.0.0; labels=both; timeFormat=normalized`

	STRUCTURE[;],STRUCTURE_ID,ACTION,DIM_1: Dimension 1,DIM_2: Dimension 2,DIM_3: Dimension 3,OBS_VALUE: Observation value,ATTR_2: Attribute 2,ATTR_3: Attribute 3,ATTR_1: Attribute 1
	dataflow,ESTAT:NA_MAIN(1.6.0): National Accounts Main Aggregates,M,A: Value A,B: Value B,2014-01-01,12.4,Y: Yes,"Normal, special and other values",N: No
	dataflow,ESTAT:NA_MAIN(1.6.0): National Accounts Main Aggregates,M,A: Value A,B: Value B,2014-02-01,10.8,Y: Yes,"Normal, special and other values",Y: Yes

#### 6) HTTP Accept header: `application/vnd.sdmx.data+csv; version=1.0.0; labels=name`

	STRUCTURE,STRUCTURE_ID,STRUCTURE_NAME,ACTION,DIM_1,Dimension 1,DIM_2,Dimension 2,DIM_3,Dimension 3,OBS_VALUE,Observation value,ATTR_1,Attribute 1,ATTR_2,Attribute 2,ATTR_3,Attribute 3
	dataflow,ESTAT:NA_MAIN(1.6.0),National Accounts Main Aggregates,M,A,Value A,B,Value B,2014-01,2014-01,12.4,,Y,Yes,"Normal, special and other values",,N,No
	dataflow,ESTAT:NA_MAIN(1.6.0),National Accounts Main Aggregates,M,A,Value A,B,Value B,2014-02,2014-02,10.8,,Y,Yes,"Normal, special and other values",,Y,Yes

#### 7) Multi-valued components

	STRUCTURE[;],STRUCTURE_ID,ACTION,DIM_1,DIM_2,DIM_3,OBS_VALUE,ATTR_1[],ATTR_2[],ATTR_3[]
	dataflow,ESTAT:NA_MAIN(1.6.0),M,A,B,2014-01,12.4,Value X;Value Y,"M, N & O;P & Q",A;B;C
	dataflow,ESTAT:NA_MAIN(1.6.0),M,A,B,2014-02,10.8,Value X;Value Y,"M, N & O;P & Q",A;C

#### 8) Non-coded multi-lingual components, varying dataflows based on the same underlying data structure

	STRUCTURE[;],STRUCTURE_ID,ACTION,DIM_1,DIM_2,DIM_3,OBS_VALUE,ATTR_1[en;fr]
	dataflow,ESTAT:NA_MAIN(1.6.0),M,A,B,2014-01,12.4,en:Any Value;fr:N'importe quelle Valeur
	dataflow,ESTAT:NA_MAIN(1.7.0),M,A,B,2014-02,10.8,"en:Value ""X"";fr:Valeur ""X"""

#### 9-A) Varying structural artefacts based on same underlying data structure

	STRUCTURE[;],STRUCTURE_ID,ACTION,DIM_1,DIM_2,DIM_3,OBS_VALUE,ATTR_1[en;fr]
	dataflow,ESTAT:DF_NA_MAIN(1.6.0),M,A,B,2014-01,12.4,en:Any Value;fr:N'importe quelle Valeur
	datastructure,ESTAT:DSD_NA_MAIN(1.7.0),M,A,B,2014-02,10.8,"en:Value ""X"";fr:Valeur ""X"""
	dataprovision,ESTAT:DPA_NA_MAIN(1.8.0),M,A,B,2014-03,11.2,"en:Value ""Y"";fr:Valeur ""Y"""

#### 9-B) Varying structural artefacts based on different underlying data structures

	STRUCTURE[;],STRUCTURE_ID,ACTION,DIM_A1B1,DIM_A2,DIM_A3C2,DIM_B2,DIM_C1,DIM_C3,MEAS_A1B1C1,MEAS_C2,ATTR_A1,ATTR_B1
	dataflow,ESTAT:DF_A(1.6.0),M,DIMVAL_A1B1,DIMVAL_A2,DIMVAL_A3C2,,,,"MEASVAL_A1B1C1",,"ATTRVAL_A1",
	datastructure,ESTAT:DSD_B(1.7.0),M,DIMVAL_A1B1,,,DIMVAL_B2,,,"MEASVAL_A1B1C1",,,"ATTRVAL_B1"
	dataprovision,ESTAT:DPA_C(1.8.0),M,,,DIMVAL_A3C2,,DIMVAL_C1,DIMVAL_C3,"MEAS_A1B1C1","MEAS_C2",,

#### 10) Varying actions

	STRUCTURE,STRUCTURE_ID,ACTION,DIM_1,DIM_2,DIM_3,OBS_VALUE,ATTR_1
	dataflow,ESTAT:NA_MAIN(1.6.0),M,A,B,2014-01,12.4,X
	dataflow,ESTAT:NA_MAIN(1.6.0),R,A,B,2014-02,10.8,Y

#### 11) Data for a non-versioned(1) data structure definition

	STRUCTURE,STRUCTURE_ID,ACTION,DIM_1,DIM_2,DIM_3,OBS_VALUE,ATTR_1
	datastructure,AGENCY:DF_ID,M,A,B,2014-01,12.4,N
	datastructure,AGENCY:DF_ID,M,A,B,2014-02,10.8,Y

#### 12) Attributes attached to partial keys for a data provision agreement

	STRUCTURE,STRUCTURE_ID,ACTION,DIM_2,DIM_3,ATTR_1
	dataprovision,AGENCY:DPA_ID(1.0.0),M,B,2014-01,N
	dataprovision,AGENCY:DPA_ID(1.0.0),M,B,2014-02,Y

#### 13) Mixing rows for attributes attached to partial keys with rows for observations 

	STRUCTURE,STRUCTURE_ID,ACTION,DIM_1,DIM_2,DIM_3,MEAS_1,ATTR_1,ATTR_2
	dataflow,AGENCY:DF_ID(1.0.0),M,A,B,2014-01,12.4,N,
	dataflow,AGENCY:DF_ID(1.0.0),M,,B,,,,Y

#### 14) Nested metadata attributes attached to partial keys

	STRUCTURE,STRUCTURE_ID,ACTION,DIM_2,COLLECTION.METHOD[en;fr],CONTACT[],CONTACT[].NAME[]
	dataflow,AGENCY:DF_ID(1.0.0),M,A,en:AAA;fr:BBB,Contact 1;Contact 2,"""Contact 1 Name 1;Contact 1 Name 2"";""Contact 1 Name 1;Contact 2 Name 2"""
	dataflow,AGENCY:DF_ID(1.0.0),M,B,en:CCC;fr:DDD,Contact 1;Contact 2;Contact 3,"""Contact 1 Name 1;Contact 1 Name 2"";;""Contact 3 Name 1;Contact 3 Name 2"""

#### 15) Non-coded XHTML-formatted values with line-breaks 

	STRUCTURE,STRUCTURE_ID,ACTION,DIM_1,DIM_2,DIM_3,OBS_VALUE,ATTR_1
	dataflow,ESTAT:NA_MAIN(1.6.0),M,A,B,2014-01,12.4,"<p>This is some ""xhtml"" with a line
	break</p>"
	dataflow,ESTAT:NA_MAIN(1.6.0),M,A,B,2014-02,10.8,"<p>This is some other ""xhtml""</p>"

#### 16) Deleting specific measure and attribute values: all non-empty values (e.g. marked with "-") are deleted

	STRUCTURE,STRUCTURE_ID,ACTION,DIM_1,DIM_2,DIM_3,OBS_VALUE,ATTR_2,ATTR_3,ATTR_1
	dataflow,ESTAT:NA_MAIN(1.6.0),D,A,B,2014-01,-,,,
	dataflow,ESTAT:NA_MAIN(1.6.0),D,A,B,2014-02,,,-,

#### 17) Deleting specific measure and attribute values with wildcarded dimensions: all non-empty values (e.g. marked with "-") are deleted for all dimension combinations where:
   - row 2: DIM2=A
   - row 3: DIM2=B

	STRUCTURE,STRUCTURE_ID,ACTION,DIM_1,DIM_2,DIM_3,OBS_VALUE,ATTR_2,ATTR_3,ATTR_1
	dataflow,ESTAT:NA_MAIN(1.6.0),D,,A,,-,,,
	dataflow,ESTAT:NA_MAIN(1.6.0),D,,B,,,,-,

#### 18) Deleting whole observations with wildcarded dimensions: all observations are deleted for all dimension combinations where:
   - row 2: DIM2=A
   - row 3: DIM2=B and DIM3=C

	STRUCTURE,STRUCTURE_ID,ACTION,DIM_2,DIM_3
	dataflow,ESTAT:NA_MAIN(1.6.0),D,A,,
	dataflow,ESTAT:NA_MAIN(1.6.0),D,B,C,

#### 19) Deleting all data for a data structure definition:

	STRUCTURE,STRUCTURE_ID,ACTION
	datastructure,ESTAT:DSD_NA_MAIN(1.6.0),D
or

	STRUCTURE,STRUCTURE_ID,ACTION,DIM_1,DIM_2,DIM_3
	datastructure,ESTAT:DSD_NA_MAIN(1.6.0),D,,,

------------------------

**(1)** Note that since SDMX 3.0.0 the syntax *AGENCY:ARTEFACT_ID(VERSION)* allows omitting the version for non-versioned artefacts. In this case using *AGENCY:ARTEFACT_ID* is sufficient, e.g. `AGENCY:DF_ID`
