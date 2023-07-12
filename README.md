# sbeacon-exploration

Serverless Beacon v2 exploration for UMCCR use cases.

## Demo instance

Our demo instance is available at https://beacon.demo.umccr.org 

Please note that this is for testing only. We may change the service without notice.

## Test dataset

- [CINECA_synthetic_cohort_EUROPE_UK1](./data/CINECA_synthetic_cohort_EUROPE_UK1)
- We ingested this demo Beacon with UMCCR subset [500 individuals](scripts/cineca_uk1_umccr_sample_list.txt) i.e. `HG03872` (row index `1501`) to `NA19059` (row index `2000`) from the original `2504` individuals, see [Excel sheet](data/CINECA_synthetic_cohort_EUROPE_UK1/Beacon-v2-Models_CINECA_UK1MS__UMCCR.xlsx).
- For VCF, we have also lifted to GRCh38.

## sBeacon

For the implementation, we test drive the serverless beacon implementation from [Transformational Bioinformatics](https://bioinformatics.csiro.au/) team at [CSIRO AEHRC](https://aehrc.csiro.au/research/cloud-native-genomics/).
  - https://bioinformatics.csiro.au/serverless-beacon/
  - https://publications.csiro.au/publications/publication/PIcsiro:EP207057
  - https://publications.csiro.au/publications/publication/PIcsiro:EP2022-0079
  - https://publications.csiro.au/publications/publication/PIcsiro:EP2022-5743
  - [Overview Slides](https://docs.google.com/presentation/d/1qV_CTCPEftCGmzzC1dK_s1evBl8BQUB1H8eUoEVIWzU/edit?usp=sharing)

### Setting up sBeacon

- CSIRO sBeacon team has added support to the recent GA4GH [Beacon v2](https://github.com/ga4gh-beacon/beacon-v2) specification.
- We (UMCCR team) are working closely with CSIRO sBeacon team on test-driving this pilot deployment effort.
- Deploy the serverless beacon following the instructions on the sBeacon GitHub repo
  - https://github.com/aehrc/terraform-aws-serverless-beacon
  - The [Postman](https://www.postman.com) collection is provided in [utils](utils/sBeacon.postman_collection.json) can be used for testing the API.
- Create submission JSON for the dataset as [specified](https://github.com/aehrc/terraform-aws-serverless-beacon/tree/dev#data-ingestion-api) for the sBeacon ingestion API
  - We use the Jupyter Notebook in [scripts](scripts) to transform (ETL) the metadata from the Excel file into the [sBeacon submission JSON](scripts/submission_from_excel.json).


## Example queries

The following is an example for scenario-driven Beacon query. 

### Example 1

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

### Example 2

Get available filtering terms for individuals endpoint

```bash
curl -s -X GET 'https://beacon.demo.umccr.org/individuals/filtering_terms' | jq
```

Get all available ontology terms, upto 1000 in this Beacon.
_(Note: You use these terms to perform filter query in following sections)_
```bash
curl -s -X GET 'https://beacon.demo.umccr.org/individuals/filtering_terms?limit=1000' | jq
```

### Example 3

Get how many individuals exist - Count query

```bash
curl -s -X GET 'https://beacon.demo.umccr.org/individuals?requestedGranularity=count' | jq
```

We have made `500` subset of the original CINECA UK1 dataset (total `2504` individuals), see [Excel sheet](data/CINECA_synthetic_cohort_EUROPE_UK1/Beacon-v2-Models_CINECA_UK1MS__UMCCR.xlsx) for details.

```
...
  "responseSummary": {
    "exists": true,
    "numTotalResults": 500
  },
...
```

Get 10 individuals' records

```bash
curl -s -X GET 'https://beacon.demo.umccr.org/individuals?requestedGranularity=record&limit=10' | jq
```

### Example 4

#### Example 4.1 - Filtering with Ontology term - Count query (GET)

* Count how many individual from `Northern Ireland` according to ontology codded term: [GAZ:00002638](https://www.ebi.ac.uk/ols/ontologies/gaz/terms?iri=http%3A%2F%2Fpurl.obolibrary.org%2Fobo%2FGAZ_00002638)

```bash
curl -s -X GET 'https://beacon.demo.umccr.org/individuals?requestedGranularity=count&filters=GAZ:00002638' | jq
```

```
...
  "responseSummary": {
    "exists": true,
    "numTotalResults": 53
  },
...
```

#### Example 4.2 - Filtering with Ontology term - Count query (POST)

```bash
curl -s -X POST 'https://beacon.demo.umccr.org/individuals' \
--header 'Content-Type: application/json' \
--data-raw '{
    "meta": {
        "apiVersion": "v2.0"
    },
    "query": {
        "requestedGranularity": "count",
        "filters": [
            {
                "id": "GAZ:00002638"
            }
        ]
    }
}' | jq
```

#### Example 4.3 - Filtering with Ontology term - Record query (GET)

```bash
curl -s -X GET 'https://beacon.demo.umccr.org/individuals?requestedGranularity=record&filters=GAZ:00002638&limit=2' | jq
```

#### Example 4.4 - Filtering with Ontology term - Record query (POST)

```bash
curl -s -X POST 'https://beacon.demo.umccr.org/individuals' \
--header 'Content-Type: application/json' \
--data-raw '{
    "meta": {
        "apiVersion": "v2.0"
    },
    "query": {
        "requestedGranularity": "record",
        "filters": [
            {
                "id": "GAZ:00002638"
            }
        ],
        "pagination": {
            "limit": 2,
            "skip": 0
        }
    }
}' | jq
```

#### Example 4.5 - Ontology Auto-expansion

Get all individuals from the higher order ontology term `British Isles` (i.e. [EBI/OLS - GAZ:00001505](https://www.ebi.ac.uk/ols/ontologies/gaz/terms?iri=http%3A%2F%2Fpurl.obolibrary.org%2Fobo%2FGAZ_00001505)). This makes use of ontology expansion to fetch all individuals with geographic origins in England, Wales, Scotland, Ireland, etc.

```bash
curl -s -X GET 'https://beacon.demo.umccr.org/individuals?requestedGranularity=count&filters=GAZ:00001505' | jq
```

_...we have about 304 individuals that match the query..!_

```
...
  "responseSummary": {
    "exists": true,
    "numTotalResults": 304
  },
...  
```

```bash
curl -s -X POST 'https://beacon.demo.umccr.org/individuals' \
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

#### Example 4.6 - Give me Individual having particular disesa code

Say individual with `asthma` [MONDO](https://www.ebi.ac.uk/ols/ontologies/mondo/terms?iri=http%3A%2F%2Fpurl.obolibrary.org%2Fobo%2FMONDO_0004979) disease code `MONDO:0004979`

```bash
curl -s -X GET 'https://beacon.demo.umccr.org/individuals?requestedGranularity=count&filters=MONDO:0004979' | jq
```

```
...
  "responseSummary": {
    "exists": true,
    "numTotalResults": 19
  },
...
```

```bash
curl -s -X GET 'https://beacon.demo.umccr.org/individuals?requestedGranularity=record&filters=MONDO:0004979' | jq
```

_...some jq magic..!_

```bash
curl -s -X GET 'https://beacon.demo.umccr.org/individuals?requestedGranularity=record&filters=MONDO:0004979' | jq '.response .resultSets[] .results'
```

_...show all individual id..!_

```bash
curl -s -X GET 'https://beacon.demo.umccr.org/individuals?requestedGranularity=record&filters=MONDO:0004979&limit=100' | jq '.response .resultSets[] .results[] .id'
```

### Example 5

Variant searching, if there exists any Genomic Variant `C > G` SNP at specified Chromosome 5 region.

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
            "assemblyId": "GRCh38",
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

_...search response return 2 matches found_ 
```
...
 "response": {
    "resultSets": [
      {
        "id": "",
        "setType": "",
        "exists": true,
        "resultsCount": 2,
        "results": [
          {
            "variantInternalId": "R1JDaDM4CWNocjUJMTAwMDAwMTIJQwlH",
            "variation": {
              "referenceBases": "C",
              "alternateBases": "G",
              "location": {
                "interval": {
                  "start": {
                    "type": "Number",
                    "value": 10000012
                  },
                  "end": {
                    "type": "Number",
                    "value": 10000013
                  },
                  "type": "SequenceInterval"
                },
                "sequence_id": "GRCh38",
                "type": "SequenceLocation"
              },
              "variantType": "SNP"
            }
          },
          {
            "variantInternalId": "R1JDaDM4CWNocjUJMTAwMDA5NTkJQwlH",
            "variation": {
              "referenceBases": "C",
              "alternateBases": "G",
              "location": {
                "interval": {
                  "start": {
                    "type": "Number",
                    "value": 10000959
                  },
                  "end": {
                    "type": "Number",
                    "value": 10000960
                  },
                  "type": "SequenceInterval"
                },
                "sequence_id": "GRCh38",
                "type": "SequenceLocation"
              },
              "variantType": "SNP"
            }
          }
        ],
        "resultsHandover": null
      }
    ]
  },
...
```

**NOTE:** 
- Please take note one of response Variant Internal ID e.g. `R1JDaDM4CWNocjUJMTAwMDAwMTIJQwlH`
- This `variantInternalId` is encoded in Base64. So, you can go to https://www.base64decode.org and decode it, like so.
```
GRCh38	chr5	10000012	C	G
```

Get details of the variant record:

```
curl -s -X GET 'https://beacon.demo.umccr.org/g_variants/R1JDaDM4CWNocjUJMTAwMDAwMTIJQwlH?requestedGranularity=record' | jq
```

### Example 6

- From Example 5 query, there exists SNP at specified location. 
- Find all individuals that correspond to this SNP variant.

```
curl -s -X POST 'https://beacon.demo.umccr.org/g_variants/R1JDaDM4CWNocjUJMTAwMDAwMTIJQwlH/individuals' \
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

#### Example 6.1

- Consider further **filter this SNP variant with individuals from the `British Isles` background** (ref: Example 4, GAZ ontology term ID)

```
curl -s -X POST 'https://beacon.demo.umccr.org/g_variants/R1JDaDM4CWNocjUJMTAwMDAwMTIJQwlH/individuals' \
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

#### Example 6.2

- Or filter this SNP variant with individuals related to `asthma` [MONDO](https://www.ebi.ac.uk/ols/ontologies/mondo/terms?iri=http%3A%2F%2Fpurl.obolibrary.org%2Fobo%2FMONDO_0004979) disease code `MONDO:0004979`

```
curl -s -X POST 'https://beacon.demo.umccr.org/g_variants/R1JDaDM4CWNocjUJMTAwMDAwMTIJQwlH/individuals' \
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
                "id": "MONDO:0004979"
            }
        ]
    }
}
' | jq
```
