---
title: PubSub API - Subscribing to Data Streams
alias: docs/api/pubsub.html
---

# The Pubsub Interface

InfluxDB's PubSub interface provides the ability to receive specific data at the call of its user. A user will typically make a subscription, which specifies the data that is desired, and then query the subscription as time goes on. Querying the subscriptions returns all of the new, specified data within the database since the last call to query the subscriptions. 


## Tools with Subscriptions

### The JSON Parameters

The requests to make subscriptions and follow a query require a JSON body in the POST request. Here these parameters are described.

**columns** - a list of column names to extract from the corresponding tablenames

**tablenames** -  a list of TimeSeries names from which the data will be taken
- When following a query this is a single string rather than a list

**duration** - in minutes, the amount of time a subscription will last without being queried before expiring. If a value of 0 is posted, the default of 10 minutes will be used.

**interval** - in seconds, the amount of time between queries during a query follow request

**startTm** - the earliest that data will be drawn from

**endTm** - the latest timestamp returned data can have

```json
[
  {
    "columns": ["value", "time", "tx"]
    "tablenames": ["zoo0", "July14", "austin"]
    "duration": 10
    "startTm": "2014-01-05 13:00:00"
    "endTm": "2014-01-01 01:55:55"
  }
]
```

Based on their indices, each entry to columns corresponds to an entry in tables. This information is used in the form of "select {columns} from {tablename} where time > {startTm} and time < {endTm}.
The basic idea is that each call to query subscriptions will look up the column

### Make Subscriptions

To make a subscription, certain parameters are needed for the POST request. If a subscription is not queried within the timeframe of its duration, the subscription expires. However, each call to query a subscription renews its duration.

```
curl -X POST 'http://localhost:8086/db/{db}/subscriptions?u=root&p=root' \
       -d '{"columns”:[“value”, "time", "tx"],“tablenames":["zoo0", "July14", "austin"],”duration":10,"startTm":"2014-08-04 07:14:00”,”endTm":"2014-08-04 16:00:00”}’

```


### Query Subscriptions

A call to query subscriptions will output all data pertinent to the existing subscriptions. This refers to any data within the values and kws table that 

### List Subscriptions

To list all of the subscriptions owned by a user send a curl request to the following URL:

```
curl 'http://localhost:8086/db/{db}/subscriptions?u=root&p=root'
```


### Delete Subscription

To delete a specific subscription send a curl request as such:

```
curl -X DELETE 'http://localhost:8086/db/{db}/subscriptions/{tablename}?u=root&p=root'
```


## Other Subscription-Like Tools

The following tools aren't exactly related to subscriptions, yet their behavior is similar. They have distinct use cases described below.


### QueryCurrent

The API also has the ability to make a current query. The idea behind a current query is to return the most recently-inserted data point into a specified time series. To make a current query, issue the following command:

```
curl -X POST 'http://localhost:8086/db/{db}/query_current/{tablename}?u=root&p=root'
```


### QueryFollow

The API has the ability to make a query follow request. Following a query entails having the data which matches a query’s parameters returned at given intervals. This data is returned through use of [go channels](https://golang.org/doc/effective_go.html#channels).

The interval data tells InfluxDB over what interval to search for new data and output it. Thus, a typical query follow request may take the following form:
 
```
curl -X POST 'http://localhost:8086/db/{db}/query_follow?u=root&p=root' \
        -d '{"column":"trader”,"tablename":"traders&points",”startTime":"2014-01-01 05:00:00","endTime":"2014-01-01 20:13:00"}'
```


**Note** that any keyword used for querying is able to be used as a regular expression. The database will handle it appropriately.
