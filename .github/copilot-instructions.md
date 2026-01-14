# Copilot Instructions for Orion Near-RT RIC Analysis

## Project Overview

This workspace contains Jupyter notebooks for analyzing O-RAN Near-RT RIC system performance:
- **Message processing times**: End-to-end latency analysis across RIC components
- **Resource usage**: CPU and memory consumption by Kubernetes namespace

## Architecture Context

The analysis covers a multi-layer O-RAN architecture with these Kubernetes namespaces:

| Namespace | Components | Purpose |
|-----------|------------|---------|
| `frontend` | User interface | Intent submission |
| `smo` | MCP Client/Server | Intent management via LLM |
| `ricrapp` | rApp | Intent-to-A1 policy translation |
| `nonrtric` | Policy Management, A1 Controller | Non-RT RIC policy handling |
| `ricplt` | A1 Mediator, E2Term, E2Mgr | Near-RT RIC platform |
| `ricxapp` | orion-xapp, e2sim | xApps and E2 node simulator |

## Notebook Patterns

### Log Analysis (`times/`)
- Fetches logs via `kubectl` with subprocess calls
- Parses multiple timestamp formats (ISO 8601, mdclog JSON, Python logging, e2sim custom)
- Extracts events using regex patterns verified against source code
- Correlates events by `policy_id` for accurate processing time calculation

Key patterns in [message_processing_time_v2.ipynb](times/message_processing_time_v2.ipynb):
```python
# Pod selection by label
POD_SELECTORS = {
    "orion-xapp": {"namespace": "ricxapp", "label": "app=ricxapp-orion-xapp"},
    # ...
}

# Event patterns verified against component source code
MESSAGE_PATTERNS = {
    "orion-xapp": {
        "a1_policy_received": r"Received A1 Policy",  # main.cpp:83
        "control_sent": r"Sending Control Request for Slice SLA Policy",  # main.cpp:138
    },
}
```

### Resource Metrics (`resources/`)
- Queries Prometheus via HTTP API (PromQL)
- Calculates mean ± std dev over configurable time windows
- Separates e2sim metrics from ricxapp namespace totals

Key pattern in [namespace_resources.ipynb](resources/namespace_resources.ipynb):
```python
PROMETHEUS_URL = "http://kube-prometheus-stack-prometheus.monitoring.svc.cluster.local:9090"
```

## Development Setup

```bash
# Activate the Python 3.12 virtual environment
source .venv/bin/activate

# Required packages: pandas, matplotlib, seaborn, numpy, requests
```

## Conventions

1. **Plot styling**: Use `sns.set_theme(style='whitegrid', font_scale=1.7)` for consistency
2. **Color scheme**: Purple (SMO), Green (rApp/Non-RT), Blue (Near-RT RIC), Orange (E2 Node)
3. **Output formats**: Save charts as both PNG (300 DPI) and PDF
4. **Time windows**: Use `LOG_SINCE_MINUTES` for active components, `LOG_SINCE_HOURS` for SMO/rApp (less frequent logging)

## Data Flow for Analysis

```
Frontend → SMO (MCP) → rApp → Non-RT RIC → A1 Mediator → xApp → E2Term → E2 Node
```

Processing time pairs track latency between adjacent stages. When adding new analysis:
1. Add pod selector to `POD_SELECTORS` with correct namespace and label
2. Add message patterns to `MESSAGE_PATTERNS` (verify against actual log output)
3. Add event pairs to `event_pairs` in `calculate_processing_times()` with realistic `max_time` thresholds
