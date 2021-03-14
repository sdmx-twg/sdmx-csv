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

# Design principles for SDMX-CSV 2.0 Metadata Messages (aligned with SDMX 3.0.0)

- In order to ensure the identifiability of the metadata contained in the message, the header row containing the column headers is mandatory and its content is well-defined. 
- An SDMX-CSV referential metadata messages contains metadata attribute values for one or more metadataset reported for one or more metadataflows or metadata provision agreements.
- After the mandatory header row, each row contains the information related to one specific metadata attribute in a metadataset attached to an identifiable artefact. 
- In [RFC 4180](https://tools.ietf.org/html/rfc4180), csv stands for "comma-separated values". However, while SDMX-CSV uses indeed the "comma" (%x2C) as the default field separator, it adopts the wider interpretation of csv as "character-separated values". It is recommended for implementers to provide SDMX-CSV messages according to the locale of the user (e.g. as indicated in the http Accept-Language header). It means that e.g. the semi-colon ‘;’ (as used typically in specific regions or countries) is acceptable as separator. See also the examples below. Note that the separator used in a message can be determined by retrieving the character that follows the header field of the first column which extended by a squared bracket term (see below). Implementers need here to be careful since the squared bracket term can itself contain escaped (doubled) bracket characters.

## Columns

- The first column is always used for the metadataflow or metadata provision agreement identification and other optional pieces of identification information.
- The second column is always used for the metadataset identification information.
- The third column is always used for the target type.
- The fourth column is always used for the target identification.
- The fifth column is always used for the metadata attribute identification.
- The sixth column is always used for the metadata attribute value. 
- Implementers have the possibility to add any other custom columns as required, e.g. publicationPeriod, publicationYear, reportingBegin, reportingEnd, prepared, etc.

## Column headers (first row)

- The header field of the first column always starts with one of the terms `METADATAFLOW` or `METADATAPROVISION`, depending on type of artefact for which the metadata contained in the message are defined: metadataflow or metadata provision agreement.
- Optionally, the header of the first column can be extended with the following pieces of information encapsulated in squared brackets "[]":
  - **Variant A**: a sub-field delimiter, e.g. `METADATAFLOW[;]`.
  - **Variant B** (only if all metadata contained in the message belong to the same artefact): a sub-field delimiter and the artefact identification information in the form *AGENCY:ARTEFACT_ID(VERSION)*(1), e.g. `METADATAFLOW[;AGENCY:MDF(1.0.0)]`.
  - **Variant C** (only if all metadata contained in the message belong to the same artefact and if option `labels=both` (see *[here](#optional-parameters)*)): a sub-field delimiter, the artefact identification information in the form *AGENCY:ARTEFACT_ID(VERSION)*(1), a separation term ": " and the artefact's localised name, e.g. `METADATAFLOW[;AGENCY:MDF(1.0.0): Metadataflow name]`.
  - For the variant C, if the artefact name contains squared brackets then they must be doubled, e.g. `METADATAFLOW[;AGENCY:MDF(1.0.0): Metadataflow name [[1]]]`.
- The header of the second column always contains `METADATASET`.
  - If all metadata contained in the message belong to the same metadataset then this field can be extended with the metadata identification information in the form *AGENCY:ARTEFACT_ID(VERSION)*(1) encapsulated in squared brackets "[]", e.g. `METADATASET[AGENCY:MDS(1.0.0)]`. In that case, the metadata identification information does not need to be repeated in each subsequent row. In addition, if option `labels=both` (see *[here](#optional-parameters)*) then the ID is complemented with the localised name of the metadataset separated by the term ": ", e.g. `METADATASET[AGENCY:MDS(1.0.0): Metadataset name]`.
- The header of the third column always contains `TARGET_TYPE`.
  - If all metadata contained in the message are related to a target of the same type then this field can be extended with the target type according to the resource names defined for Structural Metadata Queries encapsulated in squared brackets "[]", e.g. `TARGET_TYPE[dataflow]`. In that case, the target type does not need to be repeated in each subsequent row.
- The header of the fourth column always contains `TARGET_ID`.
  - If all metadata contained in the message are related to the same target then this field can be extended with the target identification information in the form *AGENCY:ARTEFACT_ID(VERSION)*(1) encapsulated in squared brackets "[]", e.g. `TARGET_ID[AGENCY:DF(1.0.0)]`. In that case, the target does not need to be repeated in each subsequent row. In addition, if option `labels=both` (see *[here](#optional-parameters)*) then the ID is complemented with the localised name of the target separated by the term ": ", e.g. `TARGET_ID[AGENCY:DF(1.0.0): Dataflow name]`.
- The header of the fifth column always contains `ATTRIBUTE_ID`.
- The header of the sixth column `ATTRIBUTE_VALUE`. 
- Any other custom column contains a custom but unique term, e.g. `publicationPeriod`.

## Column content (all rows after header)

- The first column contains:
  - Default: The artefact identification information in the form *AGENCY:ARTEFACT_ID(VERSION)*(1), e.g. `AGENCY:NA_MAIN(1.6.0)`.
  - Default if option `labels=both` (see *[here](#optional-parameters)*): The artefact identification information in the form *AGENCY:ARTEFACT_ID(VERSION)* and its localised name separated by the term ": ", e.g.  `AGENCY:NA_MAIN(1.6.0): National Accounts Main Aggregates`.
  - If variant B or if option `labels=both` (see *[here](#optional-parameters)*) with variant C: The fields are being left empty.
- The second column always contains the metadataset identification information in the form *AGENCY:ARTEFACT_ID(VERSION)*(1), e.g. `AGENCY:MD_SET(1.0.0)`.
  - If option `labels=both` (see *[here](#optional-parameters)*): The ID and the localised name of the metadataset separated by the term ": ", e.g. `ESTAT:MD_SET(1.0.0): Metadataset 1`.
  - If the metadataset identification information is already given in the column header then this field is left empty.
- The third column always contains the target type according to the resource names defined for Structural Metadata Queries, e.g. `dataflow`.
- The fourth column always contains the target identification information in the form *AGENCY:ARTEFACT_ID(VERSION)*(1), e.g. `AGENCY:DF(1.0.0)`.
  - If option `labels=both` (see *[here](#optional-parameters)*): The ID and the localised name of the target separated by the term ": ", e.g. `AGENCY:DF(1.0.0): Dataflow 1`.
- The fifth column always contains the metadata attribute identification information included all parent metadata attributes separated by a dot "." in the form *METADATA_ID[.METADATA_ID]+*, complemented by at least one pair of squared brackets to indicate the position of that attribute in the current branch of nested attributes within the metadataset, e.g. `CONTACT[2].PHONE[3]`. An additional pair of squared brackets at the end of the identification information is to be used if the value of the corresponding ATTRIBUTE_VALUE column contains multiple values and/or multiple languages and in the later case encapsulates the ISO 2-letter language codes that can be encountered in that column, separated by the character in the squared brackets defined in the METADATAFLOW/METADATAPROVISION column header field, e.g. `CONTACT[2].PHONE[1][]` for multiple values or `GENERAL_INFO[1].DESCRIPTION[1][en;fr]` for multi-lingual values. 
- The sixth column contains the attribute value, e.g. `"<p>An XHTML text</p>"`. All string/textual values (complete string between column-separating commas including ID's or language codes) should always be encapsulated in quotation marks, especially if they contain commas or quotation marks. Quotation marks in strings/textual values must always be escaped by double-quotes.
- The other custom columns contain any potentially localised custom content.

## Localisation 

- HTTP content negotiation, see [RFC 2616 - HTTP 1.1 Header Field Definitions](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html)
  - Always use this mime-type in the Accept header: `application/vnd.sdmx.metadata+csv; version=1.0.0`.
  - The client can indicate preferred languages through the Accept-Language header, e.g. `fr, en-gb;q=0.8, en;q=0.7`.
- Always localise all artefact names according to the preferred language. The first best language match according to the user’s preferred language choices in the http Accept-Language header (or if that is not available than according to the system's default language order) is to be used for each localisable name element. The message does however not indicate the returned language per localisable name element. In case that there is no such language match for a particular localisable name element, it is optional to return the element in a system-default language or alternatively to not return the name element.
**It is recommended to indicate all languages used anywhere in the message for localised name elements through http Content-Language response header (languages of the intended audience).**  
Note: For multi-language metadata attribute values, all language versions are provided independently from the preferred language (see below).   

## Multi-valued metadata attributes

- Multiple values are separated by a special sub-field separation character, e.g. `;`.
- This sub-field separation character has to be defined as first character in the squared bracket term of the header field of the first column, e.g. `METADATAFLOW[;]`.
- Such metadata attributes are indicated by having its ID followed by empty squared brackets "[]", e.g. `ATTR4[1][]`.
- For coded multi-valued metadata attributes, if option `labels=both` (see *[here](#optional-parameters)*) then each individual value is to be prefixed with its ID and the term ": ", e.g. `A: Value A;B: Value B`.

## Non-coded multi-lingual metadata attributes

- Non-coded metadata attributes allow for multi-lingual values. Those values are separated by a special sub-field separation character, e.g. `;`.
- This sub-field separation character has to be defined as first character in the squared bracket term of the header field of the first column, e.g. `METADATAFLOW[;]`.
- Such metadata attributes are indicated by having its ID followed by the list of possible 2-letter ISO language codes separated by the sub-field separator and encapsulated squared brackets "[]", e.g. `ATTR2[1][en;fr]`.
- Each individual language value is to be prefixed with its 2-letter ISO language code and a colon character ":", e.g. `en:Value;fr:Valeur`. Thus, in distinction to the ID prefix for coded values when using the HTTP accept header `labels=both` (see *[here](#optional-parameters)*), the language prefix `xx:` doesn't have an extra space character.

## Non-coded multi-lingual multi-valued components

- When non-coded multi-lingual metadata attributes have multiple values, then all individual values are separated by a special sub-field separation character, e.g. `;`.
- This sub-field separation character has to be defined as first character in the squared bracket term of the header field of the first column, e.g. `METADATAFLOW[;]`.
- Such metadata attributes are also indicated by having its ID followed by the list of possible language codes separated by the sub-field separator and encapsulated squared brackets "[]", e.g. `ATTR2[en;fr;de]`.
- Each individual language value is to prefixed with its 2-letter ISO language code and a colon character ":", e.g. `en:Value1`.
- Not each value needs all language versions. In order to allow knowing to which value the different language items belong, each multi-lingual value set is to be encapsulated in double quotes, e.g. `"en:Value1;fr:Valeur1";"en:Value2;de:Wert2"`. However, mote that fields with quotes must themselves be encapsulated in double quotes and that the inner quotes need to be doubled, thus the fully complete example is `"""en:Value1;fr:Valeur1"";""en:Value2;de:Wert2"""`.

## Non-coded XHTML-valued components

- Some non-coded metadata attributes allow for XHTML values.
- Each XHTML value is to be encapsulated in double quotes, e.g. `"<p>This is some ""metadata html""</p>"`. Remember that the inner quotes need to be doubled.
- The CSV format allows fields to contain line breaks if those fields are enclosed in double quotes. Thus XHTML values can also contain line breaks.

# Optional parameter

The following optional parameters can be added to the HTTP Accept header. It needs to be separated by the character combination `"; "`.
- labels (id|both; default=id): This parameter applies to all Nameable SDMX Artefacts contained in the header and the body of the message: 
  - If the parameter value is `id` then only the id of the Artefacts is displayed.
  - If the parameter value is `both` then the concatenated id and localised name of the Artefacts (see the section on [localised names](#localised-names) on how the message deals with languages) separated by `": "` are displayed. Note that the character combination `": "` could also be part of the Artefact name and could therefore occur several times within the concatenated string.

Support of the above non-default parameter is not required by implementers.

# Examples

Note: All examples assume the minimal HTTP Accept header: `application/vnd.sdmx.metadata+csv; version=1.0.0`

#### 1) Ordinary case

	METADATAFLOW,METADATASET,TARGET_TYPE,TARGET_ID,ATTRIBUTE_ID,ATTRIBUTE_VALUE
	OECD:MDF(1.0.0),OECD:MDS(1.0.0),Dataflow,OECD:DF(1.0.0),ATTRIBUTE_1[1],CODE_ID
	OECD:MDF(1.0.0),OECD:MDS(1.0.0),Dataflow,OECD:DF(1.0.0),ATTRIBUTE_1[1].ATTRIBUTE_1_2[1],"<p>An XHTML text</p>"
	OECD:MDF(1.0.0),OECD:MDS(1.0.0),Dataflow,OECD:DF(1.0.0),ATTRIBUTE_1[1].ATTRIBUTE_1_2[2],"<p>Another XHTML text</p>"
	OECD:MDF(1.0.0),OECD:MDS(1.0.0),Dataflow,OECD:DF(1.0.0),ATTRIBUTE_2[2],"Text with ""quotes"""
	OECD:MDF(1.0.0),OECD:MDS(1.0.0),Dataflow,OECD:DF(1.0.0),ATTRIBUTE_2[3],123

Note: The following default parameter settings are automatically applied:
- labels=id

#### 2) Metadata attribute with multiple values, metadata attributes with multi-lingual values: Variant A

	METADATAFLOW[;],METADATASET,TARGET_TYPE,TARGET_ID,ATTRIBUTE_ID,ATTRIBUTE_VALUE
	OECD:MDF(1.0.0),OECD:MDS(1.0.0),Dataflow,OECD:DF(1.0.0),ATTRIBUTE_1[1],CODE_ID
	OECD:MDF(1.0.0),OECD:MDS(1.0.0),Dataflow,OECD:DF(1.0.0),ATTRIBUTE_1[1].ATTRIBUTE_1_2[1][en;fr],"en:""<p>An XHTML text</p>"";fr:""<p>Un texte XHTML</p>"""
	OECD:MDF(1.0.0),OECD:MDS(1.0.0),Dataflow,OECD:DF(1.0.0),ATTRIBUTE_1[1].ATTRIBUTE_1_2[2][en;fr],"en:""<p>Another XHTML text</p>"";fr:""<p>Un autre texte XHTML</p>"""
	OECD:MDF(1.0.0),OECD:MDS(1.0.0),Dataflow,OECD:DF(1.0.0),ATTRIBUTE_2[2],"Text with ""quotes"""
	OECD:MDF(1.0.0),OECD:MDS(1.0.0),Dataflow,OECD:DF(1.0.0),ATTRIBUTE_2[3][],Text 1;Text 2;Text 3

#### 3) Condensed format: Variant B

	METADATAFLOW[;OECD:MDF(1.0.0)],METADATASET[OECD:MDS(1.0.0)],TARGET_TYPE[Dataflow],TARGET_ID[OECD:DF(1.0.0)],ATTRIBUTE_ID,ATTRIBUTE_VALUE
	,,,,ATTRIBUTE_1[1],CODE_ID
	,,,,ATTRIBUTE_1[1].ATTRIBUTE_1_2[1],"<p>An XHTML text</p>"
	,,,,ATTRIBUTE_1[1].ATTRIBUTE_1_2[2],"<p>Another XHTML text</p>"
	,,,,ATTRIBUTE_2[2],"Text with ""quotes"""
	,,,,ATTRIBUTE_2[3],123

#### 4) Metadata attributes with multiple values, metadata attributes with multi-lingual values, condensed format: Variant B

	METADATAFLOW[;OECD:MDF(1.0.0)],METADATASET[OECD:MDS(1.0.0)],TARGET_TYPE[Dataflow],TARGET_ID[OECD:DF(1.0.0)],ATTRIBUTE_ID,ATTRIBUTE_VALUE
	,,,,ATTRIBUTE_1[1],CODE_ID
	,,,,ATTRIBUTE_1[1].ATTRIBUTE_1_2[1][en;fr],"en:""<p>An XHTML text</p>"";fr:""<p>Un texte XHTML</p>"""
	,,,,ATTRIBUTE_1[1].ATTRIBUTE_1_2[2][en;fr],"en:""<p>Another XHTML text</p>"";fr:""<p>Un autre texte XHTML</p>"""
	,,,,ATTRIBUTE_2[2][],Text 1;Text 2;Text 3
	,,,,ATTRIBUTE_2[3][],123;456
  
#### 5) Localisation: HTTP Accept header: `application/vnd.sdmx.metadata+csv; version=1.0.0; labels=both`, HTTP Accept-Language header: `fr-FR, en;q=0.7`, metadata attribute with multiple values, metadata attributes with multi-lingual values, Variant C

	METADATAFLOW[|OECD:MDF(1.0.0): Metadata flow d'exemple];METADATASET[OECD:MDS(1.0.0): Metadataset d'exemple];TARGET_TYPE[Dataflow];TARGET_ID[OECD:DF(1.0.0): Dataflow name d'exemple];ATTRIBUTE_ID;ATTRIBUTE_VALUE
	;;;;ATTRIBUTE_1[1];CODE_ID
	;;;;ATTRIBUTE_1[1].ATTRIBUTE_1_2[1][en;fr];"en:""<p>An XHTML text</p>""|fr:""<p>Un texte XHTML</p>"""
	;;;;ATTRIBUTE_1[1].ATTRIBUTE_1_2[2][en;fr];"en:""<p>Another XHTML text</p>""|fr:""<p>Un autre texte XHTML</p>"""
	;;;;ATTRIBUTE_2[2];"Text with ""quotes"""
	;;;;ATTRIBUTE_2[3][];123|456

Note that in this example the client prefers French (fr) language with the France (FR) locale, but will also accept any type of English. Therefore, in the message the French language with the France locale is applied, transforming also the field separator from comma (,) to semicolon (;), and the decimal separator from dot (.) to comma (,).

#### 6) Localisation: HTTP Accept header: `application/vnd.sdmx.metadata+csv; version=1.0.0; labels=both`, HTTP Accept-Language header: `fr-FR, en;q=0.7`, metadata attribute with multiple values, metadata attributes with multi-lingual values, different metadata set and targets, Variant C

	METADATAFLOW[|OECD:MDF(1.0.0): Metadata flow d'exemple];METADATASET;TARGET_TYPE;TARGET_ID;ATTRIBUTE_ID;ATTRIBUTE_VALUE
	;OECD:MDS(1.0.0): Metadataset d'exemple;Dataflow;OECD:DF(1.0.0): Dataflow name d'exemple;ATTRIBUTE_1[1];CODE_ID
	;OECD:MDS(1.0.0): Metadataset d'exemple;Dataflow;OECD:DF(1.0.0): Dataflow name d'exemple;ATTRIBUTE_1[1].ATTRIBUTE_1_2[1][en;fr];"en:""<p>An XHTML text</p>""|fr:""<p>Un texte XHTML</p>"""
	;OECD:MDS(1.0.0): Metadataset d'exemple;Dataflow;OECD:DF(1.0.0): Dataflow name d'exemple;ATTRIBUTE_1[1].ATTRIBUTE_1_2[2][en;fr];"en:""<p>Another XHTML text</p>""|fr:""<p>Un autre texte XHTML</p>"""
	;OECD:MDS(1.0.0): Metadataset d'exemple;Dataflow;OECD:DF(1.0.0): Dataflow name d'exemple;ATTRIBUTE_2[2];"Text with ""quotes"""
	;OECD:MDS(1.0.0): Metadataset d'exemple;Dataflow;OECD:DF(1.0.0): Dataflow name d'exemple;ATTRIBUTE_2[3][];123|456
	;OECD:MDS(1.1.0): Nouveau metadataset d'exemple;Dataflow;OECD:DF(1.1.0): Nouveau dataflow name d'exemple;ATTRIBUTE_1[1];CODE_ID

#### 7) Non-coded multi-lingual multi-valued metadata attributes, varying metadataflows, Variant A

	METADATAFLOW[;],METADATASET,TARGET_TYPE,TARGET_ID,ATTRIBUTE_ID,ATTRIBUTE_VALUE
	OECD:MDF(1.0.0),OECD:MDS(1.0.0),Dataflow,OECD:DF(1.0.0),ATTRIBUTE_1[1][en;fr;de],"""en:Value1;fr:Valeur1"";""en:Value2;de:Wert2"""
	OECD:MDF(1.1.0);OECD:MDS(1.1.0);Dataflow;OECD:DF(1.1.0);ATTRIBUTE_1[1][en;fr;de],"""en:Value1;fr:Valeur1"";""en:Value2;de:Wert2"""

#### 8) Metadata for a non-versioned(1) data provision agreement, Variant B

	METADATAPROVISION[;OECD:DF],METADATASET,TARGET_TYPE,TARGET_ID,ATTRIBUTE_ID,ATTRIBUTE_VALUE
	,OECD:MDS(1.0.0),Dataflow,OECD:DF(1.0.0),ATTRIBUTE_1[1],CODE_ID

#### 9) Non-coded XHTML-formatted metadata attribute values with line-breaks 

	METADATAFLOW[;OECD:MDF(1.0.0)],METADATASET[OECD:MDS(1.0.0)],TARGET_TYPE[Dataflow],TARGET_ID[OECD:DF(1.0.0)],ATTRIBUTE_ID,ATTRIBUTE_VALUE
	,,,,ATTRIBUTE_1[1],"<p>This is some ""xhtml"" with a line
	break</p>"
	,,,,ATTRIBUTE_1[2],"<p>This is some other ""xhtml""</p>"

------------------------

**(1)** Note that since SDMX 3.0.0 the syntax *AGENCY:ARTEFACT_ID(VERSION)* allows omitting the version for non-versioned artefacts. In this case using *AGENCY:ARTEFACT_ID* is sufficient, e.g. `OECD:DF`