# SigPulse: Signals Planning and Monitoring System

**Project Foundation & Architectural Conception Document**

## 1. Executive Summary

### 1.1 Vision

SigPulse is the unified operational command center for the organization's entire communications and signal infrastructure across Algerian territory. The vision is to eliminate operational silos by bridging financial acquisition, physical infrastructure deployment, active network telemetry, field maintenance, and **AI-driven strategic foresight** into a single, continuous, and highly secure lifecycle management platform. By leveraging a dynamic **Digital Twin**, the system transitions operations from reactive troubleshooting to proactive crisis management and predictive resilience.

### 1.2 Goal

To deploy an enterprise-grade, Domain-Driven Design (DDD) platform that enables the organization to proactively plan geographical network expansions, strictly govern the procurement of hardware, enforce multi-level security, actively monitor multi-format traffic (voice, video, data) in real-time, simulate catastrophic failures, and utilize artificial intelligence to drive strategic decision-making.

### 1.3 Core Objectives

1. **End-to-End Traceability:** Track every hardware component from the initial planning requisition, through formal tendering and procurement, to physical installation and eventual decommissioning.
2. **Geographical & Mathematical Validation:** Mandate that every new edge site expansion connects to the main backbone with a mathematically validated Radio Frequency (RF) link budget and Line-of-Sight (LoS) clearance before hardware is purchased.
3. **Zero-Trust Role Segregation:** Enforce strict operational boundaries between the Planning, Procurement, Maintenance, and Command units.
4. **Resilient Telemetry:** Isolate high-frequency network monitoring from the relational inventory database to ensure system stability during massive network events.
5. **Automated Incident Resolution:** Instantly cross-reference real-time telemetry drops with the static infrastructure topology to pinpoint root causes and execute redundant link failovers automatically.
6. **Data-Driven Decision Making (AI):** Utilize machine learning models on historical telemetry and maintenance data to predict hardware failures, recommend vendor reliability, and optimize QoS routing.
7. **Proactive Crisis Simulation:** Maintain a live "Digital Twin" of the infrastructure, allowing architects to inject synthetic faults (e.g., severe weather dropping regional links) to validate failover survivability before a real crisis occurs.

---

## 2. Functional Conception

The system is organized into eight interconnected operational pillars:

### 2.1 Topography & Deployment Planning

* **GIS Terrain Integration:** Planners use map interfaces to plot new edge sites across wilayas, defining structural tower and power requirements.
* **Link Budget & Capacity:** Calculates transmission power, receiver sensitivity, and free-space path loss to validate link viability.
* **Backbone Hierarchy:** Enforces rules ensuring every new `EDGE_SITE` logically and physically connects back to a `REGIONAL_DISTRIBUTION` or `BACKBONE_CORE` node.

### 2.2 End-to-End Procurement

* **Automated BOM:** Validated site plans generate a Bill of Materials.
* **Statutory Workflows:** Manages purchase requisitions, budget commitments, tender bidding, contract awards, and formal Purchase Orders.
* **Warehouse Receiving:** Enforces Quality Assurance (QA) inspections before procured items enter active stock inventory.

### 2.3 CMDB & Dynamic Inventory

* **Physical Asset Tracking:** Manages serial numbers, MAC addresses, and modular parent-child hardware chassis.
* **Configuration Baselines:** Stores cryptographic hashes of authorized hardware configurations to prevent unauthorized tampering.

### 2.4 Multi-Format Traffic & QoS

* **Traffic Segmentation:** Allocates logical bandwidth queues across physical links to separate `VOICE` (telephony), `VISIOPHONE` (video), and `CLASSIFIED_DATA`.
* **Prioritization:** Ensures massive data transfers cannot saturate the link and drop low-latency communication formats.

### 2.5 Continuous Telemetry & Failover

* **Non-Blocking Polling:** Executes thousands of concurrent health checks (uptime, latency, jitter, QoS queue saturation).
* **Redundancy Orchestration:** Pre-maps primary and backup links. Automatically reroutes backbone traffic upon detecting a primary path loss-of-signal.

