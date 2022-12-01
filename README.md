# sbeacon-exploration

Serverless Beacon v2 exploration for UMCCR use cases.

## Demo instance

Our demo instance is available at https://beacon.demo.umccr.org 

Please note that this is for testing only. We may change the service without notice.

## Test dataset

- [CINECA_synthetic_cohort_EUROPE_UK1](./data/CINECA_synthetic_cohort_EUROPE_UK1)

## sBeacon

For the implementation, we test drive the serverless beacon implementation from [Transformational Bioinformatics](https://bioinformatics.csiro.au/) team at [CSIRO AEHRC](https://aehrc.csiro.au/research/cloud-native-genomics/).
  - https://bioinformatics.csiro.au/serverless-beacon/
  - https://publications.csiro.au/publications/publication/PIcsiro:EP207057
  - https://publications.csiro.au/publications/publication/PIcsiro:EP2022-0079
  - https://docs.google.com/presentation/d/1oIw76fhR6u97SWVy_a6QoSBRbCbQ5oGUf0Njwao3I6g/edit?usp=sharing

### Setting up sBeacon

- CSIRO sBeacon team has added support to the recent GA4GH [Beacon v2](https://github.com/ga4gh-beacon/beacon-v2) specification.
- We (UMCCR team) are working closely with CSIRO sBeacon team on test-driving this pilot deployment effort.
- This development activity is happening in `dev` branch of sBeacon GitHub repo.
- Deploy the serverless beacon following the instructions on the sBeacon GitHub repo
  - https://github.com/aehrc/terraform-aws-serverless-beacon/tree/dev
  - The [Postman](https://www.postman.com) collection is provided in [utils](utils/sBeacon.postman_collection.json) can be used for testing the API.
- Create submission JSON for the dataset as [specified](https://github.com/aehrc/terraform-aws-serverless-beacon/tree/dev#data-ingestion-api) for the sBeacon ingestion API
  - We use the Jupyter Notebook in [scripts](scripts) to transform (ETL) the metadata from the Excel file into the [sBeacon submission JSON](scripts/submission_from_excel.json).


## Example queries

The following is an example for scenario-driven Beacon query. 

### Example1

Get service info

```bash
curl -s -X GET 'https://beacon.demo.umccr.org/info' | jq
```

Get service map

```bash
curl -s -X GET 'https://beacon.demo.umccr.org/map' | jq
```

Get service configuration

```bash
curl -s -X GET 'https://beacon.demo.umccr.org/configuration' | jq
```

### Example2

Get all individuals

```bash
curl -s -X GET 'https://beacon.demo.umccr.org/individuals?requestedGranularity=record&limit=10' | jq
```

### Example3

Get all available filtering terms for individuals endpoint

```bash
curl -s -X GET 'https://beacon.demo.umccr.org/individuals/filtering_terms' | jq
```

### Example4

Get all individuals from the British Isles (i.e. [EBI/OLS - GAZ:00001505](https://www.ebi.ac.uk/ols/ontologies/gaz/terms?iri=http%3A%2F%2Fpurl.obolibrary.org%2Fobo%2FGAZ_00001505)). This makes use of ontology expansion to fetch individuals with geographic origins in England, Wales, Scotland, Ireland, etc.

```bash
curl -s -X POST 'https://beacon.demo.umccr.org/individuals?requestedGranularity=record' \
--header 'Content-Type: application/json' \
--data-raw '{
    "meta": {
        "apiVersion": "v2.0"
    },
    "query": {
        "requestedGranularity": "record",
        "filters": [
            {
                "id": "GAZ:00001505"
            }
        ],
        "pagination": {
            "limit": 100,
            "skip": 0
        }
    }
}' | jq
```

### Example5

Search, if there exists any Genomic Variant `C > G` SNP at specified Chromosome 5 region.

```bash
curl -s -X POST 'https://beacon.demo.umccr.org/g_variants' \
--header 'Content-Type: application/json' \
--data-raw '{
    "meta": {
        "apiVersion": "v2.0"
    },
    "query": {
        "includeResultsetResponses": "HIT",
        "requestedGranularity": "record",
        "requestParameters": {
            "assemblyId": "GRCh37",
            "start": [
                10000000,
                10001000
            ],
            "end": [
                10000000,
                10001000
            ],
            "referenceBases": "C",
            "referenceName": "5",
            "alternateBases": "G"
        }
    }
}' | jq
```

**NOTE:** Please take note of the Beacon response Variant Internal ID i.e. `"R1JDaDM3CTUJMTAwMDAxMjQJQwlH"`

```
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "info": {},
  "meta": {
    "beaconId": "sbeacon.umccr.org",
    "apiVersion": "v2.0.0",
    "returnedSchemas": [
      {
        "entityType": "info",
        "schema": "beacon-map-v2.0.0"
      }
    ],
    "returnedGranularity": "record",
    "receivedRequestSummary": {
      "apiVersion": "v2.0.0",
      "requestedSchemas": [],
      "pagination": {
        "limit": 100,
        "skip": 0
      },
      "requestedGranularity": "record"
    }
  },
  "response": {
    "resultSets": [
      {
        "exists": true,
        "id": "redacted",
        "results": [
          {
            "variantInternalId": "R1JDaDM3CTUJMTAwMDAxMjQJQwlH",
            "variation": {
              "referenceBases": "C",
              "alternateBases": "G",
              "location": {
                "interval": {
                  "start": {
                    "type": "Number",
                    "value": 10000124
                  },
                  "end": {
                    "type": "Number",
                    "value": 10000125
                  },
                  "type": "SequenceInterval"
                },
                "sequence_id": "GRCh37",
                "type": "SequenceLocation"
              },
              "variantType": "SNP"
            }
          }
        ],
        "resultsCount": 1,
        "resultsHandovers": [],
        "setType": "genomicVariant"
      }
    ]
  },
  "responseSummary": {
    "exists": true,
    "numTotalResults": 1
  }
}
```

### Example6

- From Example5 query, there exists SNP at specified location. 
- Find all individuals that correspond to this SNP variant.

```
curl -s -X POST 'https://beacon.demo.umccr.org/g_variants/R1JDaDM3CTUJMTAwMDAxMjQJQwlH/individuals' \
--header 'Content-Type: application/json' \
--data-raw '{
    "meta": {
        "apiVersion": "v2.0"
    },
    "query": {
        "requestedGranularity": "record",
        "pagination": {
            "limit": 100,
            "skip": 0
        },
        "filters": []
    }
}' | jq
```

#### Example6.1

- Consider further **filter this SNP variant with individuals from the British Isles background** (ref: Example4, GAZ ontology term ID)

```
curl -s -X POST 'https://beacon.demo.umccr.org/g_variants/R1JDaDM3CTUJMTAwMDAxMjQJQwlH/individuals' \
--header 'Content-Type: application/json' \
--data-raw '{
    "meta": {
        "apiVersion": "v2.0"
    },
    "query": {
        "requestedGranularity": "record",
        "pagination": {
            "limit": 100,
            "skip": 0
        },
        "filters": [
            {
                "id": "GAZ:00001505"
            }            
        ]        
    }
}
' | jq
```

#### Example6.2

- Or consider **filter this SNP variant with individuals related to Asthma**; ICD disease code `ICD10:J45`

```
curl -s -X POST 'https://beacon.demo.umccr.org/g_variants/R1JDaDM3CTUJMTAwMDAxMjQJQwlH/individuals' \
--header 'Content-Type: application/json' \
--data-raw '{
    "meta": {
        "apiVersion": "v2.0"
    },
    "query": {
        "requestedGranularity": "record",
        "pagination": {
            "limit": 100,
            "skip": 0
        },
        "filters": [
            {
                "id": "ICD10:J45"
            }
        ]
    }
}
' | jq
```
