# Week 2 — Distributed Tracing

  - [Homework](#homework)
    + [Instrument Honeycomb with OTEL](#instrument-honeycomb-with-otel)
    + [Run queries to explore traces within Honeycomb](#run-queries-to-explore-traces-within-honeycomb)
    + [Instrument AWS X-Ray into backend flask application](#instrument-aws-x-ray-into-backend-flask-application)
    + [Configure and provision X-Ray daemon within docker-compose and send data back to X-Ray API](#configure-and-provision-x-ray-daemon-within-docker-compose-and-send-data-back-to-x-ray-api)
    + [Observe X-Ray traces within the AWS Console](#observe-x-ray-traces-within-the-aws-console)
    + [Integrate Rollbar for Error Logging](#integrate-rollbar-for-error-logging)
    + [Trigger an error an observe an error with Rollbar](#trigger-an-error-an-observe-an-error-with-rollbar)
    + [Install WatchTower and write a custom logger to send application log data to CloudWatch Log group](#install-watchtower-and-write-a-custom-logger-to-send-application-log-data-to-cloudwatch-log-group)
  - [Challenges](#challenges)
    + [Instrument Honeycomb for the frontend-application to observe network latency between frontend and backend](#instrument-honeycomb-for-the-frontend-application-to-observe-network-latency-between-frontend-and-backend)
    + [Add custom instrumentation to Honeycomb to add more attributes](#add-custom-instrumentation-to-honeycomb-to-add-more-attributes)
    + [Run custom queries in Honeycomb and save them](#run-custom-queries-in-honeycomb-and-save-them)

## Homework

### Instrument Honeycomb with OTEL

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



### Instrument AWS X-Ray into backend flask application



### Configure and provision X-Ray daemon within docker-compose and send data back to X-Ray API



### Observe X-Ray traces within the AWS Console



### Integrate Rollbar for Error Logging



### Trigger an error an observe an error with Rollbar



### Install WatchTower and write a custom logger to send application log data to CloudWatch Log group



## Challenges

### Instrument Honeycomb for the frontend-application to observe network latency between frontend and backend



### Add custom instrumentation to Honeycomb to add more attributes



### Run custom queries in Honeycomb and save them