### 2.6 ITSM & Maintenance

* **Automated Ticketing:** Converts sustained telemetry alarms into actionable Trouble Tickets.
* **Field Dispatch:** Issues Field Work Orders to technicians and logs spare part consumption.
* **Preventive Maintenance:** Schedules routine servicing (e.g., generator checks, tower inspections).

### 2.7 AI Analytics & Statistical Decision Support

* **Predictive Maintenance (Helper):** Analyzes historical hardware degradation (e.g., a specific optical switch model failing frequently at high temperatures) and flags deployed assets for replacement before they fail.
* **Vendor Reliability Scoring:** Aggregates procurement SLAs, QA inspection pass rates, and field RMA frequency to generate statistical rankings, assisting the Procurement Unit during tender evaluations.
* **Capacity Forecasting:** Uses machine learning on TimescaleDB metrics to forecast when specific regional backbone links will hit bandwidth saturation based on current expansion rates.

### 2.8 Infrastructure Simulator & Digital Twin

* **Synthetic Fault Injection:** Allows the Planning and Command units to simulate the loss of specific nodes, severed links, or widespread power grid failures (e.g., simulating a blackout in a specific wilaya).
* **Cascading Impact Analysis:** Calculates exactly how traffic will reroute during the simulated crisis, identifying "choke points" where backup links would be overwhelmed by the diverted traffic.
* **Redundancy Validation:** Mathematically proves that the `RedundancyGroup` definitions will actually keep `VOICE` and `VISIOPHONE` services online during a catastrophic Layer-1 failure.

---

## 3. Technical Architecture & Infrastructure

SigPulse relies on a highly scalable, hybrid-database architecture utilizing the Spring Boot ecosystem for enterprise reliability, integrated with specialized data science runtimes.

| Architectural Layer | Recommended Technology | Justification / Role |
| --- | --- | --- |
| **Client Presentation** | React JS (TypeScript) | Single Page Application (SPA) offering distinct unit-based workspaces. |
| **Visualizations** | React Flow & Mapbox | React Flow for logical topology mapping; Mapbox for GIS terrain deployment. |
| **Core API & Logic** | Spring Boot (Java) | Central DDD engine enforcing security, workflow state machines, and business rules. |
| **Telemetry Worker** | Spring WebFlux / Virtual Threads | Executes highly concurrent, non-blocking SNMP/ICMP network polling. |
| **Relational Hub** | PostgreSQL (Spring Data JPA) | Ensures strict ACID compliance and referential integrity for CMDB and Procurement. |
| **Monitoring Hub** | TimescaleDB | Time-series database specifically optimized for high-volume telemetry ingestion. |
| **AI & Simulation Engine** | Python (FastAPI, PyTorch/Scikit-Learn) | A dedicated microservice querying historical PostgreSQL/TimescaleDB data to execute graph-based crisis simulations and train predictive statistical models. |
| **Event Broker** | Apache Kafka | Consumes offline alerts and broadcasts failover commands, triggering both live re-routing and AI model retraining. |

---

## 4. Data Entity Schema

The persistence layer is modeled on a Domain-Driven Design (DDD) architecture, cleanly segregating the system into modular contexts.

### 4.1 Enterprise Administration & Acquisition Layer

Handles Identity, Security Clearances, and the Procurement lifecycle.

