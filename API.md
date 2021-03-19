# API

## Overview

The server distinguishes between a **private** (the user is logged-in) and **public** (the user is not logged-in) access. Many information (including most of our metadata for Switzerland) is only available with a private access. The API endpoints are the same but an empty list will be returned if no public information is available.


### Authentication

JSON Web Tokens (JWT) are used for authentication. Upon login (see below), a token will be provided. The token has to be sent with every subsequent request. There are two ways to send the token:

* In the request header: `Authorization: Bearer <token>` (the preferred way)
* In the query params: `?jwt=<token>`


## Resources

### Country

Returns a list of all country names.

**Request:**

```
GET /resource/country
```

**Response:**

```
[string]
```


### Region

Returns a list of all region (continent) names.

**Request:**

```
GET /resource/region
```

**Response:**

```
[string]
```


### Sample (new)

Returns a list of samples. This endpoint will soon supersede `/resource/sample` and all the `/plot/variant/*` endpoints. Then, it will be renamed to `/resource/sample`.

**Request:**

```
GET /resource/sample2
Request params:
  - region: string
  - country: string
  - mutations: string, comma-separated (required)
  - matchPercentage: float (default: 1)
  - dataType: string (possible values: "SURVEILLANCE")
```

**Response:**

```
[
  {
    date: Date,
    region: string,
    country: string,
    division?: string,
    zipCode?: string,
    ageGroup?: string,
    hospitalized?: boolean,
    deceased?: boolean,
    count: integer
  }
]
```

If a value is not provided, it is either unknown or **private**. `count` gives the number of instances with the same attributes.



### Sample

Returns a list of samples that matches the query. The maximal number of returned results is limited at 1000.

**Request:**

```
GET /resource/sample
Request params:
  - country: string
  - mutations: string, comma-separated (required)
  - matchPercentage: float (default: 1)
  - dataType: string (possible values: "SURVEILLANCE")
```

**Response:**

```
{
  total: integer,
  data: [
    {
      name: string,
      country: string,
      date: Date,
      mutations: [string],
      metadata: {
        country: string,
        division, string,
        location: string,
        zipCode: string,
        host: string,
        age: integer,
        sex: string
      }
    }
  ]
}
```

`data.metadata` is always **private** (i.e., `null` for **public** access). `data.mutations` is only **public** for samples submitted by selected labs. 


### Sample: Sequences in Fasta Format

Returns the sequences corresponding to the results of `GET /resource/sample` in the fasta format. The maximal number of returned results is also limited at 1000.

**Request:**

```
GET /resource/sample-fasta
Request params:
  - country: string
  - mutations: string, comma-separated (required)
  - matchPercentage: float (default: 1)
  - dataType: string (possible values: "SURVEILLANCE")
```

**Response:**

```
string
```

The endpoint is **private**.



## Plots

### Variant: Age Distribution

Returns the age distribution of a variant. The ages are grouped (e.g., "10-19", "20-30"). For Switzerland, most of the age information is **private**.

**Request:**

```
GET /plot/variant/age-distribution
Request params:
  - country: string (required)
  - mutations: string, comma-separated (required)
  - matchPercentage: float (default: 1)
  - dataType: string (possible values: "SURVEILLANCE")
```

**Response:**

```
[
  {
    x: string,
    y: {
      count: integer,
      total: integer,
      proportion: {
        value: float,
        ciLower: float,
        ciUpper: float,
        confidenceLevel: float
      }
    }
  }
]
```


### Variant: Time Distribution

Returns the time distribution.

**Request:**

```
GET /plot/variant/time-distribution
Request params:
  - country: string (required)
  - mutations: string, comma-separated (required)
  - matchPercentage: float (default: 1)
  - dataType: string (possible values: "SURVEILLANCE")
```

**Response:**

```
[
  {
    x: {
      yearWeek: string (<iso year>-<iso week>),
      firstDayInWeek: Date
    },
    y: {
      count: integer,
      total: integer,
      proportion: {
        value: float,
        ciLower: float,
        ciUpper: float,
        confidenceLevel: float
      }
    }
  }
]
```


### Variant: Time Distribution (International)

Returns the time distribution for all countries.

**Request:**

```
GET /plot/variant/international-time-distribution
Request params:
  - mutations: string, comma-separated (required)
  - matchPercentage: float (default: 1)
```

