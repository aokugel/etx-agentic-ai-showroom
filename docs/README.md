# 🤖 ETX Agentic AI Super Lab

## 🪄 Customize The Instructions
The box at the top of the page allows you to load the docs with your lab variables prefilled. Fill in the following fields and click **Save**:

1. **Username** — e.g. `user1`. This value will be used for namespaces and resource names.
2. **Password** — your lab password.
3. **Cluster Domain** — the `apps.*` portion of your OpenShift domain (see below).

The values are persisted in your browser's local storage. Click **Clear** to reset all saved values.

* After saving, you should see your username and password below:

    ```bash
    <USER_NAME>
    <PASSWORD>
    ```

* For the cluster domain, enter only the `apps.*` portion of your OpenShift domain. For example, if your console address is <code class="language-yaml">https://console-openshift-console.apps.cluster.example.com/</code>
 then enter `apps.cluster.example.com` in the cluster domain field.

    You should see your cluster domain below:

    ```bash
    <CLUSTER_DOMAIN>
    ```

## 🦆 Conventions
When running through the exercises, we've tried to call out where things need replacing. If you saved your details above, most `<PLACEHOLDER>` values will be filled in automatically. If any remain, replace them manually with your actual values. For example, if your username is `user1` and you see `<USER_NAME>`, replace it with `user1` like so:
    <div class="highlight" style="background: #f7f7f7">
    <pre><code class="language-bash">
    name: &lt;USER_NAME&gt;
    # ^ this becomes
    name: user1
    </code></pre></div>

There are lots of code blocks for you to copy and paste. They have a little ✂️ icon on the right when you move your cursor over the code block.

```bash
    echo "like this one :)"
```

**Important:** Not all code blocks are meant to be copied. Blocks without the ✂️ icon are **expected output**. Use them to validate your results or YAML against the given block.

## 📋 Lab Overview

This lab illustrates how agentic AI can be applied to help solve a real-world business problem. You will configure tools for an AI agent, deploy it as a service, trigger it from a CI/CD pipeline, and observe its behavior through distributed tracing.

By the end of this workshop, participants will have:

- ✅ A fully functional AI agent that can autonomously handle CI/CD failures
- ✅ Hands-on experience integrating tools (websearch, OpenShift, GitHub) into an agentic framework
- ✅ Practical knowledge of deploying and triggering agents in real-world workflows
- ✅ Experience with observability and tracing of AI systems

## 🗺️ Workflow

1. Introduction and Overview
2. Llama Stack and the Playground
3. Configure Tools and Observability
4. Deploy and Run the Agent
5. View and Analyze Traces

Alright, let's get started! Head over to the first exercise and begin your Agentic AI journey! 🏃💨