```mermaid
erDiagram
    %% IDENTITY & SECURITY
    UserJpaEntity ||--o{ LocalCredentialJpaEntity : "authenticates"
    UserJpaEntity ||--o{ SubjectSecurityAttributeJpaEntity : "has_attributes"
    UserJpaEntity ||--o{ SecurityAuditLogJpaEntity : "audited_via"
    AuthorizationPolicyJpaEntity ||--|{ AuthorizationPolicyRuleJpaEntity : "enforces"

    UserJpaEntity {
        UUID id PK
        String username UK
        ClearanceLevel classification_level
        String organization_unit_code
    }
    LocalCredentialJpaEntity {
        UUID id PK
        UUID user_id FK
        String password_hash
        Instant locked_until
    }
    SubjectSecurityAttributeJpaEntity {
        UUID id PK
        UUID user_id FK
        String compartment_code
    }
    AuthorizationPolicyJpaEntity {
        UUID id PK
        String policy_code UK
        PolicyEffect default_effect
    }
    AuthorizationPolicyRuleJpaEntity {
        UUID id PK
        UUID policy_id FK
        String resource_pattern
        String spel_condition
    }
    SecurityAuditLogJpaEntity {
        UUID id PK
        Instant timestamp
        String action_type
        String hmac_integrity_seal
    }

    %% PROCUREMENT
    VendorJpaEntity ||--o{ ContractJpaEntity : "awarded"
    VendorJpaEntity ||--o{ VendorBidJpaEntity : "submits"
    PurchaseRequisitionJpaEntity ||--|{ RequisitionLineItemJpaEntity : "contains"
    PurchaseRequisitionJpaEntity ||--|| BudgetCommitmentJpaEntity : "funds"
    TenderCallJpaEntity ||--|{ TenderLotJpaEntity : "divided_into"
    TenderLotJpaEntity ||--o{ VendorBidJpaEntity : "bids_on"
    ContractJpaEntity ||--o{ PurchaseOrderJpaEntity : "governs"
    PurchaseOrderJpaEntity ||--|{ POLineItemJpaEntity : "specifies"
    PurchaseOrderJpaEntity ||--o{ GoodsReceiptNoteJpaEntity : "delivers"
    GoodsReceiptNoteJpaEntity ||--|| QualityInspectionReportJpaEntity : "qa_check"
    PurchaseOrderJpaEntity ||--o{ VendorInvoiceJpaEntity : "invoices"
    VendorInvoiceJpaEntity ||--|| PaymentLiquidationJpaEntity : "pays"

    PurchaseRequisitionJpaEntity {
        UUID id PK
        String requisition_number UK
        RequisitionStatus status
    }
    RequisitionLineItemJpaEntity {
        UUID id PK
        UUID requisition_id FK
        int requested_quantity
    }
    BudgetCommitmentJpaEntity {
        UUID id PK
        UUID requisition_id FK
        BigDecimal committed_amount
    }
    TenderCallJpaEntity {
        UUID id PK
        String tender_reference UK
        TenderType tender_type
    }
    TenderLotJpaEntity {
        UUID id PK
        UUID tender_call_id FK
        BigDecimal ceiling_budget
    }
    VendorBidJpaEntity {
        UUID id PK
        UUID tender_lot_id FK
        UUID vendor_id FK
        BigDecimal financial_offer_amount
    }
    VendorJpaEntity {
        UUID id PK
        String company_name
        String contract_reference
    }
    ContractJpaEntity {
        UUID id PK
        UUID vendor_id FK
        String contract_number UK
    }
    PurchaseOrderJpaEntity {
        UUID id PK
        UUID contract_id FK
        String po_number UK
        POStatus status
    }
    POLineItemJpaEntity {
        UUID id PK
        UUID purchase_order_id FK
        int ordered_quantity
    }
    GoodsReceiptNoteJpaEntity {
        UUID id PK
        UUID purchase_order_id FK
        String grn_number UK
    }
    QualityInspectionReportJpaEntity {
        UUID id PK
        UUID grn_id FK
        InspectionVerdict verdict
    }
    VendorInvoiceJpaEntity {
        UUID id PK
        UUID purchase_order_id FK
        BigDecimal billed_amount
    }
    PaymentLiquidationJpaEntity {
        UUID id PK
        UUID invoice_id FK
        String payment_mandate_reference UK
    }

```

### 4.2 Physical & Logical Infrastructure Layer

Maps the CMDB hardware, geographical site deployments, RF frequency clearances, and logical network topologies.

