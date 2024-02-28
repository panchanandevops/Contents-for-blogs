# Deep Dive into Observability with OpenTelemetry

## What is observability?
Observability is the ability to understand the internal state of a system by examining its outputs. In the context of software, this means being able to understand the internal state of a system by examining its telemetry data, which includes traces, metrics, and logs.

To make a system observable, it must be instrumented. That is, the code must emit traces, metrics, or logs. The instrumented data must then be sent to an observability backend.

## What is OpenTelemetry?

OpenTelemetry is an Observability framework and toolkit designed to create and manage telemetry data such as traces, metrics, and logs. Crucially, OpenTelemetry is vendor- and tool-agnostic, meaning that it can be used with a broad variety of Observability backends, including open source tools like Jaeger and Prometheus, as well as commercial offerings.

OpenTelemetry is not an observability backend like Jaeger, Prometheus, or other commercial vendors. OpenTelemetry is focused on the generation, collection, management, and export of telemetry. 

A major goal of OpenTelemetry is that you can easily instrument your applications or systems, no matter their language, infrastructure, or runtime environment. Crucially, the storage and visualization of telemetry is intentionally left to other tools.

---

## Why OpenTelemetry?

With the rise of cloud computing, microservices architectures, and increasingly complex business requirements, the need for software and infrastructure observability is greater than ever.

OpenTelemetry satisfies the need for observability while following two key principles:

1. You own the data that you generate. There’s no vendor lock-in.
2. You only have to learn a single set of APIs and conventions.


Both principles combined grant teams and organizations the flexibility they need in today’s modern computing world.

---

## OpenTelemetry Operator for Kubernetes

This is an implementation of a Kubernetes Operator, that **manages collectors** and **auto-instrumentation** of the workload using OpenTelemetry **instrumentation libraries**.

OpenTelemetry is composed of two main components:

- OpenTelemetry Collector
- auto-instrumentation of the workloads using OpenTelemetry instrumentation libraries


---

## Deployment Modes for the OpenTelemetry Operator in Kubernetes

The OpenTelemetry Operator for Kubernetes offers several deployment modes for the OpenTelemetry Collector, each catering to specific use cases and presenting unique considerations. Understanding these modes is crucial for selecting the most appropriate approach for your environment.

### 1. Deployment:

* **Description:** This is the **default and recommended mode** for most deployments. It utilizes a Kubernetes Deployment resource to launch a **single instance** of the OpenTelemetry Collector. This ensures **high availability** through automatic container restarts in case of failures.
* **Advantages:**
    * **Simplicity:** Deployment offers **straightforward setup and management**, minimizing configuration effort.
    * **High Availability:** Guarantees the Collector's availability even if the single pod encounters issues.
* **Considerations:**
    * **Resource Constraints:** This mode might not be suitable for **resource-intensive workloads** requiring significant processing power or memory.

### 2. DaemonSet:

* **Description:** This mode launches a **separate Collector instance on each node** in your Kubernetes cluster. This approach guarantees **comprehensive telemetry data collection** from all nodes, irrespective of application placement.
* **Advantages:**
    * **Complete Coverage:** Ensures **comprehensive data collection** from all nodes, making it ideal for scenarios with applications distributed across the cluster.
    * **Scalability:** Automatically scales the Collector horizontally as new nodes are added to the cluster.
* **Considerations:**
    * **Resource Consumption:** Each node running a Collector instance can lead to **increased resource utilization**.
    * **Management Complexity:** Managing multiple Collector instances can be more **complex** compared to the Deployment mode.

### 3. StatefulSet:

* **Description:** This mode utilizes a Kubernetes StatefulSet to deploy the Collector. This ensures **data persistence** by retaining assigned storage and maintaining data across pod restarts or disruptions. This is essential when the Collector stores collected data locally, using a persistent volume claim (PVC).
* **Advantages:**
    * **Data Persistence:** Guarantees the **preservation of collected data** even during pod restarts or rescheduling.
    * **Stateful Data Management:** Suitable for scenarios requiring **persistent storage of telemetry data**.
* **Considerations:**
    * **Increased Complexity:** Requires additional configuration compared to the Deployment mode, particularly regarding storage management.
    * **Scaling Challenges:** Scaling StatefulSets can be **more complex** compared to simple Deployments.

### 4. Sidecar:

