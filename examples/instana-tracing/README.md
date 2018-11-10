# Envoy + Zipkin + Instana

This example uses the Instana agent to collect the Zipkin-compatible spans and send them to the Instana backend for analysis.
This example is inspired by Envoy's [Zikpin example](https://www.envoyproxy.io/docs/envoy/latest/start/sandboxes/zipkin_tracing).

## Structure

This example uses the [Docker Spring boot application](https://spring.io/guides/gs/spring-boot-docker/),as ultimately the provider of the response through the envoy chain.

<pre>
                           +-----------------------------------------------------------------------+
                           |                               Docker                                  |
                           |                                                                       |
+--------------------+     |   +-------------+          +-----------+          +---------------+   |    +--------------+
|                    |     |   |             |          |           |          |               |   |    |              |
|  Browser requests  +-------->+   Gateway   +--------->+   Proxy   +--------->+  Spring Boot  |   |    |   Instana    |
|  <docker-ip>:8080  |     |   |    Envoy    |          |   Envoy   |          |      App      |   |    |   Backend    |
|                    |     |   |             |          |           |          |               |   |    |              |
+--------------------+     |   +------+------+          +-----+-----+          +---------------+   |    +------------^-+
                           |          |                       |                                    |                 |
                           |          |                       |                                    |                 |
                           |   Zipkin |                       | Zipkin                             |                 |
                           |    Spans |    +--------------+   |  Spans                             |                 |
                           |          |    |              |   |                                    |                 |
                           |          +--> |   Instana    | <-+                                    |                 |
                           |               |    Agent     |                                        |                 |
                           |               |              +----------------------------------------------------------+
                           |               +--------------+                                        |
                           |                                                                       |
                           |                                                                       |
                           +-----------------------------------------------------------------------+
</pre>

## Requirements

1. You need `docker` and `docker-compose` installed on the host. (Likely you want `docker-machine` too, but if you use Docker for Mac or Docker for Windows that should work as well.)
2. You need an Instana tenant to point the agent to its respective license key.

## Configuration

The easiest way to configure the Instana agent in this demo is lilely via environment properties passed
to the Instana agent via the `docker-compose.yml` file.
The explanation about what these environment properties means and what values should they have is available at the [Instana documentation page to run the Agent on Docker](https://docs.instana.io/quick_start/agent_setup/container/docker/).

1. \[Required\] Edit `docker-compose.yml` by adding appropriate values for the following environment variables:
   * `INSTANA_AGENT_ENDPOINT`
   * `INSTANA_AGENT_ENDPOINT_PORT`
   * `INSTANA_AGENT_KEY`
2. \[Optional\] You likely also want to configure a few more settings for the agent, such as:
   * `INSTANA_AGENT_ZONE`
   * `INSTANA_AGENT_TAGS=envoy-zipkin-instana-demo,<other comma-separated tags go here>`

## Build

1. From the folder containing this README.md file, build the Docker container wrapping the Spring boot app with either gradle of maven as follows:
   * Gradle: `pushd hello-docker && ./gradlew build docker && popd`
   * Maven: `pushd hello-docker && ./mvnw install dockerfile:build && popd`
2. To start the demo, execute `docker-compose up --build -d`. (If you are using docker-machine, remember to run `eval $(docker-machine env)` before)
3. Spam a bit of HTTP GET requests to `http://<docker-ip>:8080`. (If you are running docker-machine, the current IP address is likely retrieved via `docker-machine ip default`.)
4. Access your Instana tenant and enjoy your Envoy OpenTracing traces in Instana!

For maximum enjoyment, you likely want to [set up an appplication in the Application Perspective](https://docs.instana.io/products/application_service_management/#application-perspectives) to make the best out of this demo. In this case, the tag `envoy-zipkin-instana-demo` we configured via environment properties in the `docker-compose.yml` file is just the thing ;-)