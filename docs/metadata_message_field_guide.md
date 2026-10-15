# Metadata Message Field Guide

## Introduction

SDMX-CSV Metadata Message is an SDMX reference metadata exchange format based
on [RFC 4180](https://tools.ietf.org/html/rfc4180). CSV is a widely used,
standardised and simple format to exchange data, supported by many tools.

SDMX-CSV integrates with other specifications:

- The SDMX RESTful API specification (e.g. content negotiation with a media
  type to get SDMX-CSV representations, specific formats for responses,
  language selection through HTTP content negotiation).
- The [RFC 4180](https://tools.ietf.org/html/rfc4180) specification.

### Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "MAY" and "OPTIONAL" in this document are to be
interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119)
and [RFC 8174](https://www.rfc-editor.org/rfc/rfc8174) when, and only when,
they appear in all capitals.

This guide distinguishes two roles:

- **Sender**: a system that writes an SDMX-CSV message, e.g. an SDMX web
  service returning a response, or a metadata provider preparing an upload.
- **Receiver**: a system that reads an SDMX-CSV message, e.g. a client tool
  consuming a response, or an SDMX storage system processing an upload.

### RFC 4180

SDMX-CSV follows the rules of [RFC 4180](https://tools.ietf.org/html/rfc4180),
which defines a common format and media type for CSV files. Implementers should
read the (very short) RFC; in a nutshell it defines:

- that all records MUST have the same number of fields as the header row;
- when double quotes are used and how they are escaped (by doubling them);
- that spaces are part of a field and MUST NOT be ignored;
- the media type and the default character set.

SDMX-CSV extends RFC 4180 in one respect: although "CSV" stands for
"comma-separated values" and the comma (`%x2C`) is the default field separator,
SDMX-CSV adopts the wider interpretation "character-separated values". See
[Separators](#separators).

## Message overview

An SDMX-CSV metadata message contains the metadata attribute values of one or
more metadatasets, reported for one or more metadataflows or metadata
provision agreements. It consists of one mandatory header row followed by any
number of rows, each containing **one metadataset** attached to **one or more
identifiable artefacts** (its targets).

```csv
MDSTRUCTURE,MDSTRUCTURE_ID,METADATASET_ID,TARGET_TYPES,TARGET_IDS,CONTACT
metadataflow,OECD:MDF(1.0.0),OECD:MDS(1.0.0),dataflow,OECD:DF(1.0.0),Statistics Directorate
```

### Column overview

| Column name | Description | Position |
| --- | --- | --- |
| **Fixed columns** | | |
| [`MDSTRUCTURE`](#mdstructure) | Type of structure by which the metadataset is defined: `metadataflow` or `metadataprovision`. The character directly after this term is the field separator. | 1 |
| [`MDSTRUCTURE[;]`](#mdstructure) | `MDSTRUCTURE` extended with the sub-field separator in square brackets. REQUIRED when the message contains metadatasets with multiple targets, or multi-instance or multi-lingual metadata attributes. | 1 |
| [`MDSTRUCTURE_ID`](#mdstructure_id) | Structure identification `AGENCY:ID(VERSION)`. | 2 |
| [`MDSTRUCTURE_NAME`](#mdstructure_name) | Localised name of the structure. Only with `labels=name`. | 3 (optional) |
| [`METADATASET_ID`](#metadataset_id) | Metadataset identification `AGENCY:ID(VERSION)`. | Directly after the structure identification: 3, or 4 with `labels=name` |
| [`METADATASET_NAME`](#metadataset_name) | Localised name of the metadataset. Only with `labels=name`. | Directly after `METADATASET_ID` (optional) |
| [`ACTION`](#action-deprecated) | Deprecated; ignored if present. | Directly after the metadataset identification (optional) |
| [`IS_PARTIAL_LANGUAGE`](#is_partial_language) | `1` if the metadataset contains only a subset of the available languages, `0` (default) otherwise. | Directly after `ACTION` if present, otherwise after the metadataset identification (optional) |
| [`TARGET_TYPES`](#target_types) | Structure types of all targets of the metadataset, e.g. `dataflow`. | Directly after the columns above |
| [`TARGET_IDS`](#target_ids) | Identification of all targets of the metadataset. | Directly after `TARGET_TYPES` |
| [`TARGET_NAMES`](#target_names) | Localised names of all targets. Only with `labels=name`. | Directly after `TARGET_IDS` (optional) |
| **Metadata attribute columns** | | |
| [`ATTR`](#metadata-attribute-columns) | Metadata attribute ID. | After the fixed columns |
| [`PARENT.CHILD`](#nested-metadata-attributes) | Nested metadata attribute, prefixed with the IDs of all its parent attributes. | — |
| [`ATTR: Name`](#metadata-attribute-columns) | ID and localised name in one header. Only with `labels=both`. | Replaces `ATTR` |
| [`Name`](#metadata-attribute-columns) | Additional column with localised names. Only with `labels=name`. | Directly after its metadata attribute column |
| **Other columns** | | |
| [`publicationPeriod`](#custom-columns) (any custom term) | Implementer-defined column. | Not specified |

Multi-instance and multi-lingual metadata attributes have no specific header
syntax: the header is the plain (dotted) ID, and the type of the metadata
attribute is defined by the metadata structure definition (see
[Metadata attribute values](#metadata-attribute-values)).

## Separators

### Field separator

- Senders SHOULD use the comma `,` as field separator, unless the user's locale
  (e.g. as indicated in the HTTP `Accept-Language` header) calls for another
  one, e.g. the semicolon `;` used in some regions or countries.
- The field separator of a message is the character directly following the
  fixed first header term `MDSTRUCTURE`, or, when that term is extended with a
  square-bracket term (see below), the character directly following `]`.
  Receivers MUST determine the field separator this way.

### Sub-field separator

- Multiple targets, multi-instance and multi-lingual metadata attribute values
  use a sub-field separator, e.g. `;`.
- When a message contains such values, the sender MUST declare the sub-field
  separator as the only character inside square brackets appended to the first
  header term, e.g. `MDSTRUCTURE[;]` or `MDSTRUCTURE[|]`.
- The sub-field separator MUST differ from the field separator.

## Columns

### Column order

1. `MDSTRUCTURE`
2. `MDSTRUCTURE_ID`
3. `MDSTRUCTURE_NAME` (only with `labels=name`)
4. `METADATASET_ID`
5. `METADATASET_NAME` (only with `labels=name`)
6. `ACTION` (deprecated, optional)
7. `IS_PARTIAL_LANGUAGE` (optional)
8. `TARGET_TYPES`
9. `TARGET_IDS`
10. `TARGET_NAMES` (only with `labels=name`)
11. Metadata attribute columns
12. Custom columns (any position)

The fixed columns (1–10) MUST appear in this order, without gaps between those
present.

Metadata attribute columns:

- SDMX web services SHOULD return metadata attribute columns in the order in
  which the metadata attributes are defined in the underlying metadata
  structure definition(s). When metadata from several metadata structures are
  present: first the metadata attributes of the first metadata structure, then
  the remaining metadata attributes of the second metadata structure, and so
  forth.
- Receivers processing uploads MUST accept metadata attribute columns in any
  order.

### `MDSTRUCTURE`

**Header:** `MDSTRUCTURE`, or `MDSTRUCTURE[x]` where `x` is the sub-field
separator (see [Sub-field separator](#sub-field-separator)).

**Content:** the type of artefact by which the row's metadataset is defined:

| Value | Artefact |
| --- | --- |
| `metadataflow` | Metadataflow |
| `metadataprovision` | Metadata provision agreement |

### `MDSTRUCTURE_ID`

**Header:** `MDSTRUCTURE_ID`.

**Content:**

- Default: the structure identification in the form
  _AGENCY:ARTEFACT_ID(VERSION)_, e.g. `ESTAT:MDF(1.6.0)`. For non-versioned
  artefacts the version is omitted, e.g. `OECD:MDP`[^nv].
- With `labels=both`: the structure identification and its localised name
  separated by `": "`, e.g. `ESTAT:MDF(1.6.0): Metadataflow name`.

### `MDSTRUCTURE_NAME`

**Header:** `MDSTRUCTURE_NAME`.

Present only with `labels=name`, directly after `MDSTRUCTURE_ID`.

**Content:** the structure's localised name, e.g. `Metadataflow name`.

### `METADATASET_ID`

**Header:** `METADATASET_ID`.

**Content:**

- Default: the metadataset identification in the form
  _AGENCY:ARTEFACT_ID(VERSION)_, e.g. `AGENCY:MD_SET(1.0.0)`. For
  non-versioned metadatasets the version is omitted[^nv].
- With `labels=both`: the metadataset identification and its localised name
  separated by `": "`, e.g. `ESTAT:MD_SET(1.0.0): Metadataset 1`.

### `METADATASET_NAME`

**Header:** `METADATASET_NAME`.

Present only with `labels=name`, directly after `METADATASET_ID`.

**Content:** the metadataset's localised name, e.g. `Metadata set name`.

### `ACTION` (deprecated)

**Header:** `ACTION`.

This column was previously used for the action to be performed with the
metadataset. It is deprecated:

- Senders SHOULD NOT include it.
- If it is present, receivers MUST ignore it.
- To add, replace or delete metadatasets, use the appropriate HTTP method when
  submitting the message to an SDMX REST API, as documented in
  [Maintaining reference metadata](https://github.com/sdmx-twg/sdmx-rest/blob/complement-maintenance-doc/doc/maintenance.md#maintaining-reference-metadata).

!!! warning "Open point"

    The link above points to a working branch of `sdmx-rest`
    (`complement-maintenance-doc`). It should be updated to the released
    documentation.

### `IS_PARTIAL_LANGUAGE`

**Header:** `IS_PARTIAL_LANGUAGE`.

This column is optional.

**Content:**

| Value | Meaning |
| --- | --- |
| `0` | The metadataset contains the complete set of available languages (default, also assumed when the column is absent) |
| `1` | The metadataset contains only a subset of the available languages |

A metadataset can contain only some of the available languages, e.g. when it
results from an SDMX REST GET query with an `Accept-Language` header. When a
metadataset marked `1` is submitted to an SDMX REST web service, the receiver
MUST only add or update the included languages and MUST NOT change the other
languages.

!!! warning "Open point"

    The guide does not state the effect of `0` on languages that exist in the
    target system but are absent from the submitted metadataset (e.g. whether
    they are deleted). To be clarified.

### `TARGET_TYPES`

**Header:** `TARGET_TYPES`.

**Content:** the structure types of all targets of the metadataset, using the
resource names defined for SDMX REST structural metadata queries, e.g.
`dataflow` or `codelist`. Multiple targets are separated by the sub-field
separator, e.g. `dataflow;dataflow`, in the same order as in
[`TARGET_IDS`](#target_ids).

### `TARGET_IDS`

**Header:** `TARGET_IDS`.

**Content:**

- Default: the identification of all targets of the metadataset in the form
  _AGENCY:ARTEFACT_ID(VERSION)_[^nv], separated by the sub-field separator,
  e.g. `AGENCY:DF1(1.0.0);AGENCY:DF2(1.0.0)`.
- With `labels=both`: each target identification and its localised name
  separated by `": "`, e.g. `AGENCY:DF(1.0.0): Dataflow name` or
  `AGENCY:DF1(1.0.0): Dataflow 1 name;AGENCY:DF2(1.0.0): Dataflow 2 name`.

### `TARGET_NAMES`

**Header:** `TARGET_NAMES`.

Present only with `labels=name`, directly after `TARGET_IDS`.

**Content:** the localised names of all targets, separated by the sub-field
separator, e.g. `Dataflow name` or `Dataflow 1 name;Dataflow 2 name`.

### Metadata attribute columns

Each metadata attribute of the included metadataset(s) is represented in one
column, or two columns with `labels=name`.

When a message contains metadata from different metadata structures, the
columns not related to the row's metadata structure MUST be left empty.

**Header:**

The full ID of a metadata attribute is its own ID prefixed by the IDs of all
its parent metadata attributes, separated by a dot `.`, in the form
_METADATA_ID[.METADATA_ID]+_, e.g.
`ATTRIBUTE_GRANDPARENT_ID.ATTRIBUTE_PARENT_ID.ATTRIBUTE_CHILD_ID`. No suffix
indicates whether a metadata attribute is multi-instance or multi-lingual;
see [Metadata attribute values](#metadata-attribute-values).

| `labels` | Header | Example |
| --- | --- | --- |
| `id` (default) | Full ID | `CONTACT.NAME` |
| `both` | Full ID and localised name separated by `": "` | `CONTACT.NAME: Contact name` |
| `name` | Full ID, followed by an additional column whose header is the localised name | `CONTACT.NAME`, `Contact name` |

**Content:**

| `labels` | Coded metadata attribute | Non-coded metadata attribute |
| --- | --- | --- |
| `id` (default) | Code ID(s), e.g. `A` or `A;B` | Value(s), e.g. `"<p>An XHTML text</p>"` |
| `both` | Code ID(s) and localised name(s) separated by `": "`, e.g. `A: A value name` or `A: A value name;B: B value name` | Value(s) |
| `name` | Code ID(s); the additional column contains the localised name(s), e.g. `A value name` or `A value name;B value name`, and is empty if the value has no localised name | Value(s); the additional column is empty |

Note that `": "` may also occur inside a name and may therefore occur several
times within a concatenated value.

### Custom columns

Senders MAY add any other custom columns, e.g. `publicationPeriod`,
`publicationYear`, `reportingBegin`, `reportingEnd` or `prepared`.

- The header MUST be a custom but unique term.
- The content MAY be localised.

## Metadata attribute values

### Quoting

- A field MUST be enclosed in double quotes if it contains the field
  separator, a double quote or a line break; double quotes inside the field
  MUST be escaped by doubling them (RFC 4180).
- All string and textual values (the complete field, including IDs or language
  codes) SHOULD be enclosed in double quotes, even when not required.

!!! warning "Open point"

    The recommendation to always quote textual values exists only in the
    metadata guide, and the examples do not follow it. To be decided whether it
    is kept, and if so, whether it also applies to data messages.

Whether a metadata attribute is multi-instance and/or multi-lingual is defined
only by the metadata structure definition; column headers carry no indication
of it. Receivers MUST use the metadata structure definition to interpret the
content of such columns, e.g. to know whether the sub-field separator splits
the field into several values.

### Multi-instance metadata attributes

- **Header:** the plain (dotted) metadata attribute ID, e.g. `ATTR`.
- **Content:** the values of all instances of the metadata attribute within
  the metadataset, separated by the sub-field separator, e.g. `A;B`.
- With `labels=both`, each coded value is prefixed with its ID and `": "`, e.g.
  `A: Value A;B: Value B`.
- Requires the sub-field separator to be declared (`MDSTRUCTURE[x]`).

### Non-coded multi-lingual metadata attributes

- **Header:** the plain (dotted) metadata attribute ID, e.g. `ATTR`.
- **Content:** each language value prefixed with its ISO 639-1 language code
  and a colon `:`, the values separated by the sub-field separator, e.g.
  `en:Value;fr:Valeur`. Unlike the ID prefix used with `labels=both`, there is
  no space after the colon.
- Requires the sub-field separator to be declared (`MDSTRUCTURE[x]`).

### Non-coded multi-lingual multi-instance metadata attributes

- **Header:** the plain (dotted) metadata attribute ID, e.g. `ATTR`.
- **Content:** all values of all instances, each language value prefixed with
  its language code and a colon, e.g. `en:Value1`.
- Not every instance needs all language versions. To show which language
  values belong to which instance, each instance's value set MUST be enclosed
  in double quotes, the sets separated by the sub-field separator, e.g.
  `"en:Value1;fr:Valeur1";"en:Value2;de:Wert2"`. Since the field itself then
  contains double quotes, it MUST be enclosed in double quotes and the inner
  quotes MUST be doubled:
  `"""en:Value1;fr:Valeur1"";""en:Value2;de:Wert2"""`.
- Requires the sub-field separator to be declared (`MDSTRUCTURE[x]`).

### Nested metadata attributes

- Each metadata attribute is presented in its own column, even when nested.
- **Header:** the full ID path, e.g. `CONTACT.NAME` or `PROCESS.STEP`,
  whether or not any attribute in the path is multi-instance or
  multi-lingual.

!!! warning "Open point"

    The guide does not describe how the values of a child of a multi-instance
    parent are arranged, e.g. how the values in `CONTACT.NAME` map to the
    `CONTACT` instances when `CONTACT` is multi-instance. The data message guide specifies positional values,
    with an empty sub-field for a parent instance without a value
    (`a;;c`) and nested double quotes for deeper levels. Proposal: adopt the
    same rule here.

### Non-coded XHTML values

- Some non-coded metadata attributes allow XHTML values.
- Each XHTML value MUST be enclosed in double quotes, with inner quotes
  doubled, e.g. `"<p>This is some ""metadata html""</p>"`.
- As allowed by RFC 4180 for quoted fields, XHTML values MAY contain line
  breaks, although HTML viewers ignore them.

## Rows

- Each row contains one metadataset, attached to one or more targets.
- A message MAY contain several metadatasets, for one or more metadataflows or
  metadata provision agreements, and based on one or more metadata structures.

!!! warning "Open point"

    The previous text stated that "in the context of appending or deleting
    metadata, certain columns may be omitted, see below", but no such rules
    followed. Since `ACTION` is deprecated in favour of HTTP methods, this
    statement has been removed. To be confirmed whether any column may be
    omitted in maintenance requests.

## HTTP integration

### Media type

Clients SHOULD request SDMX-CSV metadata messages with the following HTTP
`Accept` header:

`application/vnd.sdmx.metadata+csv; version=2.1.0`

This media type is listed among the response types of the
[SDMX REST API reference metadata queries](https://github.com/sdmx-twg/sdmx-rest/blob/master/doc/metadata.md).

### Optional parameters

Optional parameters MAY be added to the `Accept` header, separated by `"; "`.

**`labels`** (`id` | `name` | `both`; default `id`): applies to all nameable
SDMX artefacts in the header and the body of the message.

- `id`: only the ID of artefacts is shown.
- `name`: the ID/value and the localised name are shown in separate columns,
  the ID/value column directly preceding its related localised name column
  (see [Columns](#columns)).
- `both`: the ID and the localised name are concatenated, separated by `": "`
  (see [Localisation](#localisation) for how languages are chosen).

### Localisation

- Clients MAY indicate preferred languages through the HTTP
  [`Accept-Language`](https://www.rfc-editor.org/rfc/rfc9110#field.accept-language)
  header, e.g. `fr, en-gb;q=0.8, en;q=0.7`.
- Senders MUST localise all artefact names according to the preferred
  language: the best language match according to the user's preferences in
  `Accept-Language` (or, if not available, the system's default language
  order) is used for each localisable name element. The message does not
  indicate the language returned per name element.
- If no language matches for a particular name element, the sender MAY return
  it in a system-default language or MAY omit it.
- Senders SHOULD indicate all languages used anywhere in the message for
  localised name elements through the HTTP `Content-Language` response header.
- Multi-lingual metadata attribute values are not affected: all language
  versions are always provided, independently of the preferred language (see
  [Non-coded multi-lingual metadata attributes](#non-coded-multi-lingual-metadata-attributes)).
  A metadataset containing only some languages is marked with
  [`IS_PARTIAL_LANGUAGE`](#is_partial_language).

## Examples

Note: unless stated otherwise, all examples assume the minimal HTTP `Accept`
header `application/vnd.sdmx.metadata+csv; version=2.1.0`.

??? example "Ordinary case"

    ```csv
    MDSTRUCTURE,MDSTRUCTURE_ID,METADATASET_ID,TARGET_TYPES,TARGET_IDS,ATTRIBUTE_1,ATTRIBUTE_1.CHILD,ATTRIBUTE_2
    metadataflow,OECD:MDF(1.0.0),OECD:MDS(1.0.0),dataflow,OECD:DF(1.0.0),A STRING VALUE,"<p>An XHTML text with ""quotes""</p>",123
    ```

    !!! note

        The default parameter setting `labels=id` applies.

??? example "Metadata attribute with multiple instances and multi-lingual values"

    ```csv
    MDSTRUCTURE[;],MDSTRUCTURE_ID,METADATASET_ID,TARGET_TYPES,TARGET_IDS,ATTRIBUTE_1,ATTRIBUTE_1.ATTRIBUTE_1_2,ATTRIBUTE_2,ATTRIBUTE_3
    metadataflow,OECD:MDF(1.0.0),OECD:MDS(1.0.0),dataflow,OECD:DF(1.0.0),CODE_ID,"""en:""""<p>An XHTML text</p>"""";fr:""""<p>Un texte XHTML</p>"""""";""en:""""<p>Another XHTML text</p>"""";fr:""""<p>Un autre texte XHTML</p>""""""","""Text with """"quotes"""""";""Another text""",123;456
    ```

??? example "Localisation: HTTP Accept header: `application/vnd.sdmx.metadata+csv; version=2.1.0; labels=both`, HTTP Accept-Language header: `fr-FR, en;q=0.7`"

    Metadata attribute with multiple instances, metadata attributes with
    multi-lingual values.

    ```csv
    MDSTRUCTURE[|];MDSTRUCTURE_ID;METADATASET_ID;TARGET_TYPES;TARGET_IDS;ATTRIBUTE_1: Attribut d'exemple 1;ATTRIBUTE_1.ATTRIBUTE_1_2: Attribut d'exemple 12;ATTRIBUTE_2: Attribut d'exemple 2
    metadataflow;OECD:MDF(1.0.0): Metadataflow d'exemple;OECD:MDS(1.0.0): Metadataset d'exemple;dataflow;OECD:DF(1.0.0): Dataflow d'exemple;CODE_ID: Nom du code;"""en:""""<p>An XHTML text</p>""""|fr:""""<p>Un texte XHTML</p>""""""|""en:""""<p>Another XHTML text</p>""""|fr:""""<p>Un autre texte XHTML</p>""""""";123,45|6,789
    ```

    The client prefers French with the France locale (`fr-FR`), but also
    accepts any English. The message therefore applies French with the France
    locale, which also changes the field separator from comma (`,`) to
    semicolon (`;`) and the decimal separator from dot (`.`) to comma (`,`).
    The field separator is the character following `MDSTRUCTURE[|]`.

??? example "Localisation: HTTP Accept header: `application/vnd.sdmx.metadata+csv; version=2.1.0; labels=name`, HTTP Accept-Language header: `en-US`"

    Metadata attribute with multiple instances, metadata attributes with
    multi-lingual values, different targets and metadatasets.

    ```csv
    MDSTRUCTURE[;],MDSTRUCTURE_ID,MDSTRUCTURE_NAME,METADATASET_ID,METADATASET_NAME,TARGET_TYPES,TARGET_IDS,TARGET_NAMES,ATTRIBUTE_1,Attribute 1,ATTRIBUTE_1.ATTRIBUTE_1_2,Attribute 12,ATTRIBUTE_2,Attribute 2
    metadataflow,OECD:MDF(1.0.0),Metadataflow name,OECD:MDS(1.0.0),Metadataset name,dataflow;dataflow,OECD:DF(1.0.0);OECD:DF(1.1.0),Dataflow name 1;Dataflow name 2,CODE_ID,Code name,"""en:""""<p>An XHTML text</p>"""";fr:""""<p>Un texte XHTML</p>"""""";""en:""""<p>Another XHTML text</p>"""";fr:""""<p>Un autre texte XHTML</p>""""""",,123.45;6.789,
    metadataflow,OECD:MDF(1.0.0),Metadataflow name,OECD:MDS(1.1.0),Metadataset new name,codelist,OECD:CL(1.0.0),Codelist name,CODE_ID,Code name,"""en:""""<p>Text 1</p>"""";fr:""""<p>Texte 1</p>"""""";""en:""""<p>Text 2</p>"""";fr:""""<p>Texte 2</p>""""""",,0,
    ```

    The name columns of the non-coded metadata attributes `ATTRIBUTE_1_2` and
    `ATTRIBUTE_2` are empty.

??? example "Varying metadataflows"

    ```csv
    MDSTRUCTURE[;],MDSTRUCTURE_ID,METADATASET_ID,TARGET_TYPES,TARGET_IDS,ATTRIBUTE_1,ATTRIBUTE_2
    metadataflow,OECD:MDF(1.0.0),OECD:MDS(1.0.0),dataflow,OECD:DF(1.0.0),CODE_ID,"""en:Value1;fr:Valeur1"";""en:Value2;de:Wert2"""
    metadataflow,OECD:MDF(1.1.0),OECD:MDS(1.1.0),dataflow,OECD:DF(1.1.0),CODE_ID,"""en:Value1;fr:Valeur1"";""en:Value2;de:Wert2"""
    ```

??? example "Non-versioned metadataset for a non-versioned metadata provision agreement"

    ```csv
    MDSTRUCTURE[;],MDSTRUCTURE_ID,METADATASET_ID,TARGET_TYPES,TARGET_IDS,ATTRIBUTE_1,ATTRIBUTE_2
    metadataprovision,OECD:MDP,OECD:MDS,dataflow,OECD:DF(1.0.0),CODE_ID,"en:Value1;fr:Valeur1"
    ```

??? example "Non-coded metadata attribute values with line breaks"

    ```csv
    MDSTRUCTURE[;],MDSTRUCTURE_ID,METADATASET_ID,TARGET_TYPES,TARGET_IDS,ATTRIBUTE_1
    metadataflow,OECD:MDF(1.0.0),OECD:MDS(1.0.0),dataflow,OECD:DF(1.0.0),"""This text with a line
    break"";""This is some other text"""
    ```

??? example "Metadatasets with partial languages"

    ```csv
    MDSTRUCTURE[;],MDSTRUCTURE_ID,METADATASET_ID,IS_PARTIAL_LANGUAGE,TARGET_TYPES,TARGET_IDS,ATTRIBUTE_1,ATTRIBUTE_2
    metadataflow,OECD:MDF(1.0.0),OECD:MDS(1.0.0),1,dataflow,OECD:DF(1.0.0),CODE_ID,"""en:Value1"";""en:Value2"""
    metadataflow,OECD:MDF(1.1.0),OECD:MDS(1.1.0),1,dataflow,OECD:DF(1.1.0),CODE_ID,"""en:Value1"";""en:Value2"""
    ```

---

[^nv]:
    Since SDMX 3.0.0, the syntax _AGENCY:ARTEFACT_ID(VERSION)_ allows
    omitting the version for non-versioned artefacts. In this case
    _AGENCY:ARTEFACT_ID_ is sufficient, e.g. `OECD:MDP`.
