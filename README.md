# sbeacon-exploration

Serverless Beacon v2 exploration for UMCCR use cases.


## Test dataset

For exploration we use the CINECA test data prepared by the EGA folks [here][ega_cineca_repo].



## Beacon v2 implementation

For the implementation we decided to test drive the serverless beacon from [CSIRO][csiro_home], available [here][sbeacon_repo].



## Procedure


- deploy the serverless beacon following the instructions on the sBeacon GH [repo][sbeacon_repo]
  - the [Postman][postman_home] collection in `utils` can be used for testing the API
- create submission JSON for the dataset as [specified][sbeacon_submission_json] for the sBeacon ingestion API
  - we use the jupiter notebook in `scripts` to transform the metadata in the Excel file into the sBeacon submission JSON



[ega_cineca_repo]: https://github.com/EGA-archive/beacon2-ri-tools/tree/main/CINECA_synthetic_cohort_EUROPE_UK1
[csiro_home]: https://www.csiro.au/
[sbeacon_repo]: https://github.com/aehrc/terraform-aws-serverless-beacon/tree/dev
[sbeacon_submission_json]: https://github.com/aehrc/terraform-aws-serverless-beacon/tree/dev#data-ingestion-api
[postman_home]: https://www.postman.com/