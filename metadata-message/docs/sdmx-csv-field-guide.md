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

# Design principles for SDMX-CSV 2.0 Metadata Messages (aligned with SDMX 3.0.0)

- In order to ensure the identifiability of the metadata contained in the message, the header row containing the column headers is mandatory and its content is well-defined. 
- An SDMX-CSV referential metadata message contains metadata attribute values for one or more metadatasets reported for one or more metadataflows or metadata provision agreements.
- After the mandatory header row, each row contains the information related to one specific metadataset attached to one or more identifiable artefacts (targets). 
- In [RFC 4180](https://tools.ietf.org/html/rfc4180), csv stands for "comma-separated values". However, while SDMX-CSV uses indeed the "comma" (%x2C) as the default field separator, it adopts the wider interpretation of csv as "character-separated values". It is recommended for implementers to provide SDMX-CSV messages according to the locale of the user (e.g. as indicated in the http Accept-Language header). It means that e.g. the semi-colon ‘;’ (as used typically in specific regions or countries) is acceptable as separator. See also the examples below. Note that the separator used in a message can be determined by retrieving the character that follows the header field of the first column which extended by a squared bracket term (see below).

## Columns

- The first column is always used for the underlying type of structure by which the metadataset is defined: metadataflow or metadata provision agreement.
- The next one or two columns are always used for the related structure identification.
- The next one or two columns are used for the metadataset identification.
- The next column is used for the action to be performed for the metadataset.
- The next column is used for the structure types of all targets of the metadataset.
- The next one or two columns are used for the identification of all targets of the metadataset.
- Each metadata attribute of the included metadataset(s) is represented in one or two columns. SDMX web services should return the columns in the metadata attribute order as defined in (each of) the underlying Metadata Structure Definition(s), thus in case of data defined by different metadata structures: first the metadata attributes of the first metadata structure, then the remaining metadata attributes of the second metadata structure and so forth. However, any order of these columns is valid for metadata uploads to SDMX-consuming systems.
- Implementers have the possibility to add any other custom columns as required, e.g. publicationPeriod, publicationYear, reportingBegin, reportingEnd, prepared, etc.
- In the context of appending or deleting metadata, certain columns may be omitted, see below.

## Column headers (first row)

- The header field of the first column always starts with the term `MDSTRUCTURE`.
  - This field must be extended with a sub-field delimiter encapsulated in squared brackets "[]", e.g. `MDSTRUCTURE[;]`, in case the message contains metadatasets with multiple targets or with multi-instance or multi-language metadata attributes.
- The header field of the second column always contains the term `MDSTRUCTURE_ID`.
- If option `labels=name` (see *[here](#optional-parameters)*): An additional column is added right after the structure identification column containing the term `MDSTRUCTURE_NAME`.
- The header field of the next column always contains the term `METADATASET_ID`.
- If option `labels=name` (see *[here](#optional-parameters)*): An additional column is added right after the metadataset identification column containing the term `METADATASET_NAME`.
- The header field of the next column should contain the term `ACTION`. For convenience, if this column is not present, a default action ("Information") is assumed for the whole message.
- The header field of the next column contains the term `TARGET_TYPES`.
- The header field of the next column contains the term `TARGET_IDS`.
- If option `labels=name` (see *[here](#optional-parameters)*): An additional column is added right after the target identification column containing the term `TARGET_NAMES`.
- The other columns for components contain:
  - Default: The ID of the metadata attribute reported in that column prefixed by all corresponding nested parent metadata attributes separated by a dot "." in the form *METADATA_ID[.METADATA_ID]+*, e.g. `ATTRIBUTE_GRANDPARENT_ID.ATTRIBUTE_PARENT_ID.ATTRIBUTE_CHILD_ID`. Additional pairs of squared brackets `[]` are added at the end of the IDs of those metadata attributes that have multiple instances, e.g. `CONTACT[].NAME`, `CONTACT[].PHONE[]` or `CONTACT.PHONE[]`, and/or that contain localised values. In the latter case the brackets encapsulate the ISO 2-letter language codes that can be encountered in that column, separated by the special sub-field separation character, e.g. `;`, defined in the squared bracket term of the header field of the first column, e.g. `MDSTRUCTURE[;]`. Example of a localised child attribute: `PROCESS.STEP[en;fr]`, and for multiple instances: `PROCESS.STEP[][en;fr]`.
  - If option `labels=both` (see *[here](#optional-parameters)*): The full ID (as described above under 'Default') and the localised name of the metadata attribute reported in that column separated by the term ": ", e.g. `ATTRIBUTE_ID: ATTRIBUTE_NAME.
  - If option `labels=name` (see *[here](#optional-parameters)*): An additional column is added right after the metadata attribute identification column containing the localised name of the metadata attribute reported in the previous column.
- Any other custom column contains a custom but unique term, e.g. `publicationPeriod`.

## Column content (all rows after header)

- The first column contains: `metadataflow` or `metadataprovision`, depending on type of artefact for which the metadata contained in the message are defined: metadataflow or metadata provision agreement.
- The second column contains:
  - Default: The structure identification information in the form *AGENCY:ARTEFACT_ID(VERSION)* (1), e.g. `ESTAT:MDF(1.6.0)`.
  - If option `labels=both` (see *[here](#optional-parameters)*): The structure identification information and its localised name separated by the term ": ", e.g. `ESTAT:MDF(1.6.0): Metadataflow name`.
- If option `labels=name` (see *[here](#optional-parameters)*): An additional column is added right after the structure identification column with the structure's localised name, e.g. `Metadataflow name`.
- The next column contains the metadataset identification information in the form *AGENCY:ARTEFACT_ID(VERSION)*(1), e.g. `AGENCY:MD_SET(1.0.0)`.
  - If option `labels=both` (see *[here](#optional-parameters)*): The ID and the localised name of the metadataset separated by the term ": ", e.g. `ESTAT:MD_SET(1.0.0): Metadataset 1`.
- If option `labels=name` (see *[here](#optional-parameters)*): An additional column is added right after the metadataset identification column with the metadataset's localised name, e.g. `Metadata set name`.
- The next column contains one character representing one of the current 4 action types:
  - "I": Information - Metadata is for information purposes. If such metadata messages are loaded into an SDMX database, the action "A" (Append) is assumed.
  - "A": Append -  Metadata is intended for an incremental update of existing metadatasets or the provision of new metadatasets formerly absent. This means that only the information provided explicitly in the message should be altered. Any metadata attribute value that is to be added or changed must be provided. However, the absence of a metadata attribute value at any metadata nesting level does not imply deletion; instead it is simply implied that the value is to remain unchanged. Therefore, it is valid and acceptable to send a metadata message with an action of 'Append' which (in addition to the required identification columns) contains only the column and value of a parent metadata attribute. In this case, only that value will be updated. The values of the child metadata attributes are not changed or deleted. Note that it is not permissible to update metadata attributes using incomplete identification information, e.g. without the metadataset or without the target identifier. In order to update a metadata attribute, the full identification information (all identification columns listed here) must always be provided. According to the SDMX 3.0 semantic versioning rules, it is not possible to update a semantically versioned metadataset.
  - "R": Replace - Existing metadatasets are to be fully replaced. Metadatasets formally absent will be added. According to the SDMX 3.0 semantic versioning rules, it is not possible to replace a semantically versioned metadataset. 
  - "D": Delete - Metadata is to be deleted. 'Delete' is assumed to be an incremental deletion. The deletion is to take place at the lowest level of detail provided in the row. Concretely, if a 'Delete' row only contains the identification information of the structural artefact (metadataflow or metadata provision agreement) without a metadataset then all metadatasets for the given artefact will be deleted. If a 'Delete' row only contains up to the metadataset identification without metadata attributes then the given metadataset is to be deleted. Finally, if a row contains all complete identification information up to a non-versioned metadataset and some values for metadata attributes, then only the metadata attributes with values will be deleted from the non-versioned metadataset. To be deleted attribute values must be non-empty, e.g. marked with the dash character "-". According to the SDMX 3.0 semantic versioning rules, it is not possible to modify a semantically versioned metadataset.
  - For convenience, if this column is absent then the action "Information" is assumed.
- The next column contains the types of all the targets of the metadataset according to the resource names defined for Structural Metadata Queries, e.g. `dataflow`. Multiple targets are separated by the special sub-field separation character, e.g. `;`, defined in the squared bracket term of the header field of the first column, e.g. `MDSTRUCTURE[;]`. Example for multiple target types: `dataflow;dataflow`.
- The next column contains the identification information of all the targets of the metadataset in the form *AGENCY:ARTEFACT_ID(VERSION)* (1), separated by the sub-field separation character, e.g. `AGENCY:DF1(1.0.0);AGENCY:DF2(1.0.0)`.
  - If option `labels=both` (see *[here](#optional-parameters)*): The column contains the ID and the localised name of the targets separated by the term ": ", e.g. `AGENCY:DF(1.0.0): Dataflow name` or `AGENCY:DF1(1.0.0): Dataflow 1 name;AGENCY:DF2(1.0.0): Dataflow 2 name`.
- If option `labels=name` (see *[here](#optional-parameters)*): An additional column is added right after the target identification column with the target's localised name, e.g. `Dataflow name` or `Dataflow 1 name;Dataflow 2 name`.
- The other columns for metadata attributes contain:
  - Default: The ID(s) (if coded) or value(s) (if non-coded) for the metadata attribute reported in that column, e.g. `A`, `A;B` or `"<p>An XHTML text</p>"`.
  - If option `labels=both` (see *[here](#optional-parameters)*): The ID(s) and their localised name(s) for the metadata attribute separated by the term ": " (if coded) or the value(s) (if non-coded) for the metadata attribute reported in that column, e.g. `A: A value name`, `A: A value name;B: B value name` or `"<p>An XHTML text</p>"`.
  - If option `labels=name` (see *[here](#optional-parameters)*): An additional column is added right after the metadata attribute identification column containing the localised name, e.g. `A value name` or `A value name;B value name`, of the metadata attribute value reported in the previous column. It is empty if the value has no localised name.
  -  All string/textual values (complete string between column-separating characters including ID's or language codes) should always be encapsulated in quotation marks, they must be if they contain commas or inner quotation marks. Quotation marks in strings/textual values must always be escaped by doubling the quotes.
  - When metadata from different metadata structures are present then the columns not related to the attribute's metadata structure are to be left empty.
- The other custom columns contain any potentially localised custom content.

## Localisation 

- HTTP content negotiation, see [RFC 2616 - HTTP 1.1 Header Field Definitions](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html)
  - Always use this mime-type in the Accept header: `application/vnd.sdmx.metadata+csv; version=2.0.0`.
  - The client can indicate preferred languages through the Accept-Language header, e.g. `fr, en-gb;q=0.8, en;q=0.7`.
- Always localise all artefact names according to the preferred language. The first best language match according to the user’s preferred language choices in the http Accept-Language header (or if that is not available than according to the system's default language order) is to be used for each localisable name element. The message does however not indicate the returned language per localisable name element. In case that there is no such language match for a particular localisable name element, it is optional to return the element in a system-default language or alternatively to not return the name element.
**It is recommended to indicate all languages used anywhere in the message for localised name elements through http Content-Language response header (languages of the intended audience).**  
Note: For multi-language metadata attribute values, all language versions are provided independently from the preferred language (see below).   

## Multi-instance metadata attributes

- Values from multiple instances of a metadata attribute within a metadataset are separated by the special sub-field separation character, e.g. `;`, defined in the squared bracket term of the header field of the first column, e.g. `MDSTRUCTURE[;]`.
- Such metadata attributes are indicated in the column header by having their ID followed by empty squared brackets "[]", e.g. `ATTR[]`.
- For coded multi-instance metadata attributes, if option `labels=both` (see *[here](#optional-parameters)*) then each individual value is to be prefixed with its ID and the term ": ", e.g. `A: Value A;B: Value B`.

## Non-coded multi-lingual metadata attributes

- Non-coded metadata attributes allow for multi-lingual values. Those values are separated by the special sub-field separation character, e.g. `;`, defined in the squared bracket term of the header field of the first column, e.g. `MDSTRUCTURE[;]`.
- Such metadata attributes are indicated in the column header by having their ID followed by the list of possible 2-letter ISO language codes separated by the sub-field separator and encapsulated squared brackets "[]", e.g. `ATTR[en;fr]`.
- Each individual language value is to be prefixed with its 2-letter ISO language code and a colon character ":", e.g. `en:Value;fr:Valeur`. Thus, in distinction to the ID prefix for coded values when using the HTTP accept header `labels=both` (see *[here](#optional-parameters)*), the language prefix `xx:` doesn't have an extra space character.

## Non-coded multi-lingual multi-instance metadata attributes

- When non-coded multi-lingual metadata attributes have multiple instances within a metadataset, then all individual values are included and separated by the special sub-field separation character, e.g. `;`, defined in the squared bracket term of the header field of the first column, e.g. `MDSTRUCTURE[;]`.
- Such metadata attributes are indicated in the column header by having their ID followed by squared brackets "[]" as well as by the list of possible language codes separated by the sub-field separator and encapsulated in additional squared brackets "[]", e.g. `ATTR[][en;fr;de]`.
- Each individual language value is to prefixed with its 2-letter ISO language code and a colon character ":", e.g. `en:Value1`.
- Not each value needs all language versions. In order to allow knowing to which value the different language items belong, each multi-lingual value set is to be encapsulated in double quotes, e.g. `"en:Value1;fr:Valeur1";"en:Value2;de:Wert2"`. However, note that fields with double quotes must themselves be encapsulated in double quotes and that the inner double quotes need to be doubled, thus the fully complete example is `"""en:Value1;fr:Valeur1"";""en:Value2;de:Wert2"""`.

## Non-coded XHTML-valued components

- Some non-coded metadata attributes allow for XHTML values.
- Each XHTML value is to be encapsulated in double quotes, e.g. `"<p>This is some ""metadata html""</p>"`. Remember that the inner double quotes need to be doubled.
- The CSV format allows fields to contain line break characters if those fields are enclosed in double quotes. Thus XHTML values can also contain line breaks, although HTML viewers will ignore them.

# Optional parameters

The following optional parameter can be added to the HTTP Accept header. It needs to be separated by the character combination `"; "`.
- labels (id|name|both; default=id): This parameter applies to all Nameable SDMX Artefacts contained in the header and the body of the message: 
  - If the parameter value is `id` then only the id of the Artefacts is displayed.
  - If the parameter value is `both` then the concatenated id and localised name of the Artefacts (see the section on [localised names](#localised-names) on how the message deals with languages) separated by `": "` are displayed. Note that the character combination `": "` could also be part of the Artefact name and could therefore occur several times within the concatenated string.
  - If the parameter value is `name` then the id/value and the name of the artefacts are displayed in separate columns (see *[here](#columns)*), the ID/value column always directly preceding its related localised name column.

# Examples

Note: All examples assume the minimal HTTP Accept header: `application/vnd.sdmx.metadata+csv; version=1.0.0`

#### 1) Ordinary case

	MDSTRUCTURE,MDSTRUCTURE_ID,METADATASET_ID,ACTION,TARGET_TYPES,TARGET_IDS,ATTRIBUTE_1,ATTRIBUTE_1.CHILD,ATTRIBUTE_2
	metadataflow,OECD:MDF(1.0.0),OECD:MDS(1.0.0),I,dataflow,OECD:DF(1.0.0),A STRING VALUE,"<p>An XHTML text with ""quotes""</p>",123

Note:  
The following default parameter settings are automatically applied:
- labels=id

#### 2) Metadata attribute with multiple instances and multi-lingual values

	MDSTRUCTURE[;],MDSTRUCTURE_ID,METADATASET_ID,ACTION,TARGET_TYPES,TARGET_IDS,ATTRIBUTE_1,ATTRIBUTE_1.ATTRIBUTE_1_2[][en;fr],ATTRIBUTE_2[],ATTRIBUTE_3[]
	metadataflow,OECD:MDF(1.0.0),OECD:MDS(1.0.0),I,dataflow,OECD:DF(1.0.0),CODE_ID,"""en:""""<p>An XHTML text</p>"""";fr:""""<p>Un texte XHTML</p>""""";""en:""""<p>Another XHTML text</p>"""";fr:""""<p>Un autre texte XHTML</p>""""""","""Text with """"quotes"""""";""Another text""",123;456

#### 3) Localisation: HTTP Accept header: `application/vnd.sdmx.metadata+csv; version=1.0.0; labels=both`, HTTP Accept-Language header: `fr-FR, en;q=0.7`, metadata attribute with multiple instances, metadata attributes with multi-lingual values

	MDSTRUCTURE[|],MDSTRUCTURE_ID;METADATASET_ID;ACTION;TARGET_TYPES;TARGET_IDS;ATTRIBUTE_1: Attribut d'exemple 1;ATTRIBUTE_1.ATTRIBUTE_1_2[][en|fr]: Attribut d'exemple 12;ATTRIBUTE_2[]: Attribut d'exemple 2
	metadataflow;OECD:MDF(1.0.0): Metadataflow d'exemple;OECD:MDS(1.0.0): Metadataset d'exemple;I;dataflow;OECD:DF(1.0.0): Dataflow d'exemple;CODE_ID: Nom du code;"""en:""""<p>An XHTML text</p>""""|fr:""""<p>Un texte XHTML</p>""""""|""en:""""<p>Another XHTML text</p>""""|fr:""""<p>Un autre texte XHTML</p>""""""";123,45|6,789

Note that in this example the client prefers French (fr) language with the France (FR) locale, but will also accept any type of English. Therefore, in the message the French language with the France locale is applied, transforming also the field separator from comma (,) to semicolon (;), and the decimal separator from dot (.) to comma (,).

#### 4) Localisation: HTTP Accept header: `application/vnd.sdmx.metadata+csv; version=1.0.0; labels=name`, HTTP Accept-Language header: `en-US`, metadata attribute with multiple instances, metadata attributes with multi-lingual values, different targets and metadatasets

	MDSTRUCTURE[;],MDSTRUCTURE_ID,MDSTRUCTURE_NAME,METADATASET_ID,METADATASET_NAME,ACTION,TARGET_TYPES,TARGET_IDS,TARGET_NAMES,ATTRIBUTE_1,Attribute 1,ATTRIBUTE_1.ATTRIBUTE_1_2[][en|fr],Attribute 12,ATTRIBUTE_2[],Attribute 2
	metadataflow,OECD:MDF(1.0.0),Metadataflow name,OECD:MDS(1.0.0),Metadataset name,I,dataflow;dataflow,OECD:DF(1.0.0);OECD:DF(1.1.0),Dataflow name 1;Dataflow name 2,CODE_ID,Code name,"""en:""""<p>An XHTML text</p>"""";fr:""""<p>Un texte XHTML</p>"""""";""en:""""<p>Another XHTML text</p>"""";fr:""""<p>Un autre texte XHTML</p>""""""",123.45;6.789
	metadataflow,OECD:MDF(1.0.0),Metadataflow name,OECD:MDS(1.1.0),Metadataset new name,I,codelist,OECD:CL(1.0.0),Codelist name,CODE_ID,Code name,"""en:""""<p>Text 1</p>"""";fr:""""<p>Texte 1</p>"""""";""en:""""<p>Text 2</p>"""";fr:""""<p>Texte 2</p>""""""",0

#### 5) Varying metadataflows

	MDSTRUCTURE[;],MDSTRUCTURE_ID,METADATASET_ID,ACTION,TARGET_TYPES,TARGET_IDS,ATTRIBUTE_1,ATTRIBUTE_2[][en;fr;de]
	metadataflow,OECD:MDF(1.0.0),OECD:MDS(1.0.0),I,dataflow,OECD:DF(1.0.0),CODE_ID,"""en:Value1;fr:Valeur1"";""en:Value2;de:Wert2"""
	metadataflow,OECD:MDF(1.1.0),OECD:MDS(1.1.0),I,dataflow,OECD:DF(1.1.0),CODE_ID,"""en:Value1;fr:Valeur1"";""en:Value2;de:Wert2"""

#### 6) Non-versioned metadataset for a non-versioned(1) data provision agreement

	MDSTRUCTURE[;],MDSTRUCTURE_ID,METADATASET_ID,ACTION,TARGET_TYPES,TARGET_IDS,ATTRIBUTE_1,ATTRIBUTE_2[en;fr]
	metadataprovision,OECD:MDP,OECD:MDS,I,dataflow,OECD:DF(1.0.0),CODE_ID,"en:Value1;fr:Valeur1"

#### 7) Non-coded metadata attribute values with line-breaks 

	MDSTRUCTURE[;],MDSTRUCTURE_ID,METADATASET_ID,ACTION,TARGET_TYPES,TARGET_IDS,ATTRIBUTE_1[]
	metadataflow,OECD:MDF(1.0.0),OECD:MDS(1.0.0),I,dataflow,OECD:DF(1.0.0),"""This text with a line
	break"";""This is some other text</p>"""

#### 8) Deleting all values of specific metadata attributes from a non-versioned metadatset: the values that correspond to the attribute identifiers are deleted

	MDSTRUCTURE[;],MDSTRUCTURE_ID,METADATASET_ID,ACTION,TARGET_TYPES,TARGET_IDS,ATTRIBUTE_1[],ATTRIBUTE_1[].ATTRIBUTE_1_2[],ATTRIBUTE_2
	metadataflow,OECD:MDF(1.0.0),OECD:MDS,D,dataflow,OECD:DF(1.0.0),-,-,-

#### 9) Deleting a whole metadataset:

	MDSTRUCTURE[;],MDSTRUCTURE_ID,METADATASET_ID,ACTION
	metadataflow,OECD:MDF(1.0.0),OECD:MDS(1.0.0),D
	metadataflow,OECD:MDF(1.1.0),OECD:MDS(1.1.0),D

#### 10) Deleting all metadatasets defined by a specific metadata artefact:

	MDSTRUCTURE[;],MDSTRUCTURE_ID,METADATASET_ID,ACTION
	metadataflow,OECD:MDF(1.0.0),,D
	metadataflow,OECD:MDF(1.1.0),,D

------------------------

**(1)** Note that since SDMX 3.0.0 the syntax *AGENCY:ARTEFACT_ID(VERSION)* allows omitting the version for non-versioned artefacts. In this case using *AGENCY:ARTEFACT_ID* is sufficient, e.g. `OECD:MDP`
