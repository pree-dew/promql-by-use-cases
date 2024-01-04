#### Extract service name from pod label ####

- **Use Case**

  We receive pod in almost all kubernetes metrics, it may be required sometimes to extract the
  service name from pod itself, if service name is not coming explicitly.
          
  `pod` label looks like this : `user-service-23n4-98jh-34ub`, we want

  `service` label to `user-service` extracted from pod

- **promql**
  ```
  sum by (service)(label_replace(sum by (pod) (kube_pod_status_phase{cluster=~'$cluster.*', phase='Running'}), "service", "$1", "pod", "([-a-z]+)-.*"))
  ```
- **Explanation**
  
  - `label_replace` used `pod` label after executing query accepted as first argument.
  - Applies regex on it, specified as last argument
  - Captures group specified as 3rd argument, here it is $1
  - Create a new label with key specified as 2nd argument("service") and assign captured group value to it
          