```mermaid
erDiagram
    %% TOPOGRAPHY & DEPLOYMENT
    SiteLocationJpaEntity ||--|| CivilInfrastructureJpaEntity : "contains"
    SiteLocationJpaEntity ||--o{ DeploymentProjectJpaEntity : "upgrades"
    DeploymentProjectJpaEntity ||--|{ DeploymentMilestoneJpaEntity : "tracked_by"

    SiteLocationJpaEntity {
        UUID id PK
        String site_code UK
        BigDecimal latitude
        BigDecimal longitude
        InfrastructureTier tier
    }
    CivilInfrastructureJpaEntity {
        UUID id PK
        UUID site_id FK
        TowerStructureType tower_type
    }
    DeploymentProjectJpaEntity {
        UUID id PK
        UUID target_site_id FK
        ProjectPhase current_phase
    }
    DeploymentMilestoneJpaEntity {
        UUID id PK
        UUID project_id FK
        MilestoneCategory category
        MilestoneStatus status
    }

    %% CMDB & WAREHOUSE
    SiteLocationJpaEntity ||--o{ WarehouseLocationJpaEntity : "hosts"
    WarehouseLocationJpaEntity ||--|{ WarehouseBinJpaEntity : "contains"
    EquipmentModelJpaEntity ||--o{ AssetJpaEntity : "defines_spec"
    WarehouseBinJpaEntity ||--o{ AssetJpaEntity : "stores"
    AssetJpaEntity ||--|{ InterfacePortJpaEntity : "exposes"

    WarehouseLocationJpaEntity {
        UUID id PK
        UUID site_id FK
        String warehouse_code UK
    }
    WarehouseBinJpaEntity {
        UUID id PK
        UUID warehouse_id FK
        String bin_code UK
    }
    EquipmentModelJpaEntity {
        UUID id PK
        String model_name
        HardwareFamily hardware_family
    }
    AssetJpaEntity {
        UUID id PK
        UUID equipment_model_id FK
        String serial_number UK
        AssetLifecycleState lifecycle_state
    }
    InterfacePortJpaEntity {
        UUID id PK
        UUID asset_id FK
        String port_identifier
        PhysicalPortMedium medium_type
    }

    %% TOPOLOGY
    SiteLocationJpaEntity ||--o{ NetworkNodeJpaEntity : "hosts_node"
    AssetJpaEntity ||--|| NetworkNodeJpaEntity : "operates_as"
    InterfacePortJpaEntity ||--o{ NetworkLinkJpaEntity : "connects"
    NetworkLinkJpaEntity ||--o{ LinkBudgetProfileJpaEntity : "rf_validated_by"
    NetworkLinkJpaEntity ||--o{ RedundancyGroupJpaEntity : "primary/backup"

    NetworkNodeJpaEntity {
        UUID id PK
        UUID site_id FK
        UUID core_asset_id FK
        String management_ip UK
        OperationalState state
    }
    NetworkLinkJpaEntity {
        UUID id PK
        UUID source_port_id FK
        UUID target_port_id FK
        TransmissionMedium medium
        LinkOperationalStatus status
    }
    LinkBudgetProfileJpaEntity {
        UUID id PK
        UUID link_id FK
        BigDecimal tx_power_dbm
        boolean los_validated
    }
    RedundancyGroupJpaEntity {
        UUID id PK
        UUID primary_link_id FK
        UUID secondary_link_id FK
        boolean is_failover_active
    }

```

### 4.3 Active Operations & ITSM Layer

Handles high-frequency telemetry (TimescaleDB), QoS enforcement, automatic anomaly alerting, and corrective field maintenance.

