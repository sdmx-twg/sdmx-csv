# Data Message Field Guide

## Introduction

SDMX-CSV Data Message is an SDMX data exchange format based on
[RFC 4180](https://tools.ietf.org/html/rfc4180). CSV is a widely used,
standardised and simple format to exchange data, supported by many tools.

SDMX-CSV integrates with other specifications:

- The SDMX RESTful API specification (e.g. content negotiation with a media
  type to get SDMX-CSV representations, specific formats for responses,
  language selection through HTTP content negotiation).
- The [RFC 4180](https://tools.ietf.org/html/rfc4180) specification.

SDMX-CSV is designed and optimised for general public dissemination of
statistical data and for use in common statistical software. Messages can
also be used directly to create pivot tables in spreadsheet applications.

### Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "MAY" and "OPTIONAL" in this document are to be
interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119)
and [RFC 8174](https://www.rfc-editor.org/rfc/rfc8174) when, and only when,
they appear in all capitals.

This guide distinguishes two roles:

- **Sender**: a system that writes an SDMX-CSV message, e.g. an SDMX web
  service returning a response, or a data provider preparing an upload.
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

An SDMX-CSV data message consists of one mandatory header row followed by any
number of data rows. Each data row contains either:

- the information related to **one observation**, or
- the information related to **one or more attributes attached to a partial
  key**, or
- for _Delete_ actions only, information concerning **several observations**
  when dimensions are wildcarded.

```csv
STRUCTURE,STRUCTURE_ID,ACTION,DIM_1,DIM_2,DIM_3,OBS_VALUE,ATTR_1
dataflow,ESTAT:NA_MAIN(1.6.0),M,A,B,2014-01,12.4,N
```

### Column overview

| Column name | Description | Position |
| --- | --- | --- |
| **Fixed columns** | | |
| [`STRUCTURE`](#structure) | Type of structure for which the row's data is defined: `dataflow`, `datastructure` or `dataprovision`. The character directly after this term is the field separator. | 1 |
| [`STRUCTURE[;]`](#structure) | `STRUCTURE` extended with the sub-field separator in square brackets. REQUIRED when the message contains multi-valued or multi-lingual values. | 1 |
| [`STRUCTURE_ID`](#structure_id) | Artefact identification `AGENCY:ID(VERSION)`. | 2 |
| [`STRUCTURE_NAME`](#structure_name) | Localised name of the artefact. Only with `labels=name`. | 3 (optional) |
| [`ACTION`](#action) | Action code: `M` Merge, `R` Replace, `D` Delete (`I` and `A` deprecated). | Directly after the structure identification (optional) |
| [`SERIES_KEY`](#series_key-and-obs_key) | Series key. Only with `keys=series` or `keys=both`. | Directly after `ACTION` (optional) |
| [`OBS_KEY`](#series_key-and-obs_key) | Observation key. Only with `keys=obs` or `keys=both`. | Directly after `SERIES_KEY` if present (optional) |
| **Component columns** | | |
| [`EXAMPLE`](#component-columns) | Component ID (dimension, measure, attribute, incl. metadata attributes). | After the fixed columns |
| [`EXAMPLE: Name`](#component-columns) | ID and localised name in one header. Only with `labels=both`. | Replaces `EXAMPLE` |
| [`Name`](#component-columns) | Additional column with localised names. Only with `labels=name`. | Directly after its component column |
| [`PARENT.CHILD`](#nested-metadata-attributes) | Nested metadata attribute: the attribute ID prefixed with the IDs of its parent attribute(s). | — |
| **Other columns** | | |
| [`UPDATED`](#custom-columns) (any custom term) | Implementer-defined column. | Not specified |

Multi-valued and multi-lingual components have no specific header syntax: the
header is the plain component ID, and the type of the component is defined by
the data structure definition (see [Component values](#component-values)).

## Separators

### Field separator

- Senders SHOULD use the comma `,` as field separator, unless the user's locale
  (e.g. as indicated in the HTTP `Accept-Language` header) calls for another
  one, e.g. the semicolon `;` used in some regions or countries.
- The field separator of a message is the character directly following the
  fixed first header term `STRUCTURE`, or, when that term is extended with a
  square-bracket term (see below), the character directly following `]`.
  Receivers MUST determine the field separator this way.

### Sub-field separator

- Multi-valued and multi-lingual component values use a sub-field separator,
  e.g. `;`.
- When a message contains such values, the sender MUST declare the sub-field
  separator as the only character inside square brackets appended to the first
  header term, e.g. `STRUCTURE[;]` or `STRUCTURE[|]`.
- The sub-field separator MUST differ from the field separator.

## Columns

### Column order

1. `STRUCTURE`
2. `STRUCTURE_ID`
3. `STRUCTURE_NAME` (only with `labels=name`)
4. `ACTION` (optional)
5. `SERIES_KEY` and/or `OBS_KEY`, in this order (only with the `keys`
   parameter)
6. Component columns
7. Custom columns (any position)

The fixed columns (1–5) MUST appear in this order, without gaps between those
present.

Component columns:

- SDMX web services SHOULD return component columns in the order in which the
  components are defined in the underlying data structure definition(s),
  grouped by type of component. When data from several data structures are
  present: first the dimensions of the first data structure, then the remaining
  dimensions of the second data structure and so forth; then the measures in
  the same way; then the attributes in the same way.
- Receivers processing uploads MUST accept component columns in any order.

### `STRUCTURE`

**Header:** `STRUCTURE`, or `STRUCTURE[x]` where `x` is the sub-field separator
(see [Sub-field separator](#sub-field-separator)).

**Content:** the type of artefact for which the data in the row are defined:

| Value | Artefact |
| --- | --- |
| `dataflow` | Dataflow |
| `datastructure` | Data structure definition |
| `dataprovision` | Data provision agreement |

A message MAY contain rows for different artefacts, including artefacts based
on different data structures (see [Rows](#rows)).

### `STRUCTURE_ID`

**Header:** `STRUCTURE_ID`.

**Content:**

- Default: the artefact identification in the form
  _AGENCY:ARTEFACT_ID(VERSION)_, e.g. `ESTAT:NA_MAIN(1.6.0)`. For
  non-versioned artefacts the version is omitted, e.g. `AGENCY:DF_ID`[^nv].
- With `labels=both`: the artefact identification and its localised name
  separated by `": "`, e.g.
  `ESTAT:NA_MAIN(1.6.0): National Accounts Main Aggregates`.

### `STRUCTURE_NAME`

**Header:** `STRUCTURE_NAME`.

Present only with `labels=name`, directly after `STRUCTURE_ID`.

**Content:** the artefact's localised name, e.g.
`National Accounts Main Aggregates`.

### `ACTION`

**Header:** `ACTION`.

Senders SHOULD include this column. If present, it MUST directly follow the
structure identification column(s).

**Content:** one character representing the action to be performed with the
row's data:

| Code | Action | Status |
| --- | --- | --- |
| `M` | Merge | Current |
| `R` | Replace | Current |
| `D` | Delete | Current |
| `I` | Information | Deprecated; receivers updating an SDMX storage system MUST process it as Merge |
| `A` | Append | Deprecated; receivers updating an SDMX storage system MUST process it as Merge |

If the `ACTION` column is absent, receivers MUST process all rows of the
message as Merge.

The semantics of each action are described in [Data actions](#data-actions).

### `SERIES_KEY` and `OBS_KEY`

**Headers:** `SERIES_KEY`, `OBS_KEY`.

Present only when requested through the [`keys`](#optional-parameters)
parameter, directly after `ACTION` (or after the structure identification if
`ACTION` is absent). If both are present, `SERIES_KEY` comes first.

**Content:**

- `SERIES_KEY`: the IDs/values of all dimensions except the one(s) attached to
  the observation, in data structure order, separated by a dot `.`, e.g.
  `M.USD.EUR.SP00`.
- `OBS_KEY`: the IDs/values of all dimensions, in data structure order,
  separated by a dot `.`, e.g. `M.USD.EUR.SP00.2020-01`.

### Component columns

Each data structure component included in the message (dimensions, measures,
attributes, including metadata attributes defined through a referenced
metadata structure definition) is represented in one column, or two columns
with `labels=name`.

**Inclusion rules:**

- A message MUST include the dimension columns required to uniquely identify
  the measures and attributes it contains. Other dimension columns MAY be
  omitted (see [Rows](#rows) and [Data actions](#data-actions)).
- Measure columns MAY be omitted.
- Attribute columns MAY be omitted, even for attributes with a mandatory
  status.
- When an SDMX web service streams its response, it may not know, while
  writing the header row, which measures and attributes will have values. A
  measure or attribute column MAY therefore contain only empty values.

**Header:**

| `labels` | Header | Example |
| --- | --- | --- |
| `id` (default) | Component ID | `DIM_1` |
| `both` | Component ID and localised name separated by `": "` | `DIM_1: Dimension 1` |
| `name` | Component ID, followed by an additional column whose header is the localised name | `DIM_1`, `Dimension 1` |

The header contains only the component ID, or for nested metadata attributes
the dotted ID path (`PARENT.CHILD`). No suffix indicates whether a component
is multi-valued or multi-lingual; see [Component values](#component-values).

**Content:**

| `labels` | Coded component | Non-coded component |
| --- | --- | --- |
| `id` (default) | Code ID, e.g. `A` | Value |
| `both` | Code ID and localised name separated by `": "`, e.g. `A: A value name` | Value |
| `name` | Code ID; the additional column contains the localised name, e.g. `A value name`, and is empty if the value has no localised name | Value; the additional column is empty |

Note that `": "` may also occur inside a name and may therefore occur several
times within a concatenated value.

### Custom columns

Senders MAY add any other custom columns, e.g. `UPDATED` or `PREPARED`.

- The header MUST be a custom but unique term.
- The content MAY be localised.

## Component values

Whether a component is multi-valued and/or multi-lingual is defined only by
the data structure definition; column headers carry no indication of it.
Receivers MUST use the data structure definition to interpret the content of
such columns, e.g. to know whether the sub-field separator splits the field
into several values.

### Multi-valued components

- **Header:** the plain component ID, e.g. `ATTR_1`.
- **Content:** the values separated by the sub-field separator, e.g. `X;Y`.
- With `labels=both`, each coded value is prefixed with its ID and `": "`, e.g.
  `A: Value A;B: Value B`.
- Requires the sub-field separator to be declared (`STRUCTURE[x]`).

### Non-coded multi-lingual components

- **Header:** the plain component ID, e.g. `ATTR_2`.
- **Content:** each language value prefixed with its ISO 639-1 language code
  and a colon `:`, the values separated by the sub-field separator, e.g.
  `en:Value;fr:Valeur`. Unlike the ID prefix used with `labels=both`, there is
  no space after the colon.
- Multi-lingual components are always non-coded, so the language prefix cannot
  be confused with code IDs.
- Requires the sub-field separator to be declared (`STRUCTURE[x]`).

### Non-coded multi-lingual multi-valued components

- **Header:** the plain component ID, e.g. `ATTR_2`.
- **Content:** each multi-lingual value set enclosed in double quotes, the sets
  separated by the sub-field separator, e.g.
  `"en:Value1;fr:Valeur1";"en:Value2;de:Wert2"`. Since the field itself
  contains double quotes, it MUST be enclosed in double quotes and the inner
  quotes MUST be doubled:
  `"""en:Value1;fr:Valeur1"";""en:Value2;de:Wert2"""`.
- Requires the sub-field separator to be declared (`STRUCTURE[x]`).

### Nested metadata attributes

- Each metadata attribute is presented in its own column(s), even when nested.
- **Header:** the attribute ID prefixed with the IDs of its parent
  attribute(s), separated by a dot `.`, e.g. `COLLECTION.METHOD` or
  `CONTACT.NAME`, whether or not the parent or child is multi-valued.
- **Content:** when the parent is multi-valued, all values of the attribute,
  presented like a multi-valued component and respecting their position in
  the attribute tree, e.g. `name for contact 1;name for contact 2`.
- A parent branch without a value for the attribute is indicated by an empty
  sub-field, e.g. `name for contact 1;;name for contact 3`.
- Nested parent branches are indicated by nested double quotes, with inner
  quotes doubled recursively, e.g.
  `"""name 1 for contact 1;name 2 for contact 1"";""name 1 for contact 2;name 2 for contact 2"""`.

### Non-coded XHTML values

- Some non-coded components allow XHTML values.
- Each XHTML value MUST be enclosed in double quotes, with inner quotes
  doubled, e.g. `"<p>This is some ""metadata html""</p>"`.
- As allowed by RFC 4180 for quoted fields, XHTML values MAY contain line
  breaks.

### Special values

| Value | Meaning | Applies to |
| --- | --- | --- |
| _(empty)_ | Omitted: the value is not provided (see [Data actions](#data-actions) for the effect per action) | All components |
| `~` | Switched off: the dimension is not relevant for this row (see [Data actions](#data-actions)) | Dimensions |
| `NaN` | Intentionally missing value | Measures and attributes of type float or double |
| `#N/A` | Intentionally missing value | Measures and attributes of all other types |
| `-` (or any valid value) | Marks a value to be deleted in a _Delete_ row | Measures and attributes |

Intentionally missing values MUST be used to indicate observations,
attributes or reference metadata values that are deliberately missing, even
if they are mandatory.

## Rows

- **Observation rows** contain the dimension values identifying one
  observation and its measure and attribute values. Values of attributes
  attached to partial keys MAY have to be repeated on each observation row.
- **Partial-key attribute rows** contain the attribute values attached to a
  partial key. Only the dimensions that are part of the partial key need to be
  filled; all other component columns MAY be left empty.
- **Delete rows** concern data at the lowest level of detail provided. To
  delete specific measure or attribute values, those values are non-empty,
  e.g. marked with `-`. Dimensions MAY be wildcarded by leaving the field
  empty.
- **Multiple structures:** when a message contains data from different data
  structures, the columns not related to the row's data structure MUST be left
  empty.

## Data actions

### Component states per action

| | Dimension value omitted | Dimension value switched off | Measure, attribute or reference metadata value omitted | Measure, attribute or reference metadata value present |
| --- | --- | --- | --- | --- |
| **Delete** | Bulk deletion: the dimension value doesn't matter | Only for irrelevant dimensions (see below) | Deleted only if **all** non-dimension components are omitted | Deleted |
| **Merge** | _Bulk merge is not permitted_ | Only for irrelevant dimensions (see below) | Not changed | Updated or inserted |
| **Replace** | _Bulk replace is not permitted_ | Only for irrelevant dimensions (see below) | At permitted replacement levels: deleted; otherwise not changed | Updated or inserted |
| **CSV representation** | Empty cell, or column absent | `~` | Empty cell, or column absent | Any valid or intentionally missing value |

A dimension is irrelevant, and MAY be switched off, for:

1. higher-level (reference metadata) attributes not attached to this
   dimension (including `TIME_PERIOD`);
2. measures and attributes not attached to this dimension, if the data
   structure allows for an "evolving structure" (excluding `TIME_PERIOD`).

### Merge (`M`)

Data or data-related reference metadata is merged, through either update or
insertion depending on already existing information.

- Merge does not delete any component values.
- Individual values within multi-valued measures, attributes or data-related
  reference metadata cannot be updated; the complete multi-valued value MUST be
  provided.
- Only non-dimension components (measures, attributes, data-related reference
  metadata) MAY be omitted, as long as at least one of them is present. Bulk
  merges are therefore not supported. Only the provided values are merged.
- Dimension values for higher-level (data-related reference metadata)
  attributes MAY be switched off (`~`) when those attributes are not attached
  to these dimensions.
- When used to update an SDMX storage system, all observations and all sets of
  data-related reference metadata attributes at specific dimension
  combinations affected by the Merge change their timestamp.

### Replace (`R`)

Data or data-related reference metadata is replaced, through either update,
insertion or deletion depending on already existing information.

- A full replacement takes place at specific "replacement levels": for entire
  observations, and for any specific dimension combination for data-related
  reference metadata attributes. Within these levels, provided values are
  inserted or updated and omitted values are deleted.
- Values provided for other attributes (above the observation level) are
  merged (see Merge).
- Only non-dimension components MAY be omitted. Bulk replacing is therefore
  not supported.
- Dimension values for higher-level (data-related reference metadata)
  attributes MAY be switched off (`~`) when those attributes are not attached
  to these dimensions.
- Replacing non-existing elements MUST NOT result in an error.
- When used to update an SDMX storage system, all observations and all sets of
  data-related reference metadata attributes at specific dimension
  combinations affected by the Replace change their timestamp.
- Because Replace always takes place at specific levels, it cannot replace a
  whole dataset or a whole series. A "replace all" effect can be achieved by
  combining a Delete row with a completely wildcarded key (all dimension
  values omitted) with Merge or Replace rows in the same message. Similarly, to
  replace a whole series, combine a Delete row with only the series' partial
  key with Merge or Replace rows for that series.

### Delete (`D`)

Data or data-related reference metadata is deleted, at the lowest level of
detail provided in the message.

- Any component, including dimensions, MAY be omitted. Omitting dimension
  values allows bulk deletions.
- Partially omitting non-dimension values restricts the deletion to the
  measure, attribute or data-related reference metadata values that are
  present. Instead of real values, any valid value MAY be used, e.g. `-`.
- This allows deleting all dataflow data, slices of observations for dimension
  groups such as time series, single observations, or individual measure,
  attribute and data-related reference metadata values.
- Dimension values for higher-level (data-related reference metadata)
  attributes MAY be switched off (`~`) when those attributes are not attached
  to these dimensions.
- Deleting non-existing elements or values MUST NOT result in an error.
- When used to update an SDMX storage system, all observations and all sets of
  attributes and data-related reference metadata at higher partial keys
  affected by the Delete change their timestamp.

### Transactions

All datasets within a single data message, even with varying actions, MUST be
treated as one **ACID transaction**, guaranteeing full data consistency and
validity despite errors, power failures and other mishaps. Datasets MUST be
processed in their order of appearance in the message. This allows bundling
separate Delete and Replace or Merge actions into one transactional message.

### Physical storage

The terms "delete", "merge" and "replace" do **not** imply a physical
replacement or deletion of values in the underlying database.

- SDMX web services that do not support the `includeHistory` and `asOf` URL
  parameters MAY physically replace existing values, to minimise physical
  resource requirements. Those that also do not support `updatedAfter` MAY
  physically delete values.
- SDMX web services that support these parameters (or other time-machine
  features) MUST NOT overwrite or physically delete values.
- SDMX web services that support `includeHistory` or `asOf` SHOULD NOT allow
  deleting their **historic** data content, because this would interfere with
  the interests of data consumers such as data aggregators. A feature to
  physically delete outdated content is intentionally not part of the SDMX
  standard; if required, it must be implemented as a custom feature outside
  the standard.
- All SDMX-compliant systems that support (or are configured to support) the
  `updatedAfter` URL parameter MUST systematically retain information about
  deleted data and data-related reference metadata.

### Dataset actions in web service responses

The following is RECOMMENDED[^bc] for responses to GET data queries:

1. Without the `updatedAfter`, `includeHistory`, `detail`, `attributes` or
   `measures` URL parameters: the retrieved data in a _Replace_ dataset
   (instead of the previous _Information_ dataset).
2. Without `updatedAfter` and `includeHistory`, but with `detail`,
   `attributes` or `measures`: the retrieved data in a _Merge_ dataset
   (instead of the previous _Information_ dataset).
3. With `updatedAfter`: the response MUST include all previously updated,
   inserted and deleted data or data-related reference metadata, even if bulk
   deletions were used. Two approaches are possible:
    - a _Delete_ dataset for entirely deleted observations and for entirely
      deleted sets of (data-related reference metadata) attribute values
      attached to specific dimension combinations, **and** a _Replace_ dataset
      for all other changed observations and changed attribute and
      data-related reference metadata values attached to specific dimension
      combinations; or
    - a _Delete_ dataset for entirely deleted observations, for entirely
      deleted sets of (data-related reference metadata) attribute values
      attached to specific dimension combinations and for individually deleted
      measure, attribute and reference metadata values, **and** a _Merge_
      dataset for all other updated or inserted observation, attribute and
      data-related reference metadata values.

    The database synchronisation use case requires that the response always
    allows replicating exactly the same point-in-time data content as
    currently stored in the queried data source.
4. With `includeHistory`: a number of datasets with Delete, Replace or Merge
   actions, each limited in its validity time span, that allow replicating
   exactly the same point-in-time data contents as previously stored in the
   queried data source.
5. With `asOf`: recommendations 1 and 2 apply depending on the other
   parameters. In addition, the returned dataset SHOULD have its validity time
   span limited to the point in time requested in `asOf`.

## HTTP integration

### Media type

Clients SHOULD request SDMX-CSV data messages with the following HTTP
`Accept` header:

`application/vnd.sdmx.data+csv; version=2.1.0`

### Optional parameters

Optional parameters MAY be added to the `Accept` header, separated by `"; "`.
The parameters and their values are those defined for SDMX-CSV in the
[SDMX REST API data queries](https://github.com/sdmx-twg/sdmx-rest/blob/master/doc/data.md).

**`labels`** (`id` | `name` | `both`; default `id`): applies to all nameable
SDMX artefacts in the header and the body of the message.

- `id`: only the ID/value of artefacts is shown.
- `name`: the ID/value and the localised name are shown in separate columns,
  the ID/value column directly preceding its localised name column (see
  [`STRUCTURE_NAME`](#structure_name) and
  [Component columns](#component-columns)).
- `both`: the ID/value and the localised name are concatenated, separated by
  `": "` (see [Localisation](#localisation) for how languages are chosen).

**`timeFormat`** (`original` | `normalized`; default `original`):

- `original`: time dimension (`TIME_PERIOD`) values are shown in the SDMX
  `TIME_PERIOD` format as originally recorded.
- `normalized`: `TIME_PERIOD` values are converted to the most granular
  [ISO 8601](https://www.iso.org/iso-8601-date-and-time-format.html)
  representation, taking into account the highest frequency of the data in
  the message and the moment in time when lower-frequency values were
  collected (typically the beginning, middle or end of the reporting period).
  This eases comparison and analysis of multi-frequency data, e.g. in pivot
  tables. For example, if a message contains annual and daily data and annual
  data were collected at the end of the period, the annual period `2014`
  becomes `2014-12-31`.

**`keys`** (`none` | `obs` | `series` | `both`; default `none`): requests the
addition of key columns (see [`SERIES_KEY` and `OBS_KEY`](#series_key-and-obs_key)).

- `none`: no key column is added.
- `obs`: an `OBS_KEY` column is added.
- `series`: a `SERIES_KEY` column is added.
- `both`: both `SERIES_KEY` and `OBS_KEY` columns are added, `SERIES_KEY`
  first.

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
- Multi-lingual component values are not affected: all language versions are
  always provided, independently of the preferred language (see
  [Non-coded multi-lingual components](#non-coded-multi-lingual-components)).

## Examples

Note: unless stated otherwise, all examples assume the minimal HTTP `Accept`
header `application/vnd.sdmx.data+csv; version=2.1.0`.

??? example "Ordinary case"

    ```csv
    STRUCTURE,STRUCTURE_ID,ACTION,DIM_1,DIM_2,DIM_3,OBS_VALUE,ATTR_2,ATTR_3,ATTR_1,UPDATED
    dataflow,ESTAT:NA_MAIN(1.6.0),M,A,B,2014-01,12.4,Y,"Normal, special and other values",N,2021-01-22T13:15:41Z
    dataflow,ESTAT:NA_MAIN(1.6.0),M,A,B,2014-02,10.8,Y,"Normal, special and other values",Y,2021-01-22T13:15:41Z
    ```

    !!! note

        - The default parameter settings `labels=id` and
          `timeFormat=original` apply.
        - `UPDATED` is a custom column.

??? example "Components in any order, missing component(s), component with multiple values"

    ```csv
    STRUCTURE[;],STRUCTURE_ID,ACTION,OBS_VALUE1,OBS_VALUE2,ATTR_3,ATTR_1,DIM_2,DIM_1,DIM_3
    dataflow,ESTAT:NA_MAIN(1.6.0),M,12.4,12.5,"Normal, special and other values",X;Y,B,A,2014-01
    dataflow,ESTAT:NA_MAIN(1.6.0),M,10.8,10.9,"Normal, special and other values",X;Z,B,A,2014-02
    ```

??? example "Components in any order and missing component, HTTP Accept header: `application/vnd.sdmx.data+csv; version=2.1.0; keys=series`"

    ```csv
    STRUCTURE,STRUCTURE_ID,ACTION,SERIES_KEY,OBS_VALUE1,OBS_VALUE2,ATTR_3,ATTR_1,DIM_2,DIM_1,DIM_3
    dataflow,ESTAT:NA_MAIN(1.6.0),M,A.B,12.4,12.5,"Normal, special and other values",N,B,A,2014-01
    dataflow,ESTAT:NA_MAIN(1.6.0),M,A.B,10.8,10.9,"Normal, special and other values",Y,B,A,2014-02
    ```

??? example "Localisation: HTTP Accept header: `application/vnd.sdmx.data+csv; version=2.1.0; labels=both; keys=both`, HTTP Accept-Language header: `fr-FR, en;q=0.7`"

    ```csv
    STRUCTURE[|];STRUCTURE_ID;ACTION;SERIES_KEY;OBS_KEY;DIM_1: Dimension 1;DIM_2: Dimension 2;DIM_3: Dimension 3;OBS_VALUE: Observation value;ATTR_2: Attribut 2;ATTR_3: Attribut 3;ATTR_1: Attribut 1
    dataflow;ESTAT:NA_MAIN(1.6.0): Principaux agrégats des comptes nationaux;M;A.B;A.B.2014-01;A: Value A;B: Value B;2014-01: 2014-01;12,4;Y: Oui;Normal, special and other values;N: Non
    dataflow;ESTAT:NA_MAIN(1.6.0): Principaux agrégats des comptes nationaux;M;A.B;A.B.2014-02;A: Value A;B: Value B;2014-02: 2014-02;10,8;Y: Oui;Normal, special and other values;Y: Oui
    ```

    The client prefers French with the France locale (`fr-FR`), but also
    accepts any English. The message therefore applies French with the France
    locale, which also changes the field separator from comma (`,`) to
    semicolon (`;`) and the decimal separator from dot (`.`) to comma (`,`).
    The field separator is the character following `STRUCTURE[|]`.

??? example "HTTP Accept header: `application/vnd.sdmx.data+csv; version=2.1.0; labels=both; timeFormat=normalized`"

    ```csv
    STRUCTURE,STRUCTURE_ID,ACTION,DIM_1: Dimension 1,DIM_2: Dimension 2,DIM_3: Dimension 3,OBS_VALUE: Observation value,ATTR_2: Attribute 2,ATTR_3: Attribute 3,ATTR_1: Attribute 1
    dataflow,ESTAT:NA_MAIN(1.6.0): National Accounts Main Aggregates,M,A: Value A,B: Value B,2014-01-01,12.4,Y: Yes,"Normal, special and other values",N: No
    dataflow,ESTAT:NA_MAIN(1.6.0): National Accounts Main Aggregates,M,A: Value A,B: Value B,2014-02-01,10.8,Y: Yes,"Normal, special and other values",Y: Yes
    ```

??? example "HTTP Accept header: `application/vnd.sdmx.data+csv; version=2.1.0; labels=name`"

    ```csv
    STRUCTURE,STRUCTURE_ID,STRUCTURE_NAME,ACTION,DIM_1,Dimension 1,DIM_2,Dimension 2,DIM_3,Dimension 3,OBS_VALUE,Observation value,ATTR_2,Attribute 2,ATTR_3,Attribute 3,ATTR_1,Attribute 1
    dataflow,ESTAT:NA_MAIN(1.6.0),National Accounts Main Aggregates,M,A,Value A,B,Value B,2014-01,2014-01,12.4,,Y,Yes,"Normal, special and other values",,N,No
    dataflow,ESTAT:NA_MAIN(1.6.0),National Accounts Main Aggregates,M,A,Value A,B,Value B,2014-02,2014-02,10.8,,Y,Yes,"Normal, special and other values",,Y,Yes
    ```

??? example "Multi-valued components"

    ```csv
    STRUCTURE[;],STRUCTURE_ID,ACTION,DIM_1,DIM_2,DIM_3,OBS_VALUE,ATTR_1,ATTR_2,ATTR_3
    dataflow,ESTAT:NA_MAIN(1.6.0),M,A,B,2014-01,12.4,Value X;Value Y,"M, N & O;P & Q",A;B;C
    dataflow,ESTAT:NA_MAIN(1.6.0),M,A,B,2014-02,10.8,Value X;Value Y,"M, N & O;P & Q",A;C
    ```

??? example "Non-coded multi-lingual components, varying dataflows based on the same underlying data structure"

    ```csv
    STRUCTURE[;],STRUCTURE_ID,ACTION,DIM_1,DIM_2,DIM_3,OBS_VALUE,ATTR_1
    dataflow,ESTAT:NA_MAIN(1.6.0),M,A,B,2014-01,12.4,en:Any Value;fr:N'importe quelle Valeur
    dataflow,ESTAT:NA_MAIN(1.7.0),M,A,B,2014-02,10.8,"en:Value ""X"";fr:Valeur ""X"""
    ```

??? example "Varying structural artefacts based on the same underlying data structure"

    ```csv
    STRUCTURE[;],STRUCTURE_ID,ACTION,DIM_1,DIM_2,DIM_3,OBS_VALUE,ATTR_1
    dataflow,ESTAT:DF_NA_MAIN(1.6.0),M,A,B,2014-01,12.4,en:Any Value;fr:N'importe quelle Valeur
    datastructure,ESTAT:DSD_NA_MAIN(1.7.0),M,A,B,2014-02,10.8,"en:Value ""X"";fr:Valeur ""X"""
    dataprovision,ESTAT:DPA_NA_MAIN(1.8.0),M,A,B,2014-03,11.2,"en:Value ""Y"";fr:Valeur ""Y"""
    ```

??? example "Varying structural artefacts based on different underlying data structures"

    ```csv
    STRUCTURE,STRUCTURE_ID,ACTION,DIM_A1B1,DIM_A2,DIM_A3C2,DIM_B2,DIM_C1,DIM_C3,MEAS_A1B1C1,MEAS_C2,ATTR_A1,ATTR_B1
    dataflow,ESTAT:DF_A(1.6.0),M,DIMVAL_A1B1,DIMVAL_A2,DIMVAL_A3C2,,,,MEASVAL_A1B1C1,,ATTRVAL_A1,
    datastructure,ESTAT:DSD_B(1.7.0),M,DIMVAL_A1B1,,,DIMVAL_B2,,,MEASVAL_A1B1C1,,,ATTRVAL_B1
    dataprovision,ESTAT:DPA_C(1.8.0),M,,,DIMVAL_A3C2,,DIMVAL_C1,DIMVAL_C3,MEASVAL_A1B1C1,MEASVAL_C2,,
    ```

??? example "Varying actions"

    ```csv
    STRUCTURE,STRUCTURE_ID,ACTION,DIM_1,DIM_2,DIM_3,OBS_VALUE,ATTR_1
    dataflow,ESTAT:NA_MAIN(1.6.0),M,A,B,2014-01,12.4,X
    dataflow,ESTAT:NA_MAIN(1.6.0),R,A,B,2014-02,10.8,Y
    ```

??? example "Data for a non-versioned data structure definition"

    ```csv
    STRUCTURE,STRUCTURE_ID,ACTION,DIM_1,DIM_2,DIM_3,OBS_VALUE,ATTR_1
    datastructure,AGENCY:DSD_ID,M,A,B,2014-01,12.4,N
    datastructure,AGENCY:DSD_ID,M,A,B,2014-02,10.8,Y
    ```

??? example "Attributes attached to partial keys for a data provision agreement"

    ```csv
    STRUCTURE,STRUCTURE_ID,ACTION,DIM_2,DIM_3,ATTR_1
    dataprovision,AGENCY:DPA_ID(1.0.0),M,B,2014-01,N
    dataprovision,AGENCY:DPA_ID(1.0.0),M,B,2014-02,Y
    ```

??? example "Mixing rows for attributes attached to partial keys with rows for observations"

    ```csv
    STRUCTURE,STRUCTURE_ID,ACTION,DIM_1,DIM_2,DIM_3,MEAS_1,ATTR_1,ATTR_2
    dataflow,AGENCY:DF_ID(1.0.0),M,A,B,2014-01,12.4,N,
    dataflow,AGENCY:DF_ID(1.0.0),M,,B,,,,Y
    ```

??? example "Nested metadata attributes attached to partial keys"

    ```csv
    STRUCTURE[;],STRUCTURE_ID,ACTION,DIM_2,COLLECTION.METHOD,CONTACT,CONTACT.NAME
    dataflow,AGENCY:DF_ID(1.0.0),M,A,en:AAA;fr:BBB,Contact 1;Contact 2,"""Contact 1 Name 1;Contact 1 Name 2"";""Contact 2 Name 1;Contact 2 Name 2"""
    dataflow,AGENCY:DF_ID(1.0.0),M,B,en:CCC;fr:DDD,Contact 1;Contact 2;Contact 3,"""Contact 1 Name 1;Contact 1 Name 2"";;""Contact 3 Name 1;Contact 3 Name 2"""
    ```

    In the second row, `Contact 2` has no names, indicated by the empty
    sub-field.

??? example "Non-coded XHTML-formatted values with line breaks"

    ```csv
    STRUCTURE,STRUCTURE_ID,ACTION,DIM_1,DIM_2,DIM_3,OBS_VALUE,ATTR_1
    dataflow,ESTAT:NA_MAIN(1.6.0),M,A,B,2014-01,12.4,"<p>This is some ""xhtml"" with a line
    break</p>"
    dataflow,ESTAT:NA_MAIN(1.6.0),M,A,B,2014-02,10.8,"<p>This is some other ""xhtml""</p>"
    ```

??? example "Deleting specific measure and attribute values: all non-empty values (e.g. marked with `-`) are deleted"

    ```csv
    STRUCTURE,STRUCTURE_ID,ACTION,DIM_1,DIM_2,DIM_3,OBS_VALUE,ATTR_2,ATTR_3,ATTR_1
    dataflow,ESTAT:NA_MAIN(1.6.0),D,A,B,2014-01,-,,,
    dataflow,ESTAT:NA_MAIN(1.6.0),D,A,B,2014-02,,,-,
    ```

??? example "Deleting specific measure and attribute values with wildcarded dimensions"

    All non-empty values (e.g. marked with `-`) are deleted for all dimension
    combinations where:

    - row 1: `DIM_2=A`
    - row 2: `DIM_2=B`

    ```csv
    STRUCTURE,STRUCTURE_ID,ACTION,DIM_1,DIM_2,DIM_3,OBS_VALUE,ATTR_2,ATTR_3,ATTR_1
    dataflow,ESTAT:NA_MAIN(1.6.0),D,,A,,-,,,
    dataflow,ESTAT:NA_MAIN(1.6.0),D,,B,,,,-,
    ```

??? example "Deleting whole observations with wildcarded dimensions"

    All observations are deleted for all dimension combinations where:

    - row 1: `DIM_2=A`
    - row 2: `DIM_2=B` and `DIM_3=C`

    ```csv
    STRUCTURE,STRUCTURE_ID,ACTION,DIM_2,DIM_3
    dataflow,ESTAT:NA_MAIN(1.6.0),D,A,
    dataflow,ESTAT:NA_MAIN(1.6.0),D,B,C
    ```

??? example "Deleting all data for a data structure definition"

    ```csv
    STRUCTURE,STRUCTURE_ID,ACTION
    datastructure,ESTAT:DSD_NA_MAIN(1.6.0),D
    ```

    or

    ```csv
    STRUCTURE,STRUCTURE_ID,ACTION,DIM_1,DIM_2,DIM_3
    datastructure,ESTAT:DSD_NA_MAIN(1.6.0),D,,,
    ```

---

[^nv]:
    Since SDMX 3.0.0, the syntax _AGENCY:ARTEFACT_ID(VERSION)_ allows
    omitting the version for non-versioned artefacts. In this case
    _AGENCY:ARTEFACT_ID_ is sufficient, e.g. `AGENCY:DF_ID`.

[^bc]:
    So far this is recommended for systems that do not require backward
    compatibility. With SDMX 4.0 this may become mandatory.
