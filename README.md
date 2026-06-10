# Intent-aware Orchestration in Open RAN - Analysis Repository

[![arXiv](https://img.shields.io/badge/arXiv-Pending-b31b1b.svg)](https://arxiv.org/)

This is the main repository for the research paper **"Intent-aware Orchestration in Open RAN"** (available on [arXiv](https://arxiv.org/)). It contains the core analysis, evaluations, models, and results presented in the paper.

## ORION Architecture and Sub-repositories

The complete system architecture, dubbed **ORION**, is distributed across several sub-repositories. Each repository corresponds to a distinct architectural layer and component proposed in the paper, spanning from the SMO down to the simulated RAN nodes:

### SMO & Intent Management Level (MCP Client & Server)
* **[Intent Management SMO App](https://github.com/gabiSmachado/intent_management-smoapp)**: Implements the **Frontend (Streamlit)** and the **MCP Client**. It receives natural language requests, orchestrates LLM workflows, and acts as the intent assistance plane.
* **[Intent Assistance Interface](https://github.com/gabiSmachado/intent_assistance_interface)**: Implements the **MCP Server** and **CAMARA Mock API**. Provides tool execution endpoints, performs validation using CAMARA's *NetworkSliceBooking* API schema, and ensures admission control.

### Non-RT RIC Level (OrApp)
* **[Orion rApp](https://github.com/gabiSmachado/orion-rapp)**: The **OrApp**. Acts as the intent composition function, translating the validated structured intents into precise A1 policy payloads with SLA objectives and specific slice types (e.g., eMBB, URLLC).
* **[Interpretation Scheme SMO App](https://github.com/gabiSmachado/interpretation_scheme-smoapp)**: Supplementary SMO application for interpreting high-level intents into actionable policies.

### Near-RT RIC Level (OxApp)
* **[orion-xapp](https://github.com/zanattabruno/orion-xapp/)**: The **Orion xApp (OxApp)**. A C++ application terminating the A1 interface. It dynamically translates SLA policy objectives from the OrApp into precise E2SM-RC control messages, calculating real-time PRB quotas based on cell capacity.

### RAN Level (E2 Node Emulator)
* **[e2sim-rc](https://github.com/alexandre-huff/e2sim-rc)**: The **E2Sim (e2sim-rc)**. An extended OSC E2 Simulator emulating a gNB. Supports E2SM-RC Control Style 2 (Radio Resource Allocation) and performs physical resource block (PRB) capacity validation acting as a resource guard.

## End-to-End Intent Workflow

When an intent is fully processed, it follows an advanced 6-phase lifecycle bridging these components:
1. **Ingestion**: natural language input arrives via the SMO App Frontend.
2. **Translation**: the MCP Client leverages an LLM and the MCP Server's tool definitions to structure a schema-compliant payload.
3. **Validation**: the MCP Server verifies correctness against the CAMARA slice booking API semantics.
4. **Composition**: the OrApp maps these structures to concrete SLA templates and dispatches an A1 Policy.
5. **Deployment**: the OxApp parses the A1 JSON and enforces execution limits at the E2 Node (E2Sim-RC).
6. **Monitoring and adaptation**: continuing metrics track KPIs (e.g. latency), adjusting loops when required.

## Analysis Repository Contents

This main repository (`analysis`) focuses on the evaluation side of the research. It provides the scripts, data, and results used to generate the findings presented in the paper. 

Key directories include:
* `classification_accuracy/`: Scripts and data related to model classification accuracy evaluations.
* `deepeval/`: Evaluation frameworks and deep metric tracking components.
* `llm/`: Large Language Model (LLM) integration, prompt testing, and experiments.
* `times/`: Time-related metrics, latency, and performance efficiency evaluations.
* `figures/`: Generated figures and plots used directly or indirectly in the paper.
* `resources/`: Additional datasets or shared files required for the analysis. Includes the intent classification dataset, publicly available at [`intent_dataset.csv`](https://github.com/gabiSmachado/ORION/blob/main/src/test/intent_dataset.csv).

---
*For functional details and the codebase of each specific Open RAN component, please navigate to the respective repository linked above.*
