# Snowflake Public Data - SEC Tarrif Sentiment

This demo was built to help showcase Snowflake's Public Data Products.
It leverages the SEC data of the catalog.

## Snowflake Public Data Products

In order to help customers experience the full power of their own data, Snowflake has curated hundreds of datasets that can be downloaded straight to your account from the Snowflake Marketplace!

The contents of these data sets cover a myriad of topics, from [US Real Estate](https://app.snowflake.com/marketplace/listing/GZTSZAS2KI6/snowflake-public-data-products-us-real-estate?originTab=provider&providerName=Snowflake%20Public%20Data%20Products&profileGlobalName=GZTSZAS2KCS) to information about Canada's GDP in the [Canadian Government Data](https://app.snowflake.com/marketplace/listing/GZTSZAS2KFB/snowflake-public-data-products-canadian-government?originTab=provider&providerName=Snowflake%20Public%20Data%20Products&profileGlobalName=GZTSZAS2KCS) listing.

A full list of Snowflake's data products can be browsed [here](https://data-catalog.snowflake.com/) and any of these data sets can be downloaded to your account from the [Marketplace Listing](https://app.snowflake.com/marketplace/listing/GZTSZ290BV255)


## Why Snowflake Public Data Products

You can gain access to a suite of public domain datasets all in one location on the Snowflake Marketplace. Snowflake Public Data Products provide a foundational layer of data for any analysis.

### Unified Schema
We use a unified schema that balances flexibility for new data with consistency in core tables. This approach helps users who are familiar with Snowflake Datasets to quickly orient themselves to new datasets, which are built around the concepts of entities and timeseries.

### Dataset Joinability 
We create standardized index tables, including geography and company indices, that can be joined to all relevant Snowflake tables.

### Data Release
Snowflake consistently monitors each data source and updates the data product on Snowflake Marketplace automatically.
Point in Time History
History tables provide an auditable record of data changes over time.


## This Demo

The SEC Tarrif Sentiment Demo downloads and utilizes the [SEC Filings](https://app.snowflake.com/marketplace/listing/GZTSZAS2KH9/snowflake-public-data-products-sec-filings) dataset. 

Please upload the SEC_Tarrif_Demo.ipynb notebook to your snowflake account and follow the instructions.

The included notebook will:

- Find all related 10-Q data for fiscal years 2024/2025
- Select any reports that mention "Tarrif" in the report text
- Pass the selected reports to Cortex (claude-4-sonnet) to be labeled.
- Display graphs and explore different results such as different companies sentiment towards tarrifs.