```mermaid
erDiagram
    %% QOS
    ServiceProfileJpaEntity ||--o{ TrafficAllocationJpaEntity : "allocates"
    NetworkLinkJpaEntity ||--|{ TrafficAllocationJpaEntity : "carries"

    ServiceProfileJpaEntity {
        UUID id PK
        CommunicationFormat format_category
        int dscp_priority_tag
    }
    TrafficAllocationJpaEntity {
        UUID id PK
        UUID network_link_id FK
        UUID service_profile_id FK
        BigDecimal guaranteed_bandwidth_mbps
    }

    %% TELEMETRY & ALARMS
    MonitoringProbeAgentJpaEntity ||--o{ MonitoringThresholdJpaEntity : "evaluates"
    MetricDefinitionJpaEntity ||--o{ MonitoringThresholdJpaEntity : "monitors"
    MonitoringThresholdJpaEntity ||--o{ ActiveAlarmJpaEntity : "triggers"
    MetricDefinitionJpaEntity ||--o{ NodeTelemetryHypertable : "logs"
    NetworkNodeJpaEntity ||--o{ ActiveAlarmJpaEntity : "experiences"

    MonitoringProbeAgentJpaEntity {
        UUID id PK
        String agent_identifier UK
        AgentHealthStatus status
    }
    MetricDefinitionJpaEntity {
        UUID id PK
        String metric_code UK
    }
    MonitoringThresholdJpaEntity {
        UUID id PK
        UUID metric_definition_id FK
        BigDecimal critical_threshold
    }
    ActiveAlarmJpaEntity {
        UUID id PK
        UUID network_node_id FK
        AlarmSeverity severity
        AlarmLifecycleState lifecycle_state
    }
    NodeTelemetryHypertable {
        timestamptz timestamp PK
        UUID node_id FK
        BigDecimal metric_value
    }

    %% MAINTENANCE & ITSM
    ActiveAlarmJpaEntity ||--o{ TroubleTicketJpaEntity : "escalated_to"
    TroubleTicketJpaEntity ||--o{ FieldWorkOrderJpaEntity : "resolved_by"
    FieldWorkOrderJpaEntity ||--|| FieldInterventionReportJpaEntity : "reports"
    FieldWorkOrderJpaEntity ||--o{ SparePartConsumptionJpaEntity : "consumes"
    MaintenancePlanJpaEntity ||--o{ PreventiveInterventionJpaEntity : "schedules"

    TroubleTicketJpaEntity {
        UUID id PK
        UUID source_alarm_id FK
        String ticket_number UK
        TicketSeverity severity
        TicketStatus status
    }
    FieldWorkOrderJpaEntity {
        UUID id PK
        UUID trouble_ticket_id FK
        String work_order_number UK
        WorkOrderStatus status
    }
    FieldInterventionReportJpaEntity {
        UUID id PK
        UUID work_order_id FK
        boolean service_restored_verified
    }
    SparePartConsumptionJpaEntity {
        UUID id PK
        UUID work_order_id FK
        UUID defective_asset_id FK
        UUID replacement_asset_id FK
    }
    MaintenancePlanJpaEntity {
        UUID id PK
        UUID equipment_model_id FK
        int periodicity_days
    }
    PreventiveInterventionJpaEntity {
        UUID id PK
        UUID maintenance_plan_id FK
        LocalDate scheduled_date
        InterventionStatus status
    }

```

### 4.4 AI Analytics & Crisis Simulation Layer

Manages the Digital Twin topology states, synthetic fault injection scenarios, and predictive statistical models generated by the Python/AI microservice.

