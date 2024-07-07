## Types of an Expression ##

In Prometheus's expression language, an expression or sub-expression can evaluate to one of four types:
- string
- scalar
- instant vector
- range vector

Let's deep dive into each of them.
1. ### Strings ###

   Strings are mostly used in as an argument to functions like `label_replace` etc, not used much in promql.
   Note: label value in promql like `up{job="prod"}` is not a string value as it is not a separate sub expression.

2. ### Scalars ###

  Scalars are just number without any information of label dimensions. It is used in functions like:
  `topk(4,` or `histogram_quantile(0.9` etc. Any valid 64 bit floating point value is considered as scalar.
  Note: Infinity or Nans are special float64 values , so also considered as scalars

3. ### Instant Vector ###

   Instant vectors are mapped as:
   
   timeseries 1 --> labels 1, sample 1, time t1
   
   timeseries 2 --> labels 2, sample 2, time t1
   
   timeseries 3 --> labels 3, sample 3, time t1

   Example:

    ```
    demo_api_http_requests_in_progress{instance="demo-service-0:10000", job="demo"}     1 @1720340151
    demo_api_http_requests_in_progress{instance="demo-service-1:10001", job="demo"}     1 @1720340151
    demo_api_http_requests_in_progress{instance="demo-service-2:10002", job="demo"}     2 @1720340151
    ```

   Basically, each time series has only 1 sample and all timeseries samples are reported at single time.
   We can get instant vectors from an instant vector query (like `http_request_total{}`) or either by using some function like `rate` or `sum` that returns
   instant vector as an output.

4. ### Range Vector ###

   List of timeseries with range of sample over time for each series:

   Example:

   ![Screenshot 2024-07-07 at 1 49 28 PM](https://github.com/pree-dew/promql-by-use-cases/assets/132843509/66054835-5a5f-46f7-9953-5e9df79fe9fb)


  We can get range vector as output by using these 2 ways:
  1. Using range vector selector in query, eg: `http_request_total{}[[10m]`
  2. By using a subquery expression like `<expression>[2m:10m]`

  Range vectors are useful when we want to aggregate over a window of time, eg: `rate(http_request_total{}[4m])` or `sum_over_time(http_request_total{}[5m])`

  ### Important note on metric type ###
  
  Some promql functions are compatible with only a few specific types like rate works with counter etc but promql doesn't really check that the metric type 
  is compatible or not and return some unwanted result so, user has to take care of this while writing promql.
