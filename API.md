# API

## Overview

The server distinguishes between a **private** (the user is logged-in) and **public** (the user is not logged-in) access. Many information (including most of our metadata for Switzerland) is only available with a private access. The API endpoints are the same but an empty list will be returned if no public information is available.



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


### Sample

Returns a list of samples that matches the query. The maximal number of returned results is limited at 1000.

**Request:**

```
GET /resource/sample
Request params:
  - country: string
  - mutations: string, comma-separated (required)
  - matchPercentage: float (default: 1)
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

`data.metadata` is always **private**.


### Variant

Returns a list of all known variants.

**Request:**

```
GET /resource/variant
```

**Response:**

```
[
  {
    name: string,
    mutations: [string]
  }
]
```



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
      zip_code: string
    },
    y: {
      count: integer
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
    t0Proportions: float,
    t1Proportions: float,
    absoluteDifferenceProportion: float,
    relativeDifferenceProportion: float
  }
]
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
