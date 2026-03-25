# Agentic System Observability

> **Important:** Now that your agent is running and telemetry is configured, you'll use distributed tracing to understand what's happening inside your agentic system — where time is spent, how tools are invoked, and where to optimize.

In production Generative AI systems, observability is your ability to understand what's happening inside your AI applications by examining the data they produce. Without it, you only know something is wrong when it breaks or someone complains. With proper observability, you can spot problems before they impact users and fix issues quickly.

## Red Hat AI Observability Stack

Red Hat AI provides [centralized platform observability](https://docs.redhat.com/en/documentation/red_hat_openshift_ai_self-managed/3.0/html/managing_openshift_ai/managing-observability_managing-rhoai): an integrated solution featuring the **OpenTelemetry Collector (OTC)** for data ingestion, **Prometheus** for metrics, and the **Red Hat build of Tempo** for distributed tracing.

![Observability Stack](images/observability-stack.png)

We already enabled OpenTelemetry in our Llama Stack ConfigMap (the `telemetry` provider configured in the previous module). Llama Stack automatically creates spans for each inference request and emits token usage metrics. Each request generates:

- **Traces**: Distributed traces showing the request flow with timing data
- **Metrics**: Token counters (`llama_stack_prompt_tokens_total`, `llama_stack_completion_tokens_total`) labeled by model_id and provider_id

## Understanding Distributed Tracing

Imagine following a request through your entire system—from user prompt to tool selection to tool execution to final response. Distributed tracing does exactly this by connecting spans (records of work) into a complete trace.

Each span includes:

- **Operation name**: What work was performed (e.g., "LLM inference", "tool_execution")
- **Duration**: How long it took
- **Parent span**: What triggered this operation (builds the request tree)
- **Attributes**: Metadata like query text, documents retrieved, tokens generated

When spans connect via parent-child relationships, they form a trace—the complete story of your agent's actions.

![Tracing Spans](images/tracing-spans.png)

The waterfall view shows total request time, each service's contribution, sequential vs. parallel operations, and where the most time is spent.

## Generate Trace Data

1. Navigate to `Observe` > `Traces` in the OpenShift web console:

![Navigate to Traces](images/navigate-to-traces.png)

2. Select the Tempo instance from the drop-down:

![Tempo Instance Select](images/tempo-instance-select.png)

3. Go back to the Llama Stack Playground. Select **Agent-based** and one or more tools, then complete a few agentic chat interactions that trigger tool calls.

> **Note:** You may also re-run the demo pipeline that triggers your agent to generate traces from a real pipeline failure scenario.

4. It takes a few minutes for traces to populate Tempo, so take your time in the playground.

## Filter and View Traces

1. Navigate back to the traces dashboard. You'll see constant health-check traces (`/v1/providers`, `/v1/version`) — these are readiness probes.

![Trace Spam](images/trace-spam.png)

2. Filter these out with a TraceQL query. Click `Show Query`:

![Show Query](images/show-query.png)

3. Paste this query:

```sh
{ span.raw_path!="/v1/providers" && span.raw_path!="/v1/version"}
```

4. Click `Run Query`:

![Run Query](images/run-query.png)

5. You'll see a shorter list focused on your actual interactions. Look for these trace names:
   - **create_agent_turn** — the full agent reasoning and tool-calling flow
   - **/v1/inference/chat-completions** — direct inference calls

6. Click on a **create_agent_turn** trace to dig in:

![Click Trace](images/click-trace.png)

7. View the complete trace:

![Full Trace](images/full-trace.png)

8. Click on any span to view its metadata:

![Span Metadata](images/span-metadata.png)

9. Review the key data:
   - Individual spans and their names
   - Individual span duration
   - Overall trace time
   - Metadata for each span
   - Parent-child relationships (what triggered what, what ran concurrently)

> **Note:** Download the trace details as a `.json` file to more easily search through the data.

![Download Trace](images/download-trace.png)

## Investigate Traces

Armed with trace data, work through these questions:

1. **Did the agent interpret your intent correctly?**
   - Look at your user message in the agent turn. Did the agent choose the correct toolgroup and tool?

2. **Which step took the longest: tool selection inference, tool execution, or final response inference?**
   - Compare the durations of inference spans vs. tool_execution spans.

3. **What dominates overall time: tools, inference, or overhead?**
   - Can anything be streamlined?

4. **Is the slowest step consistent across traces or variable?**

5. **Is latency due to remote inference over network, or the model itself?**
   - Look for evidence: short tool execution but long inference, signs of retries/timeouts.

6. **Are we exporting sensitive data into traces?**
   - In our example, extensive pod and networking data gets dumped into the trace:

   ![Too Much Pod Data](images/too-much-pod-data.png)

   - Think about how this security risk could be mitigated.

7. **Is tool output being reinjected verbatim?**
   - In `inference.input`, pod data may have been injected in its entirety. This is a classic driver of prompt token inflation.

8. **Are tools being invoked in the optimal order?**
   - Could independent tool calls be parallelized to reduce end-to-end latency?

9. **Across the agent turn, how many tokens were consumed?**
   - Sum the `total_tokens` per inference span and compare prompt vs. completion tokens.
   - These trace token metrics reflect Llama Stack's own inference calls but may exclude provider-side additions (proxy/gateway prompt rewriting, hidden system wrappers).
   - For full cost attribution, combine provider metrics (billing truth) with traces (step-by-step optimization diagnosis).

## Summary

You now have the tools to observe, measure, and optimize your Llama Stack agent. Use distributed tracing to identify bottlenecks, reduce token costs, and ensure your agentic system runs reliably at scale.
