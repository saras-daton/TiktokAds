# Tiktok Advertising Data Unification

This dbt package is for Data Unification of Amazon Advertising ingested data by [Daton](https://sarasanalytics.com/daton/). [Daton](https://sarasanalytics.com/daton/) is the Unified Data Platform for Global Commerce with 100+ pre-built connectors and data sets designed for accelerating the eCommerce data and analytics journey by [Saras Analytics](https://sarasanalytics.com).

### Supported Datawarehouses:
- BigQuery
- Snowflake

#### Typical challenges with raw data are:
- Array/Nested Array columns which makes queries for Data Analytics complex
- Data duplication due to look back period while fetching report data from Tiktok
- Separate tables at marketplaces/Store, brand, account level for same kind of report/data feeds

Data Unification simplifies Data Analytics by doing:
- Consolidation - Different marketplaces/Store/account & different brands would have similar raw Daton Ingested tables, which are consolidated into one table with column distinguishers brand & store
- Deduplication - Based on primary keys, the data is De-duplicated and the latest records are only loaded into the consolidated stage tables
- Incremental Load - Models are designed to include incremental load which when scheduled would update the tables regularly
- Standardization -
	- Currency Conversion (Optional) - Raw Tables data created at Marketplace/Store/Account level may have data in local currency of the corresponding marketplace/store/account. Values that are in local currency are Standardized by converting to desired currency using Daton Exchange Rates data.
	  Prerequisite - Exchange Rates connector in Daton needs to be present - Refer [this](https://github.com/saras-daton/currency_exchange_rates)
	- Time Zone Conversion (Optional) - Raw Tables data created at Marketplace/Store/Account level may have data in local timezone of the corresponding marketplace/store/account. DateTime values that are in local timezone are Standardized by converting to specified timezone using input offset hours.

#### Prerequisite 
Daton Integrations for  
- Tiktok Ads 
- Exchange Rates(Optional, if currency conversion is not required)

*Note:* 
*Please select 'Do Not Unnest' option while setting up Daton Integrataion*

# Installation & Configuration

## Installation Instructions

If you haven't already, you will need to create a packages.yml file in your DBT project. Include this in your `packages.yml` file

```yaml
packages:
  - package: saras-daton/TiktokAds
    version: v1.0.0
```

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
  tiktok_ads:
    TiktokAds:
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

### Timezone Conversion 

To enable timezone conversion, which converts the timezone columns from UTC timezone to local timezone, please mark the timezone_conversion_flag as True in the dbt_project.yml file, by default, it is False.
Additionally, you need to provide offset hours between UTC and the timezone you want the data to convert into for each raw table for which you want timezone converison to be taken into account.

Example:
```yaml
vars:
timezone_conversion_flag: False
raw_table_timezone_offset_hours: {
    "Tiktok.Ads.SPONSOREDBRANDS_TIKTOKADS_AD_GROUP_REPORT_HOURLY":-6,
    "Tiktok.Ads.SPONSOREDBRANDS_TIKTOKADS_AD_REPORT_HOURLY":-6, 
    "Tiktok.Ads.SPONSOREDBRANDS_TIKTOKADS_CAMPAIGN_REPORT_HOURLY":-6,
    }
```
Here, -7 represents the offset hours between UTC and PDT considering we are sitting in PDT timezone and want the data in this timezone

### Table Exclusions

If you need to exclude any of the models, declare the model names as variables and mark them as False. Refer the table below for model details. By default, all tables are created.

Example:
```yaml
vars:
SponsoredBrands_AdGroupsReport: False
```

## Models

This package contains models from the Amazon Advertising API which includes reports on {{sales, margin, inventory, product}}. The primary outputs of this package are described below.

| **Category**                 | **Model**  | **Description** |
| ------------------------- | ---------------| ----------------------- |
|Sponsored Brands | [TiktokAds_Reservation_campaign_report_daily](models/TiktokAds/TiktokAds_Reservation_campaign_report_daily.sql)  | A list of campaigns perfromance reports at day level |
|Sponsored Brands | [TiktokAds_ad_age_gender](models/TiktokAds/TiktokAds_ad_age_gender.sql)  | A list of Ads performance at age and gender level |
|Sponsored Brands | [TiktokAds_ad_country](models/TiktokAds/TiktokAds_ad_country.sql)  | A list of ads performance at country level |
|Sponsored Brands | [TiktokAds_ad_group_report_daily](models/TiktokAds/TiktokAds_ad_group_report_daily.sql)| A list of Ad groups performance at country level |
|Sponsored Brands | [TiktokAds_ad_group_report_hourly](models/TiktokAds/TiktokAds_ad_group_report_hourly.sql)| A list of Ad groups performance at hourly level |
|Sponsored Brands | [TiktokAds_ad_language](models/TiktokAds/TiktokAds_ad_language.sql)| A list of Ads performance at Language level |
|Sponsored Brands | [TiktokAds_ad_platform](models/TiktokAds/TiktokAds_ad_platform.sql)| A list of Ads performance at Platform level |
|Sponsored Brands | [TiktokAds_ad_report_daily](models/TiktokAds/TiktokAds_ad_report_daily.sql)| A list of Ads perfromance reports at day level |
|Sponsored Brands | [TiktokAds_ad_report_hourly](models/TiktokAds/TiktokAds_ad_report_hourly.sql)| A list of Ads perfromance reports at hourly level |
|Sponsored Brands | [TiktokAds_campaign_age_gender](models/TiktokAds/TiktokAds_campaign_age_gender.sql)| A list of campaign performance at age and gender level |
|Sponsored Brands | [TiktokAds_campaign_country](models/TiktokAds/TiktokAds_campaign_country.sql)| A list of campaign performance at country level |
|Sponsored Brands | [TiktokAds_campaign_language](models/TiktokAds/TiktokAds_campaign_language.sql)| A list of campaign performance at Language level |
|Sponsored Brands | [TiktokAds_campaign_platform](models/TiktokAds/TiktokAds_campaign_platform.sql)| A list of campaign performance at Platform level |
|Sponsored Brands | [TiktokAds_campaign_report_daily](models/TiktokAds/TiktokAds_campaign_report_daily.sql)| A list of campaign perfromance reports at day level |
|Sponsored Brands | [TiktokAds_campaign_report_hourly](models/TiktokAds/TiktokAds_campaign_report_hourly.sql)| A list of campaign perfromance reports at hourly level |




### For details about default configurations for Table Primary Key columns, Partition columns, Clustering columns, please refer the properties.yaml used for this package as below. 
	You can overwrite these default configurations by using your project specific properties yaml.
```yaml
version: 2
models:
  - name: TiktokAds_Reservation_campaign_report_daily
    config:
      materialized: incremental
      incremental_strategy: merge
      unique_key: ['Date', 'CampaignID', 'Campaignname', 'AccountName']
      partition_by: { 'field': 'Date', 'data_type': 'date' }
      cluster_by: ['Date', 'CampaignID', 'Campaignname', 'AccountName'] 

  - name: TiktokAds_ad_age_gender
    config:
      materialized: incremental
      incremental_strategy: merge
      unique_key: ['Date', 'AdID', 'CampaignID', 'Campaignname']
      partition_by: { 'field': 'Date', 'data_type': 'date' }
      cluster_by: ['Date', 'AdID', 'CampaignID', 'Campaignname'] 

  - name: TiktokAds_ad_country
    config:
      materialized: incremental
      incremental_strategy: merge
      unique_key: ['Date', 'AdID', 'Region', 'CampaignID']
      partition_by: { 'field': 'Date', 'data_type': 'date' }
      cluster_by: ['AdID', 'Date', 'CampaignID'] 

  - name: TiktokAds_ad_group_report_daily
    config:
      materialized: incremental
      incremental_strategy: merge
      unique_key: ['Date', 'AdgroupID', 'CampaignID']
      partition_by: { 'field': 'Date', 'data_type': 'date' }
      cluster_by: ['Date', 'AdgroupID', 'CampaignID'] 

  - name: TiktokAds_ad_group_report_hourly
    config:
      materialized: incremental
      incremental_strategy: merge
      unique_key: ['Time', 'AdgroupID', 'CampaignID']
      partition_by: { 'field': 'Time', 'data_type': 'timestamp', 'granularity': 'day' }
      cluster_by: [ 'Time', 'AdgroupID','CampaignID'] 

  - name: TiktokAds_ad_language
    config:
      materialized: incremental
      incremental_strategy: merge
      unique_key: ['Date', 'AdID', 'Language', 'CampaignID']
      partition_by: { 'field': 'Date', 'data_type': 'date' }
      cluster_by: ['Date', 'AdID', 'Language', 'CampaignID'] 

  - name: TiktokAds_ad_platform
    config:
      materialized: incremental
      incremental_strategy: merge
      unique_key: ['Date', 'AdID', 'OperatingSystem']
      partition_by: { 'field': 'Date', 'data_type': 'date' }
      cluster_by: ['Date', 'AdID', 'OperatingSystem'] 

  - name: TiktokAds_ad_report_daily
    config:
      materialized: incremental
      incremental_strategy: merge
      unique_key: ['Date', 'AdID', 'CampaignID']
      partition_by: { 'field': 'Date', 'data_type': 'date' }
      cluster_by: ['Date', 'AdID', 'CampaignID'] 

  - name: TiktokAds_ad_report_hourly
    config:
      materialized: incremental
      incremental_strategy: merge
      unique_key: ['Time', 'AdID', 'CampaignID']
      partition_by: { 'field': 'Time', 'data_type': 'timestamp', 'granularity': 'day' }
      cluster_by: ['Time', 'AdID', 'CampaignID'] 

  - name: TiktokAds_campaign_age_gender
    config:
      materialized: incremental
      incremental_strategy: merge
      unique_key: ['Date', 'CampaignID', 'Age', 'Gender']
      partition_by: { 'field': 'Date', 'data_type': 'date' }
      cluster_by: ['Date', 'CampaignID', 'Age', 'Gender'] 

  - name: TiktokAds_campaign_country
    config:
      materialized: incremental
      incremental_strategy: merge
      unique_key: ['Date', 'CampaignID', 'Campaignname', 'AccountName']
      partition_by: { 'field': 'Date', 'data_type': 'date' }
      cluster_by: ['Date', 'CampaignID'] 

  - name: TiktokAds_campaign_language
    config:
      materialized: incremental
      incremental_strategy: merge
      unique_key: ['Date', 'CampaignID', 'Language']
      partition_by: { 'field': 'Date', 'data_type': 'date' }
      cluster_by: ['Date', 'CampaignID', 'Language'] 

  - name: TiktokAds_campaign_platform
    config:
      materialized: incremental
      incremental_strategy: merge
      unique_key: ['Date', 'CampaignID', 'OperatingSystem']
      partition_by: { 'field': 'Date', 'data_type': 'date' }
      cluster_by: ['Date', 'CampaignID', 'OperatingSystem'] 

  - name: TiktokAds_campaign_report_daily
    config:
      materialized: incremental
      incremental_strategy: merge
      unique_key: ['Date', 'CampaignID', 'Campaignname']
      partition_by: { 'field': 'Date', 'data_type': 'date' }
      cluster_by: ['Date', 'CampaignID', 'Campaignname'] 

  - name: TiktokAds_campaign_report_hourly
    config:
      materialized: incremental
      incremental_strategy: merge
      unique_key: ['Time', 'CampaignID', 'Campaignname']
      partition_by: { 'field': 'Time', 'data_type': 'timestamp', 'granularity': 'day' }
      cluster_by: ['Time', 'CampaignID', 'Campaignname'] 
   
```



## Resources:
- Have questions, feedback, or need [help](https://calendly.com/srinivas-janipalli/30min)? Schedule a call with our data experts or email us at info@sarasanalytics.com.
- Learn more about Daton [here](https://sarasanalytics.com/daton/).
- Refer [this](https://youtu.be/6zDTbM6OUcs) to know more about how to create a dbt account & connect to {{Bigquery/Snowflake}}