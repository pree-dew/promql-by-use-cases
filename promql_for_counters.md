#### Using rate on counters ####

- **Use Case**

  How to find relevant metric value from counters using `rate`

- **promql**
  ```
  rate(metric_name{}[scrape_interval * 4])*(time_resolution)
  ```

  Example: Find number of request received per min, for a metric coming with a scrape interval of 1min

  ```
  rate(http_requests_total{}[4m])*60
  ```
- **Explanation**
  
  - Counter is one of the metric type supported by prometheus compatible system. Nature of the these metrics is
    monotonically increasing. Instant values of any such metric, is barely useful. So, to find meaningful information
    we have to find the rate of change of these metrics.
  - Prometheus compatible systems provide 3 functions to find rate of change of counters:
    - `rate`
    - `increase`
    - `irate`
  - To use any of the above function, Prometheus needs atleast 2 data points otherwise it will return the single
    point itself. Recommended duration that specifies sufficient number of data points to be received is
    4 times the scrape inteval, so that we have atleast >= 2 data points.
  - `rate` is per second average of metric value over the duration specified (in example it is 4m], Prometheus
    finds this value by finding the slope from start value till end value for the duration.
 
          
