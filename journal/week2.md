# Week 2 — Distributed Tracing

  - [Homework](#homework)
    + [Instrument Honeycomb](#instrument-honeycomb)
    + [Run queries to explore traces within Honeycomb](#run-queries-to-explore-traces-within-honeycomb)
    + [Instrument AWS X-Ray](#instrument-aws-x-ray)
    + [Observe X-Ray traces within the AWS Console](#observe-x-ray-traces-within-the-aws-console)
    + [Integrate Rollbar for Error Logging](#integrate-rollbar-for-error-logging)
    + [Trigger an error an observe an error with Rollbar](#trigger-an-error-an-observe-an-error-with-rollbar)
    + [Install WatchTower and write a custom logger to send application log data to CloudWatch Log group](#install-watchtower-and-write-a-custom-logger-to-send-application-log-data-to-cloudwatch-log-group)
  - [Challenges](#challenges)
    + [Instrument Honeycomb for the frontend-application to observe network latency between frontend and backend](#instrument-honeycomb-for-the-frontend-application-to-observe-network-latency-between-frontend-and-backend)
    + [Add custom instrumentation to Honeycomb to add more attributes](#add-custom-instrumentation-to-honeycomb-to-add-more-attributes)
    + [Run custom queries in Honeycomb and save them](#run-custom-queries-in-honeycomb-and-save-them)

## Homework

### Instrument Honeycomb

Followed [Honeycomb documentation](https://docs.honeycomb.io/getting-data-in/opentelemetry/python/) to instrument our system with OpenTelemetry and ensure that the data is sent to Honeycomb.

1. Logged into my Honeycomb account and created a new environment called *bootcamp*:

![Proof of working Honeycomb](/_docs/assets/env_honeycomb.png)

2. Set my Honeycomb API key as environment variable in Gitpod:
```sh
export HONEYCOMB_API_KEY="MY_KEY"
gp env HONEYCOMB_API_KEY="MY_KEY"
```

3. Added the following environment variables to backend-flask in Docker compose file:
```yml
services:
  backend-flask:
    environment:
      OTEL_SERVICE_NAME: "backend-flask"
      OTEL_EXPORTER_OTLP_ENDPOINT: "https://api.honeycomb.io"
      OTEL_EXPORTER_OTLP_HEADERS: "x-honeycomb-team=${HONEYCOMB_API_KEY}"
```

4. Added the following Python modules for OpenTelemetry to *requirements.txt*:
```
opentelemetry-api 
opentelemetry-sdk 
opentelemetry-exporter-otlp-proto-http 
opentelemetry-instrumentation-flask 
opentelemetry-instrumentation-requests
```

5. Opened backend-flask folder and installed these dependencies:
```
cd backend-flask
pip install -r requirements.txt
```

6. Created and initialized a tracer and Flask instrumentation in *app.py* to send data to Honeycomb:
```py
from opentelemetry import trace
from opentelemetry.instrumentation.flask import FlaskInstrumentor
from opentelemetry.instrumentation.requests import RequestsInstrumentor
from opentelemetry.exporter.otlp.proto.http.trace_exporter import OTLPSpanExporter
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor

# initialize tracing and an exporter that can send data to Honeycomb
provider = TracerProvider()
processor = BatchSpanProcessor(OTLPSpanExporter())
provider.add_span_processor(processor)
trace.set_tracer_provider(provider)
tracer = trace.get_tracer(__name__)

app = Flask(__name__)

# initialize automatic instrumentation with Flask
FlaskInstrumentor().instrument_app(app)
RequestsInstrumentor().instrument()
```

7. Ran docker compose up and verified that a dataset was created in Honeycomb:


### Run queries to explore traces within Honeycomb



### Instrument AWS X-Ray

1. Installed AWS X-Ray SDK by adding *aws-xray-sdk* to requirements.txt and running:
```
cd backend-flask
pip install -r requirements.txt
```
2. Added modules for X-Ray SDK to app.py:
```py
from aws_xray_sdk.core import xray_recorder
from aws_xray_sdk.ext.flask.middleware import XRayMiddleware
```
Configured and enabled X-Ray SDK for Flask:
```py
# app = Flask(__name__)
xray_url = os.getenv("AWS_XRAY_URL")
xray_recorder.configure(service='backend-flask', dynamic_naming=xray_url)
XRayMiddleware(app, xray_recorder)
```

3. Created a group in AWS X-Ray using Gitpod CLI:
```
aws xray create-group \
   --group-name "Cruddur" \
   --filter-expression "service(\"backend-flask\")"
```
[X-Ray Groups](/_docs/assets/xray_group_aws.png)

Verified in AWS Console:

[X-Ray Groups Console](/_docs/assets/xray_group.png)

4. Created aws/json/xray.json file for X-Ray sampling rule:
```json
{
    "SamplingRule": {
        "RuleName": "Cruddur",
        "ResourceARN": "*",
        "Priority": 9000,
        "FixedRate": 0.1,
        "ReservoirSize": 5,
        "ServiceName": "backend-flask",
        "ServiceType": "*",
        "Host": "*",
        "HTTPMethod": "*",
        "URLPath": "*",
        "Version": 1
    }
  }
```
Created a new sampling rule by running the following command:
```
aws xray create-sampling-rule --cli-input-json file://aws/json/xray.json
```
[X-Ray Sampling Rule](/_docs/assets/xray_sampling.png)

Verified in AWS Console:

[X-Ray Sampling Rule Console](/_docs/assets/xray_sampling_aws.png)

5. Added 2 X-Ray environmental variables to backend-flask in Docker compose file:
```yaml
AWS_XRAY_URL: "*4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}*"
AWS_XRAY_DAEMON_ADDRESS: "xray-daemon:2000"
```
Added X-Ray Daemon to Docker compose file:
```yaml
xray-daemon:
  image: "amazon/aws-xray-daemon"
  environment:
    AWS_ACCESS_KEY_ID: "${AWS_ACCESS_KEY_ID}"
    AWS_SECRET_ACCESS_KEY: "${AWS_SECRET_ACCESS_KEY}"
    AWS_REGION: "eu-central-1"
  command:
    - "xray -o -b xray-daemon:2000"
  ports:
    - 2000:2000/udp
```
[Commit link](https://github.com/darya-korobenko/aws-bootcamp-cruddur-2023/commit/a9583a5eacab2799f7fc36e8a10bd7eec1c9a9a9)

Ran docker compose and verified that the data is sent to X-Ray API:

[X-Ray Logs](/_docs/assets/xray_logs.png)

Verified that the traces are visible in AWS Console:

[X-Ray Traces](/_docs/assets/xray_traces.png)

6. Created a segment called *user-activities* in app.py with capture using the X-Ray SDK:
```py
@app.route("/api/activities/@<string:handle>", methods=['GET'])
@xray_recorder.capture('user-activities')
def data_handle(handle):
  model = UserActivities.run(handle)
  if model['errors'] is not None:
    return model['errors'], 422
  else:
    return model['data'], 200
```
Created a subsegment called *mock-data* in user-activities.py with xray_recorder module:
```py
subsegment = xray_recorder.begin_subsegment('mock-data')

dict = {
  "now": now.isoformat(),
  "result-size": len(model['data'])
}

subsegment.put_metadata('key', dict, 'namespace')
xray_recorder.end_subsegment()
```
Verified that segment and subsegment were created and the traces are visible in AWS Console:

[X-Ray Segments and Subsegments](/_docs/assets/xray_segment_subsegment.png)

### Observe X-Ray traces within the AWS Console



### Integrate Rollbar for Error Logging



### Trigger an error an observe an error with Rollbar



### Install WatchTower and write a custom logger to send application log data to CloudWatch Log group



## Challenges

### Instrument Honeycomb for the frontend-application to observe network latency between frontend and backend



### Add custom instrumentation to Honeycomb to add more attributes



### Run custom queries in Honeycomb and save them


