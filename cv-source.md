# TAYLOR NICHOLS

**Email:** tenichols94@gmail.com
**Location:** Raleigh, NC (open to relocation or remote)
**LinkedIn:** https://www.linkedin.com/in/taylor-nichols-ab1976135/
**Quant Research Portfolio:** https://tenicho.github.io/quant-portfolio/
**GitHub:** https://github.com/tenicho

---

## WORK EXPERIENCE

### Novanta, Apex, NC

*Acquired ATI Industrial Automation on August 31, 2021*
**August 2019 - Present**

#### Project Manager | October 2022 - 2023

- Led projects across engineering, sales, manufacturing, quality, supply chain, and product management
- Defined project scope, objectives, requirements, and specifications in partnership with Product Managers
- Developed and managed project schedules across four concurrent product development initiatives
- Completed a new product line release that introduced 6 different product variants

#### Product Owner | January 2021 - 2022

- Led work execution in a group of 8 engineers across electrical, controls, and mechanical disciplines
- Transitioned the Design Verification group to Scrum methodology
- Prioritized backlog to deliver high-value features to stakeholders quickly
- Collaborated with stakeholders to establish priorities and development roadmaps

#### Design Verification Engineer | August 2019 - Present

- Serve as lab test engineer for three product lines: tool changers, material removal, and force/torque sensors. Products are deployed in industrial automation, logistics automation, and medical applications
- Design the test approach for new product introduction (NPI) projects, configure the equipment, run the tests, analyze the data, and document and publish the findings
- Make the pass/fail determination against specification, which gates product release
- Developed specification standards across all product lines, setting the example for how product specifications are written
- Performed CFD simulations to support thermal and fluid projects, spanning three areas:
  - Steady-state energy loss and pressure drop simulations
  - Dynamic turbine analysis
  - Combined flow performance and thermal / heat exchanger analysis at the system level

**Environmental & reliability testing**

- Conduct ingress protection (IP) testing
- Thermal and humidity testing; thermal fatigue
- Salt water fog testing
- Vibration testing, including HALT and IEC 60068-2-6 sinusoidal vibration

**Mechanical testing**

- Mechanical overload testing
- Complex loading fatigue testing
- Designed custom test setups for accelerated life testing and overload conditions, covering load cases from 1 N to 5000 N

**Electrical testing**

- High current and high voltage testing
- Hipot (dielectric withstand) testing
- Thermal testing under high current conditions
- Electrical motor testing, covering phase alignment, phase resistance, phase inductance, and Ke (back-EMF constant) calculations
- Oscilloscope measurement and waveform analysis
- Read and work from electrical schematics

**Firmware & software testing**

- Firmware testing and debugging for products
- Communication protocols: serial, Modbus, EtherCAT, Ethernet
- Software testing and debugging

**Instrumentation & test systems built**

- Built and configured data acquisition (DAQ) systems using IEPE accelerometer channels, four-wire RTD and thermocouple setups, fast analog voltage differentials, and four-wire high-resolution resistance measurement
- Built a custom leak-down test setup in LabVIEW
- Led the development of the wet lab (fluid testing room), covering electrical, programming, and mechanical work, running on a B&R PLC:
  - Closed-loop pressure drop test system using a Coriolis flow meter and pressure transducers
  - Life cycle water test station with overflow safety cutoffs
- Built a separate air mass flow system measuring mass flow rate of air via thermal flow meter
- Designed test fixtures and setups

### Schneider Electric, Columbia, SC

**Industrial/Manufacturing Engineer - Intern | May 2016 - August 2016**

- Designed an ergonomic electrical wire storage rack with a replenishing mechanism utilizing Lean principles
- Collaborated with cross-functional team on a plant-wide re-layout and participated in two 5-day kaizen events
- Implemented a two-bin material replenishing system on an assembly line

---

### Spartanburg Steel Products, Spartanburg, SC

**Manufacturing Engineer - Intern | Winter 2015**

- Designed a new workstation to improve production efficiency
- Developed work instructions for a new process
- Tested various supplies and equipment to determine the most cost-efficient solution for a new process

---

### Superior Auto Body, Boiling Springs, SC

**Technician | October 2010 - December 2015**

- Diagnosed a wide range of mechanical and structural automotive issues, working alongside senior technicians
- Disassembled and reassembled vehicle body panels, HVAC systems, and interior components
- Completed prep work prior to final body repair and refinishing

---