```mermaid
erDiagram
    %% AI ANALYTICS
    AiPredictionModelJpaEntity ||--o{ ReliabilityInsightJpaEntity : "generates"
    EquipmentModelJpaEntity ||--o{ ReliabilityInsightJpaEntity : "targeted_hardware"
    VendorJpaEntity ||--o{ VendorScorecardJpaEntity : "evaluated_in"

    AiPredictionModelJpaEntity {
        UUID id PK
        String model_name UK
        ModelType algorithm_type "RANDOM_FOREST, ARIMA, NEURAL_NET"
        BigDecimal accuracy_score
        Instant last_trained_at
    }
    ReliabilityInsightJpaEntity {
        UUID id PK
        UUID model_id FK
        UUID equipment_model_id FK
        BigDecimal probability_of_failure_pct
        int expected_time_to_failure_days
        String recommended_action "REPLACE, INSPECT, FIRMWARE_UPGRADE"
    }
    VendorScorecardJpaEntity {
        UUID id PK
        UUID vendor_id FK
        BigDecimal overall_reliability_score
        BigDecimal rma_frequency_pct
        BigDecimal sla_compliance_pct
    }

    %% INFRASTRUCTURE SIMULATOR (DIGITAL TWIN)
    SimulationScenarioJpaEntity ||--|{ InjectedFaultJpaEntity : "contains"
    SimulationScenarioJpaEntity ||--|| SimulationResultJpaEntity : "produces"
    InjectedFaultJpaEntity ||--o{ NetworkNodeJpaEntity : "targets_node"
    InjectedFaultJpaEntity ||--o{ NetworkLinkJpaEntity : "targets_link"
    SimulationResultJpaEntity ||--o{ CascadingImpactJpaEntity : "identifies_bottlenecks"

    SimulationScenarioJpaEntity {
        UUID id PK
        String scenario_name UK
        CrisisType crisis_category "WEATHER, POWER_GRID, CYBER_ATTACK, FIBER_CUT"
        String geographical_scope "e.g., Wilaya of Ouargla"
        UUID executed_by FK
    }
    InjectedFaultJpaEntity {
        UUID id PK
        UUID scenario_id FK
        UUID target_node_id FK
        UUID target_link_id FK
        int trigger_time_offset_sec
        FaultType fault_type "TOTAL_LOSS, SEVERE_DEGRADATION"
    }
    SimulationResultJpaEntity {
        UUID id PK
        UUID scenario_id FK
        BigDecimal overall_survivability_score
        int isolated_nodes_count
        boolean qos_voice_maintained
    }
    CascadingImpactJpaEntity {
        UUID id PK
        UUID result_id FK
        UUID saturated_link_id FK
        BigDecimal simulated_congestion_pct
        String failure_chain_description
    }

```

---

## 5. Core Operational Workflows

### A. Statutory Procurement to Ingestion Workflow

1. **Demand Generation:** A new transmission site project generates required `RequisitionLineItemJpaEntity` entries linked to a `PurchaseRequisitionJpaEntity`.
2. **AI Vendor Selection (Helper):** The Procurement Unit queries `VendorScorecardJpaEntity` to review historically reliable vendors before issuing the tender.
3. **Competitive Tendering:** A `TenderCallJpaEntity` is published; vendor proposals are evaluated and awarded.
4. **Contract Execution:** The `ContractJpaEntity` is finalized, establishing delivery dates.
5. **Delivery & QA:** Upon shipment arrival, warehouse personnel record a `GoodsReceiptNoteJpaEntity`. `QualityInspectionReportJpaEntity` confirms technical specs before items enter the active CMDB as `IN_STOCK`.

### B. Autonomous Telemetry & Failover Loop

1. **Telemetry Polling:** The polling engine streams metrics into `NodeTelemetryHypertable`.
2. **Anomaly Detection:** Metrics violating a `MonitoringThresholdJpaEntity` trigger an `ActiveAlarmJpaEntity`.
3. **Failover Execution:** The system automatically executes a switch to the secondary path via `RedundancyGroupJpaEntity`, logging the event.
4. **ITSM Escalation:** Sustained alarms convert into `TroubleTicketJpaEntity` for field dispatch.

### C. Proactive Crisis Simulation Workflow

1. **Scenario Definition:** The Command or Planning unit creates a `SimulationScenarioJpaEntity` to test a disaster (e.g., massive storm taking down three major nodes).
2. **Fault Injection:** The simulation engine clones the current network state from the CMDB and applies `InjectedFaultJpaEntity` triggers.
3. **Cascading Analysis:** The graph engine calculates rerouting logic, logging over-saturated backup paths into `CascadingImpactJpaEntity`.
4. **Architectural Adjustment:** Planners review the `SimulationResultJpaEntity` and upgrade bandwidth on identified bottleneck links in the real world to prevent future failure.

### D. AI-Driven Predictive Maintenance Workflow

1. **Continuous Learning:** The `AiPredictionModelJpaEntity` consumes live TimescaleDB telemetry, hardware age, and environmental factors.
2. **Insight Generation:** The model generates a `ReliabilityInsightJpaEntity` indicating that a specific node has an 85% probability of failure within 14 days due to degrading optical receiver sensitivity.
3. **Preventive Intervention:** The Maintenance Unit issues a preventive `FieldWorkOrderJpaEntity` to replace the optical module before the outage occurs, ensuring zero downtime.