* **Description:** This mode involves deploying the OpenTelemetry Collector as a **sidecar container** alongside your application pods. This allows for **direct and efficient collection of telemetry data** from the application container, bypassing network-based collection methods.
* **Advantages:**
    * **Direct and Efficient Collection:** Captures data directly from the application, reducing network overhead and enhancing data reliability.
    * **Per-Application Customization:** Enables configuration specific to each application, potentially optimizing data collection for different needs.
* **Considerations:**
    * **Increased Complexity:** Requires modifications to existing application deployments to include the sidecar container, adding complexity to the setup.
    * **Resource Management:** Each application pod runs an additional container, potentially impacting resource utilization.

### Choosing the Right Mode:

The optimal deployment mode depends on your specific requirements and considerations:

* **For most basic scenarios, the default Deployment mode is sufficient.**
* **If complete data coverage across all nodes is critical, consider a DaemonSet.**
* **For data persistence, StatefulSets are necessary.**
* **Sidecar injection is valuable for efficient and direct data collection from specific applications.**

By carefully evaluating your needs and resource constraints, you can select the most appropriate deployment mode for the OpenTelemetry Operator in your Kubernetes environment, ensuring efficient and reliable telemetry data collection.

---

## What is the Client library in OpenTelemetry?

The client library in OpenTelemetry is also known as the **instrumentation library**. Applications need to have an OTel library to collect **manual or automatic instrumentation**.

Here’s a list of languages that OpenTelemetry currently supports:

- C++

- .net

- erlang/elixir

- GO

- Java

- JavaScript/Node.js

- PHP

- Python

- Ruby

- Rust

- Swift

Most of these libraries support **automatic instrumentation** that requires you to load the right dependency. You have to configure the connection to the collector through **environment variables or system properties.**

For most of the supported languages, automatic instrumentation means that it will **attach a specific library to your application's runtime and inject bytecode** to capture OpenTelemetry from several popular libraries and frameworks. It won’t instrument any of **your custom code automatically**, it has to be done **manually**. [(Learn more about supported frameworks in the OpenTelemetry Docs).](https://opentelemetry.io/docs/languages/)

---

## OpenTelemetry Collector

The collector comprises three components that are enabled through a pipeline:

- Receiver to get data into the collector, sent by either push or pull

- Processor to decide what to do with the data received

- Exporter to decide where to send the data, done by either pull or push



![](./IMG/opentel2_353fe568217a4ecd8f2727f818d6534e%20(1).png)

---

## How to configure the pipeline

The pipeline is defined through the **otel-collector-conf.yaml**, which is usually stored in Kubernetes in the **config map**.


Imagine you have a simple web application running in a single Docker container named `my-app`. You want to collect **metrics, traces, and logs** from this application for monitoring purposes. Here's an example `otelconfig.yaml` using the OpenTelemetry Collector that demonstrates collecting data from your application and exporting it to **Prometheus, Loki, and Jaeger**:

```yaml
receivers:
  otlp: # Receives OpenTelemetry data
    protocols:
      grpc:
      http:

exporters:
  # Export to Prometheus for metrics monitoring
  prometheus:
    endpoint: "localhost:9090"  # Replace with your Prometheus address

  # Export traces to Jaeger for distributed tracing analysis
  jaeger:
    endpoint: "http://jaeger:14268/api/traces"  # Replace with your Jaeger address

  # Export logs to Loki for centralized log aggregation and analysis
  loki:
    endpoint: "http://loki:3100/loki/api/v1/push"

service:
  # Define pipelines for different data types
  pipelines:
    # Metrics pipeline
    metrics:
      receivers: [otlp]
      exporters: [prometheus]

    # Traces pipeline
    traces:
      receivers: [otlp]
      exporters: [jaeger]

    # Logs pipeline
    logs:
      receivers: [otlp]
      exporters: [loki]
```

**Explanation:**

1. **Receivers:**
   - `otlp`: This receiver listens for OpenTelemetry data sent by your application or other sources using the OpenTelemetry Line Protocol (OTLP) over either gRPC or HTTP protocols.

2. **Exporters:**
   - **Prometheus:** This exporter sends collected metrics data to your Prometheus instance for visualization and alerting. 
   - **Jaeger:** This exporter sends trace data to your Jaeger instance for distributed tracing analysis, helping you understand the flow of requests across different services.
   - **Loki:** This exporter sends log data to your Loki instance for centralized storage and querying.