**Response:**

```
[
  {
    x: {
      country: string,
      week: {
        yearWeek: string (<iso year>-<iso week>),
        firstDayInWeek: Date
      }
    },
    y: {
      count: integer,
      total: integer,
      proportion: {
        value: float,
        ciLower: float,
        ciUpper: float,
        confidenceLevel: float
      }
    }
  }
]
```


### Variant: Time-Zip Code Distribution

Returns the number of samples of a variant per zip code area and week. Zip code information is only available for Switzerland and only for **private** access.

**Request:**

```
GET /plot/variant/time-zip-code-distribution
Request params:
  - country: string (required)
  - mutations: string, comma-separated (required)
  - matchPercentage: float (default: 1)
  - dataType: string (possible values: "SURVEILLANCE")
```

**Response:**

```
[
  {
    x: {
      week: {
        yearWeek: string (<iso year>-<iso week>),
        firstDayInWeek: Date
      },
      zipCode: string
    },
    y: {
      count: integer
    }
  }
]
```


## Sequencing Intensity Through Time

Returns the number of confirmed cases and the number of available sequences of a country. The number of cases is from the OWID dataset.

Usually, `numberCases` should be larger than `numberSequenced` - but no guarantees!

**Request:**

```
GET /plot/sequencing/time-intensity-distribution
Request params:
  - country: string (required)
  - dataType: string (possible values: "SURVEILLANCE")
```

**Response:**

```
[
  {
    x: {
      yearWeek: string (<iso year>-<iso week>),
      firstDayInWeek: Date
    },
    y: {
      numberCases: integer,
      numberSequenced: integer
    }
  }
]
```



## Computed

### Growing Variants

Returns a list of variants what that experienced a high relative growth compared to the previous week.

**Request:**

```
GET /computed/find-growing-variants
Request params:
  - year: integer (required)
  - week: integer (required)
  - country: string (required)
```

**Response:**

```
[
  {
    variant: {
      name: null,
      mutations: [string]
    },
    t0Count: integer,
    t1Count: integer,
    t0Proportion: float,
    t1Proportion: float,
    absoluteDifferenceProportion: float,
    relativeDifferenceProportion: float
  }
]
```


#### Model: chen2021Fitness

Based on Chen et al. (2021): "Quantification of the spread of SARS-CoV-2 variant B.1.1.7 in Switzerland"

**Request:**

```
GET /computed/model/chen2021Fitness
Request params:
  - country: string (required)
  - mutations: string, comma-separated (required)
  - matchPercentage: float (default: 1)
  - dataType: string (possible values: "SURVEILLANCE")
  - alpha: float (default: 0.95)
  - generationTime: float (default: 4.8)
  - reproductionNumberWildtype: float (default: 1)
  - plotStartDate: date
  - plotEndDate: date (required)
  - initialWildtypeCases: integer (default: 1000)
  - initialVariantCases: integer (default: 100)
```

**Response:**

```
{
  daily: {
    t: [date],
    proportion: [float],
    ciLower: [float],
    ciUpper: [float]
  },
  params: {
    a: {
      value: float,
      ciLower: float,
      ciUpper: float
    },
    t0: {
      value: float,
      ciLower: float,
      ciUpper: float
    },
    fc: {
      value: float,
      ciLower: float,
      ciUpper: float
    },
    fd: {
      value: float,
      ciLower: float,
      ciUpper: float
    }
  },
  plotAbsoluteNumbers: {
    t: [date],
    variantCases: [integer],
    wildtypeCases: [integer]
  },
  plotProportion: {
    t: [date],
    proportion: [float],
    ciLower: [float],
    ciUpper: [float]
  }
}
```



## Utils

### Current Week

Returns today's isoweek.

**Request:**

```
GET /utils/currentweek
```

**Response:**

```
integer
```



## Internal

### Login

Returns a JWT token if the submitted credentials are valid.

**Request:**

```
POST /internal/login
Request body:
  {
    username: string,
    password: string
  }
```

**Response:**

```
{
  token: string
}
```


### Temporary JWT Token

Returns a JWT token with a TTL of 3 minutes that can be used to authenticate access to a particular endpoint.

**Request:**

```
POST /internal/create-temporary-jwt
Request params:
  - restrictionEndpoint: string (required)
```

**Response**

```
{
  token: string
}
```

The endpoint is **private**.
