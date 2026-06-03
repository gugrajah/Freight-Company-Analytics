# Role & Objective
You are an expert Analytics Engineer and Supply Chain Consultant specializing in the Microsoft BI Stack (SQL Server, T-SQL, Power BI, DAX). Your goal is to draft a comprehensive, production-grade technical project specification and mock data framework. This material will be used for a post-interview technical portfolio presentation to the hiring panel at Rhenus Air & Ocean South Africa.

# Strategic Context & Constraints
* **Corporate Scope:** Rhenus Air & Ocean focuses heavily on global freight forwarding, customs brokerage, and multi-modal transit. Ensure the projects avoid standalone retail contract warehouse fulfillment unless it directly ties to an import/export consolidation hub or cross-dock facility.
* **South African Localizations:** Incorporate real-world operational factors unique to the region, such as Port of Durban terminal congestion, OR Tambo air freight terminal processing times, and SADC cross-border choke points (e.g., Beitbridge or Komatipoort border posts).
* **Technical Benchmarks:** Solutions must account for corporate performance metrics including an On-Time Delivery (OTD) target of 95% within a 30-minute schedule buffer, primary tender acceptance rates of 85%, and dashboard load response limits under 5 seconds.

# Part 1: Project Idea Blueprints
Generate exactly 3 comprehensive project ideas broken down into the following structured sections:

### 1. Title & Target Persona
* Create a high-impact title emphasizing supply chain visibility or margin optimization.
* Identify the internal business stakeholder (e.g., Regional Air & Ocean Operations Director, Cross-Border Trade Lane Manager).

### 2. Goal & Operational Scenario
* Detail the specific business friction point (e.g., Demurrage and Detention accumulation, volumetric vs. chargeable weight margin leakage, transit time variance over line-haul corridors).
* Clearly map the scenario to the required duties: connecting complex multi-leg data, tracking supply chain trends, and identifying operational bottlenecks through data visualization.

### 3. Data Modeling & Architecture
* Define the required dimension and fact tables organized in a clean Star Schema design.
* Specify the primary keys, foreign keys, and active vs. inactive relationships (e.g., handling multiple dates like Order Date, Shipped Date, and Actual Delivery Date using `USERELATIONSHIP`).

### 4. Advanced, Performance-Optimized DAX Measures
For each project, write out complete, production-ready DAX formulas (including `VAR/RETURN` blocks to ensure high performance and readability) for the following patterns:
* **Time-Intelligence:** Comparing current-period freight metrics against historical baselines (e.g., Year-over-Year transit time deltas or rolling 3-month average demurrage fees).
* **Custom KPI Logic:** Computing complex supply chain performance indicators like exact OTD percentages factoring in the 30-minute appointment buffer or volumetric charge calculations ($Chargeable\ Weight = \max(Actual\ Weight, \frac{Volume}{6000})$).

# Part 2: T-SQL Mock Data Generation Script
Provide a single, fully-annotated, valid T-SQL script executable within the latest version of SQL Server Management Studio (SSMS). The script must programmatically build out the database structure to back these project ideas.

### Script Specifications:
1.  **DDL Structure:** * Drop tables if they exist using safe relational cascading logic.
    * Create tables with appropriate data types (`VARCHAR`, `INT`, `DECIMAL`, `DATETIME2`), primary keys, foreign key constraints, and non-clustered indexes on high-cardinality foreign keys to demonstrate performance-optimization awareness.
2.  **DML Logic (Data Population):**
    * Use common table expressions (CTEs) or loops to generate a minimum of 12 months of transactional data spanning back from today's date.
    * **Built-in Localized Anomalies:** Programmatically inject systemic delays into the data:
        * Longer dwell-times (`DATETIME` differences) for ocean containers hitting the Port of Durban.
        * Increased transit time variances for cross-border road freight crossing the Beitbridge border post into the SADC region compared to standard domestic legs.
3.  **Data Quality:** Ensure the generated financial numbers, weights, volumes, and timeframes are mathematically logical (e.g., Actual Delivery Time should generally succeed Actual Pick Up Time; volumetric metrics should match real-world weight ratios).