## QUANTITATIVE RESEARCH PROJECTS

*Write-ups and full results: https://tenicho.github.io/quant-portfolio/*

### ML Trend & Momentum Cross-Sectional Equity Model

*Python | Random forest ranking model | US equities, 1998-2026*
Report: https://tenicho.github.io/quant-portfolio/strategies/ml-trend-momentum/
Code: https://github.com/tenicho/ml-trend-momentum

A machine-learning model that ranks the liquid US equity universe by predicted 40-day forward return and trades the top 1%. Built to answer one question: does basic, widely available price data still contain tradable signal, and can it be captured on a schedule a retail trader could actually run?

**Design constraints**

- Executable by anyone. Orders generated after the close, filled at the next open, exited at the close. One trading day a month, no intraday decisions, no live quotes, no special infrastructure.
- Readily accessible data only. 36 features derived from daily equity OHLCV, plus VIX and two S&P-derived series. No fundamentals, analyst data, or alternative data.
- Long horizon by design. A 40-day forecast matched to a ~2-month hold, chosen to sit outside the horizons where high-frequency firms compete and to let short-term noise filter out.
- Liquid universe. US equities at ADV ≥ $25M and price ≥ $5, roughly 1,070 names per day.

**Method**

- Random forest over a 23.6M-row point-in-time panel spanning 1998-2026, including all delisted tickers (70% of the 19,023 listings in the panel), eliminating survivorship bias
- Six-fold expanding walk-forward across 2006-2022; each fold refit from scratch with a 63-session embargo between training and test to prevent label leakage across the boundary
- Transaction costs calibrated from the data and charged in full on entry
- 2023-2026 held back entirely as out-of-sample test

**Results (out of sample)**

- Rank-IC of +0.044 (t = 2.30 after correcting for overlapping labels), against +0.049 in development
- 47 independent samples out of sample against 228 in development
- Traded book returned +37.4%/yr against SPY's +22.6%. $1 grew to $3.02 versus the index's $2.03
- Took real risk to get there: 27.4% volatility against the market's 15.2%, and a -26.5% drawdown against -18.8%
- Risk-adjusted, a three-way tie: excess Sharpe of 1.17 for the model, 1.18 for the index, 1.12 for the MTUM momentum ETF
- Separates decisively from naive momentum: sorting on 12-month momentum alone, with identical universe, costs, and schedule, returned +0.25%/yr against the model's +15.9% in development
- Month-by-month IC shows where the model struggles: sharp drawdowns and stretches when market-wide momentum breaks down. The most negative months line up with the March 2023 regional banking stress and the April 2025 tariff selloff.

**Conclusion:** the model generates real signal from a minimal data set, beating the index on absolute return in both development and holdout. That return comes at a cost. Concentration and higher volatility mean deeper drawdowns, and on a risk-adjusted basis the strategy lands roughly level with the index rather than ahead of it.

---

### Semantic Equity Classification

*Python | Transformer embeddings and unsupervised clustering | 6,006 US-listed companies*
Report: https://tenicho.github.io/quant-portfolio/research/semantic-equity-classification/
Code: https://github.com/tenicho/semantic-equity-classification

A semantic search and classification layer over US-listed companies, built so thematic screens surface peers by what a business actually does instead of how it happens to be labeled.

- Built the retrieval layer of a RAG pipeline over 6,006 US-listed companies, embedding business descriptions into 768-dimension vectors with a transformer model (BGE-base-en-v1.5) to enable natural-language thematic search
- Enabled theme-based company search that returns relevant names regardless of assigned sector or industry, surfacing peers that conventional screens miss
- Compressed embeddings to 3 interpretable dimensions via UMAP, producing compact per-company features small enough to use directly as model inputs, against 16 to 50 for a comparable PCA factor set
- Generated 40 data-driven company clusters that group by actual business activity instead of assigned label, with individual clusters spanning multiple sectors

---

### Rules-Based Momentum (Modified 12-1)

*Python | Rules-based 12-1 momentum, monthly rebalance | S&P 500*
Report: https://tenicho.github.io/quant-portfolio/strategies/rules-based-momentum/
Code: https://github.com/tenicho/rules-based-momentum

A simple, mechanical momentum strategy designed to be run manually on a monthly schedule, and tested hard enough to say where it doesn't hold up.

**What it does**

