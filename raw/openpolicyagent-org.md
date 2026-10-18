---
url: https://www.openpolicyagent.org/
date_fetched: 2026-10-18
---

```
{}
```
```
{
  "account": {
    "state": "open"
  },
  "user": {
    "risk_score": "low"
  },
  "transaction": {
    "amount": 950
  }
}
```
```
# Run your first Rego policy!
package payments
default allow := false
allow if {
	input.account.state == "open"
	input.user.risk_score in ["low", "medium"]
	input.transaction.amount <= 1000
}
# Open in the Rego Playground to see the full example.
```
**Developer Productivity:** OPA helps teams focus on delivering business value by decoupling policy from application logic. Security & platform teams centrally manage shared policies, while developer teams extend them as needed within the policy system.

**Performance:** Rego, our domain-specific policy language, is built for speed. By operating on pre-loaded, in-memory data, OPA acts as a fast policy decision point for your applications.

**Audit & Compliance:** OPA generates comprehensive audit trails for every policy decision. This detailed history supports auditing and compliance efforts and enables decisions to be replayed for analysis or debugging.

Interested to see more? Checkout the Maintainer Track Session from KubeCon.

OPA is a general-purpose policy engine that unifies policy enforcement across the stack. OPA provides a high-level declarative language that lets you specify policy for a wide range of use cases. You can use OPA to enforce policies in applications, proxies, Kubernetes, CI/CD pipelines, API gateways, and more.

The examples below are interactive! Use the **Evaluate** button to see output for the given rego and input. Then edit the rego or the input (or both) to see how the output changes.

Applications can directly integrate with OPA using the SDKs or REST API. This is great when your application needs to make domain specific runtime decisions.

```
package application.authz
# Only owner can update the pet's information. Ownership
# information is provided as part of the request data from
# the application.
default allow := false
allow if {
	input.method == "PUT"
	some petid
	input.path = ["pets", petid]
	input.user == input.owner
}
```
```
{
  "allow": false
}
```
```
{
  "role": "staff",
  "owner": "bob@example.com",
  "path": [
    "pets",
    "pet113-987"
  ],
  "user": "alice@example.com"
}
```
```
{}
```
OPA has a native integration with the Envoy External Authorization API. This example shows a simple policy using the input document supplied from that API to authorize requests.

Browse more Envoy Examples on the playground.

```
package envoy.http.public
headers := input.attributes.request.http.headers
default allow := false
allow if {
	input.attributes.request.http.method == "GET"
	input.attributes.request.http.path == "/"
}
allow if headers.authorization == "Basic charlie"
```
true

```
{
  "attributes": {
    "request": {
      "http": {
        "headers": {
          "authorization": "Basic bob"
        },
        "method": "GET",
        "path": "/",
        "protocol": "HTTP/1.1"
      }
    }
  }
}
```
```
{}
```
OPA can be used to control which Kubernetes resources can be created in a given cluster. By configuring the Kubernetes API to send admission requests to OPA, you can create custom policies to enforce your organization's rules.

Browse more Kubernetes Examples on the playground.

```
package kubernetes.validating.existence
deny contains msg if {
	value := input.request.object.metadata.labels.costcenter
	not startswith(value, "cccode-")
	msg := sprintf("Costcenter must start `cccode-`; found `%v`", [value])
}
deny contains msg if {
	not input.request.object.metadata.labels.costcenter
	msg := "Every resource must have a costcenter label"
}
```
[ "Costcenter must start `cccode-`; found `engineering`" ]

```
{
  "kind": "AdmissionReview",
  "request": {
    "kind": {
      "kind": "Pod",
      "version": "v1"
    },
    "object": {
      "metadata": {
        "name": "myapp",
        "labels": {
          "costcenter": "engineering"
        }
      },
      "spec": {
        "containers": [
          {
            "image": "nginx",
            "name": "nginx-frontend"
          },
          {
            "image": "mysql",
            "name": "mysql-backend"
          }
        ]
      }
    }
  }
}
```
```
{}
```
OPA lets you enforce fine-grained policies over which tools an AI agent can call, what parameters are permitted, and how those tools can be used.

```
package coding.tools
deny contains $"Tool {tc.tool} is not allowed" if {
	some tc in input.tool_calls
	tc.tool in _disallowed_tools
}
deny contains $"WebFetch can only load from HTTPS URLs" if {
	some tc in input.tool_calls
	tc.tool == "WebFetch"
	not startswith(tc.params.url, "https://")
}
deny contains $"WebSearch cannot load more than {_max_results} results" if {
	some tc in input.tool_calls
	tc.tool == "WebSearch"
	tc.params.num_results > _max_results
}
deny contains $"Tool timeout cannot be more than 10s" if {
	some tc in input.tool_calls
	tc.params.timeout > 10000
}
_disallowed_tools := {"Bash", "Write", "Edit"}
_max_results := 10
```
[ "Tool Bash is not allowed", "WebFetch can only load from HTTPS URLs" ]

```
{
  "tool_calls": [
    {
      "tool": "Bash",
      "params": {
        "command": "ls -la",
        "timeout": 5000
      }
    },
    {
      "tool": "WebFetch",
      "params": {
        "url": "http://example.com",
        "timeout": 5000
      }
    }
  ]
}
```
```
{}
```
