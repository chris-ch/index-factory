# index-factory

Financial Indices Factory running on AWS

## Offline usage

### install

First install serverless globally with `npm install -g serverless`

Then:

```bash
virtualenv --python=python3 venv
. venv/bin/activate
pip install -r requirements.txt
pip install --upgrade awscli
npm install --save-dev serverless-dynamodb-local serverless-wsgi serverless-python-requirements serverless-offline
sls dynamodb install
serverless plugin install --name serverless-offline
serverless plugin install --name serverless-s3-local
serverless plugin install --name serverless-dynamodb-local
```

## Running unit tests

```bash
PYTHONPATH=src python -m unittest tests/*.py
```

### Starting SLS
```bash
SLS_DEBUG=DEBUG sls offline start  # make sure stage is declared in serverless.yml custom section
```

### Starting SLS as a separate processes

```bash
sls dynamodb start
sls wsgi serve  
```

## Running BDD tests

```bash
behave --logging-level=INFO bdd
```

### Test requests

```bash
curl -H "Content-Type: application/json" -X POST http://localhost:3000/indices -d '{"indexCode": "us-small-caps", "name": "US Small Caps"}'
curl -H "Content-Type: application/json" -X POST http://localhost:3000/indices -d '{"indexCode": "us-mid-caps", "name": "US Mid Caps"}'
curl -H "Content-Type: application/json" -X POST http://localhost:3000/indices -d '{"indexCode": "us-large-caps", "name": "US Large Caps", "is_deleted": "1"}'
curl -H "Content-Type: application/json" -X GET http://localhost:3000/indices/us-small-caps
curl -H "Content-Type: application/json" -X GET http://localhost:3000/indices
```

## Comments

The python code favors the higher level boto3.resource API over boto3.client .

Because DynamoDB handles numbers in a generic way they are transformed as Decimals in Python, so that some work is required before returning json.

## Feeding S3

The endpoint needs to be 127.0.0.1 !!! localhost is failing !!!

```bash
AWS_ACCESS_KEY_ID=S3RVER AWS_SECRET_ACCESS_KEY=S3RVER aws --debug --endpoint http://127.0.0.1:8001 s3api put-object --bucket index-factory-daily-prices-bucket --key "US/2020/01/US_20200131.csv" --body resources/fake-data/US_20200131.csv
```

## Feeding DynamomDB (not working)

```bash
AWS_ACCESS_KEY_ID=S3RVER AWS_SECRET_ACCESS_KEY=S3RVER aws --debug --endpoint http://127.0.0.1:8000 dynamodb put-item --table-name index-factory-table-local --item '{ "partitionKey": {"S": "1" }, "sortKey": { "S": "1989"}}'
```

## DynamoDB console
> http://127.0.0.1:8000/shell/

Listing items:

```javascript
var params = {
    TableName: 'index-factory-table-local'
};
dynamodb.scan(params, function(err, data) {
    if (err) ppJson(err); // an error occurred
    else ppJson(data); // successful response
});
```