- Screens S&P 500 members down to the 20 most liquid by dollar volume
- Ranks those on 12-1 momentum, meaning twelve-month return with the most recent month skipped to avoid short-term reversal
- Holds the top 5 from that ranking, equal weight, rebalanced every 20 trading days
- Long only, no leverage, no discretion

**How it was tested**

- 10-year development window (Aug 2013 to Jul 2023), where all configuration choices were made
- 3-year holdout (Aug 2023 to Jul 2026), run once after the configuration was locked
- Survivorship-clean panel with point-in-time index membership; delisted names kept and sold at last traded price
- Modeled round-trip transaction costs charged on entry
- 10,000-path Monte Carlo for risk of ruin and drawdown distribution

**Results**

|                      | Strategy | SPY    | MTUM   |
| -------------------- | -------- | ------ | ------ |
| Development CAGR     | 20.3%    | 12.4%  | 11.8%  |
| Holdout total return | 4.43x    | 1.70x  | 2.09x  |
| Holdout Sharpe       | 1.22     | 0.99   | 1.04   |
| Holdout max drawdown | -37.6%   | -18.8% | -21.0% |

**Conclusion:** beat the index on both sides of the split, but falls about twice as hard. Period-by-period testing also shows the configuration ranking doesn't persist year to year. A concentrated equity exposure that can outperform the market, but the inconsistencies would make it an uncomfortable strategy to hold.

---

### US Equity Data Pipeline (1998-2026)

*Python | Data pipeline and validation | 41.8M bars, 1998-2026*
Report: https://tenicho.github.io/quant-portfolio/research/data-cleaning/
Code: https://github.com/tenicho/data-cleaning

A survivorship-clean daily price dataset for the entire US stock market going back to 1998. The pipeline has built-in checks to confirm the data is accurate, and produces a clean dataset that drops directly into different models without re-doing the cleaning each time.

- **Scope:** daily OHLCV for the full US listed universe: 41.8M bars across 19,023 listings, 1998-01-02 to 2026-07-31, plus point-in-time S&P 500 membership back to 1998
- **Survivorship:** 13,288 of 19,023 listings (70%) stop trading before the dataset ends, spread across every year instead of bunched into recent ones. A universe of survivors only is the single most common way a backtest flatters itself.
- **Three price-adjustment states in every row:** raw, split-adjusted, and fully adjusted, so returns and price levels each get the correct series instead of one compromise between them
- **Macro layer:** credit spreads, VIX term structure, and financial-stress indices from FRED and CBOE, plus market breadth and volume-weighted price deviation derived from the dataset itself
- **Verification against known outcomes:** the point-in-time roster returns Lehman Brothers at its real $3.65 close the Friday before it filed, with eight more failed companies verified the same way; split ratios reconciled against independent corporate actions
- **Documented limitations:** OHLCV only: no fundamentals, no shares outstanding, no bid/ask

---

### Round-Trip Trading Cost Estimator

*Python | Spread estimation from daily bars | Full US equity universe*
Report: https://tenicho.github.io/quant-portfolio/research/transaction-costs/
Code: https://github.com/tenicho/transaction-costs

Backtests need a realistic round-trip trading cost, but the underlying dataset is daily OHLCV with no bid/ask quotes. This project builds a cost estimate from the daily data alone.

- Tested the standard academic estimator (Corwin-Schultz) and **rejected** it, because it responds to volatility rather than liquidity and returns roughly the same cost for a megacap as for a microcap
- Adopted the EDGE estimator only where it demonstrably works, below $10M ADV, calibrated in aggregate across millions of observations instead of any single reading
- Established by simulation that liquid names sit permanently out of reach: the data needed to recover a spread grows with the fourth power of the volatility-to-spread ratio
- Bridged the untrustworthy middle by interpolating between the last measured anchor and observable quoted spreads at the liquid end, where real costs are well known and don't need estimating
- Layered on a bounded volatility multiplier, a one-tick floor, and an effective-spread haircut, so cost responds to how jumpy a stock is without letting volatility hijack the estimate

**Result:** a reusable cost function that assigns a defensible round-trip cost to any stock on any day, ready to drop into a backtest. Each component is either measured, observed, or explicitly flagged as interpolated.

---

### NPI Project Vetting Tool, Novanta

*Financial modeling | 2021*

Identified that the company had no standardized process for vetting new product introduction (NPI) projects, and built one on my own initiative.

- Created an NPI project vetting form incorporating multiple financial analysis tools:
  - Gross margin analysis
  - Project-level return on investment (ROI)
  - Discounted cash flow (DCF) calculator
  - Sensitivity analysis, with project development time, gross margin, and short- and long-term sales projections as input variables
