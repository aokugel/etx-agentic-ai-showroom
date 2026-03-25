# Conclusion

## What You've Accomplished

Through this hands-on workshop, you:

- **Explored Llama Stack** — the API layer for building generative AI applications on Red Hat AI — and used the Playground to interact with a model
- **Configured tools and MCP integrations** — added websearch, OpenShift, and GitHub tool providers to Llama Stack in a single ConfigMap update
- **Deployed an agentic application** that autonomously analyzes CI/CD failures and creates GitHub issues
- **Ran a real pipeline**, watched it fail, and saw the agent automatically triage the failure
- **Used distributed tracing** with OpenTelemetry and Tempo to understand agent behavior, identify bottlenecks, and analyze token costs

## Key Technical Takeaways

### Add Tools, Run the Pipeline, See It Work

The core workflow is straightforward: configure tool integrations in Llama Stack, deploy the agent, trigger a pipeline, and observe the results. The power comes from how quickly you can extend an agent's capabilities by plugging in new MCP servers.

### Observability Is Not Optional

Without tracing, you're flying blind. Traces reveal:

- Where time is actually spent (tool execution vs. inference vs. overhead)
- Token consumption patterns that drive costs
- Potential security issues (sensitive data leaking into traces)
- Optimization opportunities (parallelizing tool calls, reducing prompt inflation)

### Production AI Requires More Than Models

Making an agent production-ready requires tool integration, observability infrastructure, and repeatable deployment workflows — not just a good model.

## Resources

- **Red Hat AI**: https://docs.redhat.com/en/documentation/red_hat_ai/3
- **Llama Stack**: https://github.com/llamastack/llama-stack
- **Model Context Protocol**: https://modelcontextprotocol.io
- **OpenTelemetry**: https://opentelemetry.io

## Closing Thought

Agentic AI systems solve real problems when properly instrumented and integrated into existing workflows on the right AI platform. The techniques you've practiced here are the foundation for delivering reliable AI solutions to your clients and their customers.
