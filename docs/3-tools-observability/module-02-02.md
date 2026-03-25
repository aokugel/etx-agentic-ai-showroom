# Tools, MCP, and Telemetry

So far, we've seen how we can query a given model within the Llama Stack Playground and receive *static information* based on the model's training data. For many use cases—such as troubleshooting a broken CI/CD pipeline—we need the model to access *real-time information* from external systems. The capability that allows a model to interact with an external system is called a *tool*, and the ability to use tools to achieve a given objective is a core feature of an *AI agent*.

For our use case, we need the following tools:

- **Read container logs**: fetch logs from the container that ran the failing pipeline step
- **Web search**: query a web search engine for potential resolutions
- **Create GitHub issue**: open a new issue in a GitHub repository with an error summary

There are three ways to use tools with Llama Stack:

- **Built-in tool providers** — Llama Stack includes providers like websearch (via [Tavily](https://docs.tavily.com/documentation/about)). Good for getting started quickly.
- **MCP integration** — Connect [Model Context Protocol](https://modelcontextprotocol.io/docs/getting-started/intro) servers as remote tool providers. MCP is an open standard for connecting AI applications to external systems—think of it as a USB-C port for AI. We'll use this for OpenShift and GitHub.
- **Client-side tools** — Implement tools directly in client code. Useful for early prototyping but not recommended for production since it prevents reuse.

In this module we will:

1. Set up the Tavily API key for websearch
2. Fork the lab repo and create a GitHub token
3. Deploy the GitHub MCP server
4. Apply **one combined ConfigMap update** that adds all tools, MCP integrations, and observability telemetry
5. Test everything in the Playground

## Llama Stack Configuration Overview

As part of Red Hat AI, a Llama Stack instance is represented by the LlamaStackDistribution custom resource (CR). The server's behavior—including which tools and providers are available—is controlled by a `run.yaml` stored in the `llama-stack-config` ConfigMap.

Rather than editing this ConfigMap multiple times across several modules, we'll prepare all prerequisites first, then make a single comprehensive update.

## Step 1: Set Up Websearch (Tavily)

**Tavily search token**: Gather your Tavily web search API Key.

1. Set up a [Tavily](https://app.tavily.com) API key for web search. Log in using a GitHub account of one of your team members.

![Create Tavily API Key](images/tavily-apikey.png)

2. Create a new secret object using this specification while replacing the PLACEHOLDER with your Tavily key. Use the `+` icon in the UI or your terminal:

```yaml
kind: Secret
apiVersion: v1
metadata:
  name: new-tavily-search-key
  namespace: <USER_NAME>-llama-stack
stringData:
  tavily-search-api-key: <YOUR_TAVILY_API_KEY>
type: Opaque
```

3. Update the Llama Stack distribution to reference this new secret. You must use `oc edit` or `oc patch` (not the UI).

```bash
oc edit llamastackdistribution/<USER_NAME>-llama-stack -n <USER_NAME>-llama-stack
```

Replace `tavily-search-key` with `new-tavily-search-key` in the TAVILY_API_KEY secret reference.

![Updated Llama Stack config](images/llamastackdistribution-updated-key.png)

## Step 2: Set Up GitHub Access and MCP Server

We need GitHub integration for two purposes: the agent will create issues in your repository, and we'll use a GitHub MCP server to expose GitHub tools to Llama Stack.

### Fork the repository and create a token

1. **Fork the repository**: Fork the [lab repository](https://github.com/rhpds/etx-agentic-ai-gitops) to your personal GitHub account.

![GitHub Repo Fork](images/github-fork.png)

2. **Enable Issues** for your fork: **Settings** > **General** > **Features** > **Issues** (disabled by default for forks).

![GitHub Repo Enable Issues](images/github-repo-enable-issues.png)

3. **Create a GitHub personal access token**:
   1. Click your user icon > **Settings** > **Developer Settings** > **Personal Access Tokens** > **Fine-grained personal access tokens**
   2. **Generate a new token** — name it e.g. `agent-lab`
   3. **Repository access**: Select the repository you forked above
   4. **Permissions**:

      **Commit statuses**: Read-Only<br>
      **Content**: Read-Only<br>
      **Issues**: Read and Write<br>
      **Metadata**: Read-Only (auto-added)<br>
      **Pull requests**: Read-Only

![GitHub Repo Perms](images/github-repo-perms.png)

   5. Generate the token and save it — you'll need it in a moment.

### Deploy the GitHub MCP server

1. Git clone your forked repository in the browser terminal or locally. If using the showroom terminal, authenticate to GitHub first.

2. Authenticate to your cluster if working locally:

```bash
<LOGIN_COMMAND>
```

3. Review the file at `etx-agentic-ai-gitops/lab-resources/github-mcp.yaml`. It creates a **secret** (GitHub credentials), **deployment**, and **service** for the GitHub MCP server.

4. Replace the token placeholder `<YOUR_GITHUB_PERSONAL_ACCESS_TOKEN>` in the secret with your token. Save the file.

5. Deploy:

```sh
oc apply -f etx-agentic-ai-gitops/lab-resources/github-mcp.yaml -n <USER_NAME>-llama-stack
```

6. **If you deployed locally**, revert the file edit and delete the token to avoid accidental commits.

## Step 3: Apply the Combined ConfigMap Update

Now we'll update the `llama-stack-config` ConfigMap once with everything: the MCP tool runtime, all tool groups (websearch, OpenShift, GitHub), and OpenTelemetry telemetry.

1. Within the `<USER_NAME>-llama-stack` project, navigate to ConfigMaps and select `llama-stack-config`.

![LlamaStack ConfigMap location](images/llamastack-configmap-location.png)

2. Click on the YAML tab.

![LlamaStack ConfigMap](images/llamastack-configmap.png)

3. Replace the entire `run.yaml` content with the following. This is the complete final configuration including all tool providers and telemetry:

```yaml
kind: ConfigMap
apiVersion: v1
metadata:
  name: llama-stack-config
data:
  run.yaml: |
    # Llama Stack configuration
    version: '2'
    image_name: rh
    apis:
    - inference
    - tool_runtime
    - agents
    - telemetry
    - vector_io
    models:
      - metadata: {}
        model_id: granite-3-2-8b-instruct
        provider_id: vllm
        provider_model_id: granite-3-2-8b-instruct
        model_type: llm
    providers:
      inference:
      - provider_id: vllm
        provider_type: "remote::vllm"
        config:
          url: https://litellm-prod.apps.maas.redhatworkshops.io/v1
          context_length: 4096
          api_token: ${env.DEFAULT_MODEL_API_TOKEN}
          tls_verify: true
      tool_runtime:
      - provider_id: tavily-search
        provider_type: remote::tavily-search
        config:
          api_key: ${env.TAVILY_API_KEY}
          max_results: 3
      - provider_id: model-context-protocol
        provider_type: remote::model-context-protocol
        config: {}
      agents:
      - provider_id: meta-reference
        provider_type: inline::meta-reference
        config:
          persistence_store:
            type: sqlite
            db_path: ${env.SQLITE_STORE_DIR:=~/.llama/distributions/rh}/agents_store.db
          responses_store:
            type: sqlite
            db_path: ${env.SQLITE_STORE_DIR:=~/.llama/distributions/rh}/responses_store.db
      telemetry:
      - provider_id: meta-reference
        provider_type: inline::meta-reference
        config:
          service_name: "${env.OTEL_SERVICE_NAME:=}"
          sinks: ${env.TELEMETRY_SINKS:=console}
          otel_exporter_otlp_endpoint: ${env.OTEL_EXPORTER_OTLP_ENDPOINT:=}
          sqlite_db_path: /opt/app-root/src/.llama/distributions/rh/trace_store.db
    server:
      port: 8321
    tools:
      - name: builtin::websearch
        enabled: true
    tool_groups:
    - provider_id: tavily-search
      toolgroup_id: builtin::websearch
    - toolgroup_id: mcp::openshift
      provider_id: model-context-protocol
      mcp_endpoint:
        uri: http://ocp-mcp-server.mcp-openshift.svc.cluster.local:8000/sse
    - toolgroup_id: mcp::github
      provider_id: model-context-protocol
      mcp_endpoint:
        uri: http://github-mcp-server:80/sse
```

Here's what we added compared to the default configuration:

- **`model-context-protocol` tool runtime** — enables MCP-based tool providers
- **`mcp::openshift` tool group** — connects the pre-deployed OpenShift MCP server (for reading cluster resources, pod logs, etc.)
- **`mcp::github` tool group** — connects the GitHub MCP server you just deployed (for creating issues, listing repos, etc.)
- **`telemetry` provider** — enables OpenTelemetry distributed tracing so we can observe agent behavior in Tempo later
- **`telemetry` API** — registered in the `apis` list (replaces `safety` which we removed since we're not using guardrails in this lab)

> **Note:** The process of registering MCP servers in Red Hat AI 3.x is different than these steps. Instead of editing `llama-stack-config` directly, there is a separate ConfigMap resource for MCP servers.

4. Hit `Save`. The Llama Stack operator restarts Llama Stack since its configuration changed. This may take several minutes.

5. Restart the playground by deleting its pod (starting with `llama-stack-playground`). Wait until it's Ready and Running.

![Llama Stack Playground restart](images/llamastack-playground-restart.png)

## Step 4: Test the Tools in the Playground

### Test Websearch

1. Refresh the Playground in the browser. Select **Agent-based**, then select the built-in **websearch** tool.

![LlamaStack Playground with websearch](images/llamastack-playground-websearch.png)

2. Ask:

```text
What is the weather today in Brisbane?
```

The LLM now answers with real-time data provided by the websearch tool.

![LlamaStack Playground regular websearch](images/llamastack-playground-websearch-regular.png)

### Test OpenShift MCP Tools

1. Select the **openshift** MCP Server entry in the tool groups.

![Llama Stack MCP OpenShift tool](images/llamastack-playground-mcp-openshift.png)

2. Expand `Tools from` to review the available tools.

![Llama Stack MCP OpenShift tool list](images/llamastack-playground-mcp-openshift-tools.png)

3. Try:

```text
List the pods in the mcp-openshift namespace.
```

4. Experiment with different prompts to activate the various OpenShift tools.

> **Tip:** Not all prompts work perfectly every time. This is a common reality with tool calling and LLMs. Larger, more capable models tend to perform more reliably. Granite does well but isn't perfect.

### Test GitHub MCP Tools

1. Select **Agent-based** with the **github** MCP tool provider. Try (replace with your GitHub user):

```text
List the branches of the ${YOUR_GITHUB_USER}/etx-agentic-ai-gitops repository.
```

![LlamaStack MCP GitHub](images/llama-playground-mcp-github-chat.png)

2. Review the list of GitHub tools available:

![LlamaStack MCP OpenShift tool list](images/llamastack-playground-mcp-github-tools.png)

3. **Create a GitHub issue** — try this prompt (replace `${YOUR_GITHUB_USER}`):

```text
Create a GitHub issue for a fake error in order to demonstrate GitHub tool usage.

Use the "create_issue" tool with these tool parameters:
{"arguments": {"owner": "${YOUR_GITHUB_USER}", "repo": "etx-agentic-ai-gitops", "title": "Fake Error: Agentic AI Service Unresponsive", "body": "The Agentic AI service is not responding. This is a fake error report."}
Do not add any optional parameters.
```

![LlamaStack Playground Github issue prompt](images/playground-github-issue.png)

> **Note:** If unsuccessful, try refreshing the playground.

4. Confirm that the issue was created in your repo.

![Github issue](images/github-issue.png)

## Agent Types: Regular vs. ReAct

As you experiment in the Playground, you'll notice the option to use **Regular** or **ReAct** agents.

**Regular agents** follow a straightforward flow: receive input → decide if a tool is needed → call the tool → process the response → answer.

![Regular agent workflow](images/agent.png)

**ReAct agents** use a **Re**asoning + **Act**ing loop: Think about what tool to use → Act by calling it → Observe the result → decide if more steps are needed. This makes ReAct more flexible for complex, multi-step tasks.

![ReAct Agent](images/react-agent.png)

## Summary

In this module, we configured our entire Llama Stack tooling and telemetry setup in a single ConfigMap update:

- **Websearch** via the built-in Tavily provider
- **OpenShift tools** via the Kubernetes MCP server
- **GitHub tools** via the GitHub MCP server
- **OpenTelemetry telemetry** for distributed tracing (we'll view traces after running the agent)

We also tested each tool integration in the Playground and reviewed how Regular and ReAct agents handle tool invocation differently.

We've now got all the pieces we need to build and deploy our CI/CD agent. Let's jump in!
