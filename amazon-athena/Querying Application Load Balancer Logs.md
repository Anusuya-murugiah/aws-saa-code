## Policy for S3 bucket

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::127311923021:root"
      },
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::bucket-name/AWSLogs/*"
    }
  ]
}
```

## Create table in Athena

```sql
CREATE EXTERNAL TABLE IF NOT EXISTS alb_logs (
    type string,
    time string,
    elb string,
    client_ip string,
    client_port int,
    target_ip string,
    target_port int,
    request_processing_time double,
    target_processing_time double,
    response_processing_time double,
    elb_status_code string,
    target_status_code string,
    received_bytes bigint,
    sent_bytes bigint,
    request_verb string,
    request_url string,
    request_proto string,
    user_agent string
)
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe'
WITH SERDEPROPERTIES (
    'serialization.format' = '1',
    'field.delim' = ' '
)
LOCATION 's3://YOUR-S3-BUCKET/AWSLogs/YOUR-ACCOUNT-ID/elasticloadbalancing/us-east-1/';
```

## EXAMPLE QUERIES

### View the first 100 access log entries in chronological order:

```sql
SELECT *
FROM alb_logs
ORDER by time ASC
LIMIT 100;
```

### List clients in descending order, by the number of times that each client visited a specified URL:

```sql
SELECT client_ip, elb, request_url, count(*) as count from alb_logs
GROUP by client_ip, elb, request_url
ORDER by count DESC;
```

### Shows the URLs visited by Chrome browser users:

```sql
SELECT request_url
FROM alb_logs
WHERE user_agent LIKE '%Chrome%'
LIMIT 10;
```

