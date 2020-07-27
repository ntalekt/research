# Troubleshooting Progress: 5 / 12

## Section 13:226

### Application Failure

-   Service that exposed mysql via ClusterIP was named `mysql` instead of `mysql-service`
-   mysql-service had an incorrect TargetPort
