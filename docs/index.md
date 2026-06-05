# SDMX-CSV

SDMX-CSV is a new data exchange format for public data dissemination. SDMX-CSV
format conforms to the RFC 4180 standard specification, and it supports the SDMX
2.1 Information Model. SDMX-CSV is streamable from a supporting web service in
order to limit the amount of server resources. In addition, the format aims to
ease understanding and use of the data messages by non-experts with a simple but
generic message structure that supports different types of data structures. CSV
is a widely used standardised simple format used to exchange data and is
supported by many tools. Testing and development of the format has focused on
statistical and analytical applications.

The SDMX-CSV Data Message works together with the SDMX RESTful Web Services API.
It will assist developers to write software that requests data responses in CSV
from SDMX RESTful API. The current version of SDMX-CSV Data Message focuses on
easy data ingestion by industry-standard analytical or statistical software.
SDMX RESTful API also supports reference metadata queries and structural
metadata queries. Today, these are not supported by the SDMX-CSV format.
