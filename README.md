# Amazon Delivery Service Partner Data Unification

This dbt package is for the Amazon Delivery Service Partner data unification Ingested by [Daton](https://sarasanalytics.com/daton/). [Daton](https://sarasanalytics.com/daton/) is the Unified Data Platform for Global Commerce with 100+ pre-built connectors and data sets designed for accelerating the eCommerce data and analytics journey by [Saras Analytics](https://sarasanalytics.com).

### Supported Datawarehouses:
- BigQuery
- Snowflake

#### Typical challenges with raw data are:
- Array/Nested Array columns which makes queries for Data Analytics complex
- Data duplication due to look back period while fetching report data from Amazon Marketing UI
- Separate tables at marketplaces/Store, brand, account level for same kind of report/data feeds

By doing Data Unification the above challenges can be overcomed and simplifies Data Analytics. 
As part of Data Unification, the following functions are performed:
- Consolidation - Different marketplaces/Store/account & different brands would have similar raw Daton Ingested tables, which are consolidated into one table with column distinguishers brand & store
- Deduplication - Based on primary keys, the data is De-duplicated and the latest records are only loaded into the consolidated stage tables
- Incremental Load - Models are designed to include incremental load which when scheduled would update the tables regularly
- Standardization -
	- Currency Conversion (Optional) - Raw Tables data created at Marketplace/Store/Account level may have data in local currency of the corresponding marketplace/store/account. Values that are in local currency are standardized by converting to desired currency using Daton Exchange Rates data.
	  Prerequisite - Exchange Rates connector in Daton needs to be present - Refer [this](https://github.com/saras-daton/currency_exchange_rates)
	- Time Zone Conversion (Optional) - Raw Tables data created at Marketplace/Store/Account level may have data in local timezone of the corresponding marketplace/store/account. DateTime values that are in local timezone are standardized by converting to specified timezone using input offset hours.

#### Prerequisite 
Daton Integrations for  
- Amazon DSP 
- Exchange Rates (Optional, if currency conversion is not required)


# Configuration 

## Required Variables

This package assumes that you have an existing dbt project with a BigQuery/Snowflake profile connected & tested. Source data is located using the following variables which must be set in your `dbt_project.yml` file.
```yaml
vars:
    raw_database: "your_database"
    raw_schema: "your_schema"
```

## Setting Target Schema

Models will be create unified tables under the schema (<target_schema>_stg_amazon). In case, you would like the models to be written to the target schema or a different custom schema, please add the following in the dbt_project.yml file.

```yaml
models:
  amazon_dsp:
    +schema: custom_schema_extension
```

## Optional Variables

Package offers different configurations which must be set in your `dbt_project.yml` file. These variables can be marked as True/False based on your requirements. Details about the variables are given below.

### Currency Conversion 

To enable currency conversion, which produces two columns - exchange_currency_rate & exchange_currency_code, please mark the currency_conversion_flag as True. By default, it is False.
Prerequisite - Daton Exchange Rates Integration

Example:
```yaml
vars:
    currency_conversion_flag: True
```

### Table Exclusions

If you need to exclude any of the models, declare the model names as variables and mark them as False. Refer the table below for model details. By default, all tables are created.

Example:
```yaml
vars:
CampaignReport: False
```

## Models

This package contains models from the Amazon Adverstising API which includes reports on {{audience, campaign, inventory, product, geography, technology}}. The primary outputs of this package are described below.

| **Category**                 | **Model**  | **Description** |
| ------------------------- | ---------------| ----------------------- |
|Audience | [AudienceReport](models/AmazonDSP/AmazonDSPAudienceReport.sql)  | Audience reports contain data based on your audience |
|Inventory | [InventoryReport](models/AmazonDSP/AmazonDSPInventoryReport.sql)  | Inventory reports contain data based on your inventory, such as deal and supply source information |
|Campaign | [CampaignReport](models/AmazonDSP/AmazonDSPCampaignReport.sql)  | Campaign reports include performance data for campaigns that have performance activity on your requested dates |
|Geography | [GeographyReport](models/AmazonDSP/AmazonDSPGeographyReport.sql)| Geography reports contain your customers’ geographical data |
|Technology | [TechnologyReport](models/AmazonDSP/AmazonDSPTechnologyReport.sql)| Technology reports contain data based on your customers’ technology |
|Products | [ProductsReport](models/AmazonDSP/AmazonDSPProductsReport.sql)| Products reports contain data based on the products featured in your campaigns|





### For details about default configurations for Table Primary Key columns, Partition columns, Clustering columns, please refer the properties.yaml used for this package as below. 
	You can overwrite these default configurations by using your project specific properties yaml.
```yaml
version: 2
models:
  - name: AmazonDSPCampaignReport
    config:
      materialized: incremental
      incremental_strategy: merge
      unique_key: ['advertiserId','ReportDate','OrderId','LineItemId','CreativeID','CreativeAdId']
      partition_by: { 'field': 'reportDate', 'data_type': 'date' }
      cluster_by: ['advertiserId','OrderId','LineItemId','CreativeID'] 

  - name: AmazonDSPAudienceReport
    config:
      materialized: incremental
      incremental_strategy: merge
      unique_key: ['advertiserId','ReportDate','OrderId','LineItemId']
      partition_by: { 'field': 'reportDate', 'data_type': 'date' }
      cluster_by: ['advertiserId','OrderId','LineItemId'] 

  - name: AmazonDSPGeographyReport
    config:
      materialized: incremental
      incremental_strategy: merge
      unique_key: ['advertiserId','ReportDate','OrderId','LineItemId']
      partition_by: { 'field': 'reportDate', 'data_type': 'date' }
      cluster_by: ['advertiserId','OrderId','LineItemId'] 

  - name: AmazonDSPInventoryReport
    config:
      materialized: incremental
      incremental_strategy: merge
      unique_key: ['advertiserId','ReportDate','OrderId','LineItemId']
      partition_by: { 'field': 'reportDate', 'data_type': 'date' }
      cluster_by: ['advertiserId','OrderId','LineItemId'] 

  - name: AmazonDSPProductsReport
    config:
      materialized: incremental
      incremental_strategy: merge
      unique_key: ['advertiserId','ReportDate','OrderId','LineItemId']
      partition_by: { 'field': 'reportDate', 'data_type': 'date' }
      cluster_by: ['advertiserId','OrderId','LineItemId'] 
      
  - name: AmazonDSPTechnologyReport
    config:
      materialized: incremental
      incremental_strategy: merge
      unique_key: ['ReportDate','OrderId','LineItemId']
      partition_by: { 'field': 'reportDate', 'data_type': 'date' }
      cluster_by: ['OrderId','LineItemId'] 


```



## Resources:
- Have questions, feedback, or need [help](https://calendly.com/srinivas-janipalli/30min)? Schedule a call with our data experts or email us at info@sarasanalytics.com.
- Learn more about Daton [here](https://sarasanalytics.com/daton/).
- Refer [this](https://youtu.be/6zDTbM6OUcs) to know more about how to create a dbt account & connect to {{Bigquery/Snowflake}}
