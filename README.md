# sbeacon-exploration

Serverless Beacon v2 exploration for UMCCR use cases.

## Test dataset

- [CINECA_synthetic_cohort_EUROPE_UK1](./data/CINECA_synthetic_cohort_EUROPE_UK1)

## AEHRC sBeacon

For the implementation, we test drive the serverless beacon implementation from [CSIRO AEHRC](https://aehrc.csiro.au/research/cloud-native-genomics/).
  - https://publications.csiro.au/publications/publication/PIcsiro:EP207057
  - https://publications.csiro.au/publications/publication/PIcsiro:EP2022-0079

### Procedure

- AEHRC sBeacon team has added support to the recent [Beacon v2 standard](https://github.com/ga4gh-beacon/beacon-v2).
- We (UMCCR team) is working closely with AEHRC sBeacon team on test-driving this pilot deployment effort.
- This development activity is happening in `dev` branch of sBeacon GitHub repo.
- Deploy the serverless beacon following the instructions on the sBeacon GitHub [repo](https://github.com/aehrc/terraform-aws-serverless-beacon/tree/dev)
  - The [Postman](https://www.postman.com) collection in `utils` can be used for testing the API
- Create submission JSON for the dataset as [specified](https://github.com/aehrc/terraform-aws-serverless-beacon/tree/dev#data-ingestion-api) for the sBeacon ingestion API
  - We use the Jupyter Notebook in `scripts` to transform (ETL) the metadata from the Excel file into the sBeacon submission JSON