3. **Service:**
   - **Pipelines:** Each pipeline defines how to process and export specific data types:
     - **metrics:** This pipeline receives metrics data from the `otlp` receiver and exports it to the `prometheus` exporter.
     - **traces:** This pipeline receives trace data from the `otlp` receiver and exports it to the `jaeger` exporter.
     - **logs:** This pipeline receives log data from the `otlp` receiver and exports it to the `loki` exporter.

**How to use this configuration:**

1. **Instrument your application:** Make sure your application is instrumented with an OpenTelemetry library (e.g., Python: `opentelemetry-python`) to send metrics, traces, and logs using the OpenTelemetry protocol.
2. **Deploy supporting services:** Ensure you have Prometheus, Jaeger, and Loki running and accessible in your environment.
3. **Create a ConfigMap:** Create a Kubernetes ConfigMap named `otel-config` with the above YAML content.
4. **Mount the ConfigMap in your Collector Deployment:** Mount the ConfigMap containing this configuration file into your OpenTelemetry Collector deployment.


**Remember:** This is a basic example for learning purposes. For production deployments, consider factors like security, scalability, and high availability.

---

## Sampling in OpenTelemetry

Sampling in OpenTelemetry (OTel) is a technique for selectively processing a subset of telemetry data (traces or metrics) to optimize resource utilization, control costs, and ensure the responsiveness of downstream analysis systems. This is crucial in high-volume environments where processing all generated data can become overwhelming.

### Types of Sampling in OpenTelemetry:

OpenTelemetry supports various sampling strategies to manage the volume of data being processed and exported:

#### 1. Head-Based Sampling:

* **Decision:** Made at the very beginning of a trace creation.
* **Mechanism:** A pre-defined sampling rate (e.g., 10%) determines whether the entire trace is sampled or discarded.
* **Use case:** Suitable for scenarios where the beginning of a trace holds significant information for analysis (e.g., error conditions).

#### 2. Tail-Based Sampling:

* **Decision:** Made after a trace is complete.
* **Mechanism:** Based on specific criteria evaluated at the end of the trace (e.g., duration, presence of specific events).
* **Use case:** Useful for identifying and analyzing long-running or error-prone traces.

#### 3. Probabilistic Sampling:

* **Decision:** Made for each trace or metric data point independently.
* **Mechanism:** Each trace or data point has a pre-defined probability (e.g., 50%) of being sampled.
* **Use case:** Provides a statistically significant sample of data across a large volume, balancing data quantity with representativeness.

#### 4. Adaptive Sampling:

* **Dynamically adjusts:** Based on real-time factors like system load or error rates.
* **Mechanism:** Aims to maintain a targeted sampling rate while adapting to changing conditions.
* **Use case:** Ideal for dynamic environments where data volume fluctuates, ensuring efficient resource utilization while capturing sufficient data for analysis.

**Choosing the right sampling strategy** depends on your specific needs and performance goals. Consider factors like:

* **Data volume:** Higher volume might necessitate sampling for efficiency.
* **Analysis requirements:** Understanding the crucial characteristics of interest influences sampling criteria.
* **Downstream system capacity:** Consider the ability of your analysis tools to handle the sampled data.

Remember, sampling introduces trade-offs. While **reducing data volume, it also means potential loss of information**. Selecting the appropriate sampling strategy ensures a balance between **resource efficiency and capturing valuable insights** from your telemetry data.

---

## Conclusion
Observability helps us understand w**hat's happening inside our systems** by looking at the data they produce. OpenTelemetry is a toolkit that helps **collect** this data, like **traces, metrics, and logs**, making it easier to monitor our applications. With OpenTelemetry, we can track how our software behaves, diagnose issues, and improve performance. 


Sampling in OpenTelemetry is like taking a representative sample of data to save resources while still getting useful insights. **It's a bit like tasting a small spoonful of soup to know what the whole pot tastes like.** So, by using OpenTelemetry, we can keep an eye on our systems without drowning in too much data.

A **heartfelt thank** you to all our readers for exploring the world of observability and OpenTelemetry with us. Your curiosity and engagement make our journey in understanding and optimizing systems truly fulfilling. We appreciate your time and interest in learning about these essential tools for modern software development. Here's to continued exploration, growth, and success in your endeavors. **Thank you for being a part of our community!**