- Standardized the NPI project vetting process company-wide; the tool was adopted and implemented
- Awarded performance stock units (PSUs) following implementation

---

## MOBILE APPLICATIONS

### Stock Screener: Trade Ideas, *Co-Developer*

*iOS (native Swift) | Python backend | AWS | Launched December 2024*
[App Store](https://apps.apple.com/us/app/stock-screener-trade-ideas/id6738607499) | 29 five-star reviews

A stock research platform combining a custom in-house screener, AI-assisted stock and market analysis, and a reference wiki for US equities. Co-developed across the full stack, working on AI infrastructure, the iOS front end, and the Python backend rather than owning a single layer.

**Features built:**

- **Theia Picks:** a bootstrapped smart search router that turns a natural-language prompt into a custom stock screener, a stock list, or both, along with the reasoning behind them. The router decides which tools to use for each request, drawing on one or more of three: the in-house screener, web search, and the model's own knowledge.
- **Pulse:** a condensed market overview. Aggregates macro data from multiple providers (CPI, PPI, commodity indexes, housing indexes) alongside major index movement (S&P, NASDAQ, Dow), then feeds it into an AI model to produce a short, readable summary of the market, including movements that commonly go overlooked.
- **AI Wiki:** generates stock analysis by combining processed stock data (fundamentals, technical indicators, growth metrics, analyst ratings) with recent relevant news events, curated into a single easy-to-consume format.
- **Stock Screener:** custom screener infrastructure allowing users to build filters across a wide set of data points.
- **Stock Wiki:** reference data for publicly traded U.S. equities: fundamentals, technical indicators, analyst ratings, growth metrics, and news events.
- **Watchlist:** user-created watchlists.

**Data and retrieval work:**

- Built the ingestion and post-processing layer for third-party market data: trailing twelve-month (TTM) calculations, quarter-over-quarter growth tracking, and extensive data cleaning
- Bootstrapped a retrieval-augmented generation (RAG) system housed in S3, where post-processed data is embedded and stored, then retrieved as a data source for the AI features

**Model evaluation**

- Built a repeatable process for evaluating the LLM-assisted features, scoring outputs 0 to 5 on fidelity, insight, prioritization, and usability
- Curated a prompt set for each feature and task, ran 100+ prompts per candidate model, and used a higher-tier model as the judge against the rubric
- Compared candidate models across providers on score, latency, and cost, including Claude Haiku and Sonnet, DeepSeek, and AWS Nova Pro and Nova Lite
- Used the results to choose which model runs each task instead of defaulting to one model everywhere

**Stack:** Native Swift (iOS) with push notifications; Python backend; AWS Amplify, RDS (MySQL), Bedrock (AI features), Lambda, S3, EventBridge, SNS, CloudWatch, IAM roles.

---

### Stock Chart Trading Game, *Solo Developer*

*iOS (native) | Python | Launched February 1, 2026*
[App Store](https://apps.apple.com/us/app/stock-chart-trading-game/id6758588241) | 4.5 average across 6 ratings

A game that tests chart-reading skills against real historical market data. Users predict whether each chart is bullish or bearish; because every chart comes from an actual past event, players get genuine feedback on how those setups would have played out.

- Sourced real historical stock data and generated the charts in Python
- Front-end only, with no backend, no account required, and no internet connection needed after download
- Built over a few weekends; currently ships with a single timeframe, with additional features planned

---

### Fishing Weight Scale Estimate, *Solo Developer*

*iOS (native) | Launched February 25, 2025*
[App Store](https://apps.apple.com/us/app/fishing-weight-scale-estimate/id6742200055) | 7 five-star reviews

A fish weight estimator built to solve a personal need: getting a weight estimate without an internet connection. Fully offline.

---

## INDEPENDENT WRITING

### Capital Curiosity, *Founder & Author* | March 2025 to Present

*Substack | 379 subscribers*
https://substack.com/@tenichols94

A publication making part of my own stock research process publicly available. I actively manage my own portfolio, and I write up the analysis I do on companies I'm looking into, as much for my own justification that I understand a business as for readers.

- Publish original equity research write-ups drawn from my live analysis process
- Cover notable developments in current holdings and in other companies worth writing about
- Grew to 379 subscribers since first post on March 19, 2025

**Selected write-ups (highest readership):**

- Crocs (CROX): Undervalued, Misunderstood, and Ready for a Breakouthttps://tenichols94.substack.com/p/crocs-crox-undervalued-misunderstood
- Comstock Holding Companies, Inc. (CHCI): An Asset-Light Cash Machine with a Governance Twisthttps://tenichols94.substack.com/p/comstock-holding-companies-inc-chci
- Adobe (ADBE): When the Market Panics, Do the Financials Agree?
  https://tenichols94.substack.com/p/adobe-adbe-when-the-market-panics

---

## RESEARCH & TEACHING

### Master's Thesis Research | 2017 - 2019

*Clemson University*

- Designed and built the experimental apparatus to create polygonal hydraulic jumps
- Developed image processing techniques in MATLAB, including a custom calibration method, to calculate the geometric characteristics of each jump
- Identified hysteresis between hydraulic jump mode states
- Showed the data collapses on a characteristic ratio, area / (perimeter × height), against Reynolds number across all mode numbers and experimental methods, and on height / perimeter against Weber number
- Together these collapses demonstrated the relative importance of inertial, viscous, and surface tension forces in governing this multi-modal phenomenon
- Work published in *Physical Review Fluids* (2020, 2023)
- Presented at the Clemson University Mechanical Engineering research conference

### Mechanical Engineering Lab Teaching Assistant | 2017 - 2019

*Clemson University*

- Developed lesson plans and presented material to classes of 10-15 students
- Taught experimental concepts through live demonstration
- Graded coursework and gave students constructive feedback

### Fluid and Thermal Lab Lead | 2016

*Clemson University*

- Led a team of high school and undergraduate students in fluid experiments
- Reviewed safety protocols and ensured proper adherence
- Mentored team members to guide experiment setups and analysis

### Undergraduate Tutor | 2014 - 2017

*Clemson University*

- Provided in-class teaching assistance for up to 50 students in general engineering, MATLAB, and SolidWorks courses
- Held weekly tutoring sessions outside of class for each course
- Provided personalized 1-on-1 tutoring to students in math and programming

---

## EDUCATION

**Clemson University**, Clemson, SC
Master of Science in Mechanical Engineering | August 2019
Thesis advisor: Dr. J.B. Bostwick

**Clemson University**, Clemson, SC
Bachelor of Science in Mechanical Engineering | December 2017

**Greenville Technical College**, Greenville, SC
Diploma in Auto Body Repair | July 2013

**Relevant Coursework**

- *Mathematics & Computation:* Differential and Integral Calculus, Differential Equations, Advanced Engineering Statistics, Numerical Methods, Linear Algebra, Computational Fluid Dynamics
- *Thermal & Fluid Sciences:* Fluid Mechanics, Thermodynamics, Transient Thermodynamics, Turbulent Flow, Heat Transfer, Advanced Convection Heat Transfer, Gas Turbines
- *Other Engineering:* Controls, Materials Science, Mechanics of Materials, Manufacturing, Aeronautical Design, Electrical Engineering, Chemistry, Technical Writing

---

## SKILLS AND COMPETENCIES

**Programming Languages**

- Proficient in Python, MATLAB, Swift
- Experience in R, C++, VBA, and Fortran
- SQL / MySQL

**Python Libraries**

- Data and numerical: pandas, NumPy, SciPy, PyArrow
- Statistics: statsmodels
- Machine learning: scikit-learn, XGBoost, LightGBM, CatBoost
- Deep learning: PyTorch, TensorFlow, Keras
- NLP and embeddings: Sentence-Transformers
- Dimensionality reduction and clustering: UMAP, PCA, K-Means
- Visualization: Matplotlib, Seaborn
- Data access: requests, urllib, MySQL Connector

**Cloud & Backend (AWS)**

- AWS Amplify, Bedrock, Lambda, RDS (MySQL), S3, EventBridge, Simple Notification Service (SNS), CloudWatch, IAM roles
- Python backend development
- Database design and management

**Quantitative Research & Machine Learning**

- Random forest and cross-sectional return modeling
- Walk-forward validation, expanding-window backtesting, embargo periods to prevent label leakage
- Out-of-sample holdout design; development / holdout separation
- Monte Carlo simulation for risk of ruin and drawdown distribution
- Feature engineering from price and volume data
- Rank information coefficient (rank-IC), Sharpe ratio, drawdown and volatility analysis
- Survivorship-bias-free panel construction; point-in-time index membership
- Semantic search over embedded text; unsupervised clustering for company classification
- Market microstructure: spread estimation (EDGE, Corwin-Schultz), transaction cost modeling
- Momentum and factor strategy construction

**AI Application Development**

- AWS Bedrock
- Agentic system design: tool selection and resource routing by the model
- LLM application development: prompt-driven tool use, natural-language-to-query systems
- LLM evaluation: rubric design, model-as-judge scoring, cost and latency benchmarking across providers
- Retrieval-augmented generation (RAG): embedding and retrieval over processed datasets
- Combining structured data with generative models to produce analysis

**Data Engineering**

- Third-party data ingestion and post-processing pipelines
- Large-panel construction and validation (41.8M-bar daily equity panel)
- Corporate action handling: splits, dividends, price adjustment states
- Data provenance and reconciliation against independent sources
- Trailing twelve-month (TTM) and quarter-over-quarter growth calculations
- Data cleaning and normalization at scale

**Mobile Development**

- Native iOS development in Swift
- Push notifications
- Shipped applications on the Apple App Store (solo and co-developed)

**Test Engineering**

- Design Verification Test (DVT) and Engineering Development Test (EDT)
- Specification creation and test procedure development
- Test fixture and test system design
- Environmental and reliability testing: ingress protection (IP), thermal, humidity, thermal fatigue, salt water fog, vibration, HALT, IEC 60068-2-6
- Mechanical testing: overload, complex loading fatigue, custom accelerated life setups (1 N to 5000 N)
- Electrical testing: high current, high voltage, hipot, oscilloscope measurement, schematic interpretation
- Electrical motor testing: phase alignment, phase resistance, phase inductance, Ke calculations
- Firmware and software testing and debugging

**Instrumentation & Automation**

- Data acquisition (DAQ) systems:
  - Integrated Electronic Piezoelectric (IEPE) for accelerometers
  - Four-wire Resistive Temperature Detector (RTD) and thermocouple setups
  - Fast analog voltage differentials
  - Four-wire high-resolution resistance
- LabVIEW; B&R PLC programming
- Instrumentation: Coriolis flow meters, thermal mass flow meters, pressure transducers
- Communication protocols: serial, Modbus, EtherCAT, Ethernet
- Programming microcontrollers and microprocessors

**Simulation & Design**

- CFD simulation: steady-state energy loss / pressure drop, dynamic turbine analysis, combined flow and thermal system analysis
- SolidWorks
- Designed and analyzed electrical circuits, communication systems, motors, and a Bluetooth speaker

**Mechanical & Manufacturing**

- Deep knowledge of mechanical systems (ex: repaired motors, transmissions, and differentials)
- Conducted Human-Robot Collaborative Manufacturing experiments using the Baxter Humanoid Robot
- Lean principles, kaizen, two-bin replenishment systems

**Financial Modeling & Analysis**

- Discounted cash flow (DCF) modeling
- Sensitivity analysis
- Gross margin analysis; return on investment (ROI)
- Equity research and write-ups; active personal portfolio management
- Fundamental analysis, technical indicators, growth metrics, analyst ratings
- Macroeconomic data (CPI, PPI, commodity and housing indexes, major equity indexes)

**Project & Product Management**

- Scrum methodology
- Backlog prioritization
- Roadmap and schedule development
- Stakeholder management

---

## PEER-REVIEWED PUBLICATIONS

1. S. Tamim, **T. Nichols**, J. Lundbek Hansen, T. Bohr, and J.B. Bostwick, "Corner universality in polygonal hydraulic jumps," *Physical Review Fluids*, 8 (3), L032001 (2023)
   https://journals.aps.org/prfluids/abstract/10.1103/PhysRevFluids.8.L032001
2. **T. Nichols** and J.B. Bostwick, "Geometry of polygonal hydraulic jumps and the role of hysteresis," *Physical Review Fluids*, 5 (4), 044005 (2020)
   https://journals.aps.org/prfluids/abstract/10.1103/PhysRevFluids.5.044005

---

## THESIS

**T. Nichols**, "Geometric Characterization of Polygonal Hydraulic Jumps and the Role of Weir Geometry" (2019). All Theses. https://tigerprints.clemson.edu/all_theses/3194

---

## CERTIFICATIONS

- Finance for Non-Finance Professionals, Rice University | February 2022
- Certified Scrum Product Owner, Scrum Alliance | January 2021
- SolidWorks CSWA Certified Associate | Spring 2015
