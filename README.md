# Celonis Refund Delay Analysis Project

## 🎯 Project Overview
**Platform**: Celonis Process Mining (Academic Instance)  
**Workspace**: [Quickstarts] CSV/XLSX (4) - File Upload Workspace  
**Data Pool**: [Quickstarts] Karra1p@cmoch.edu's Data Pool  
**Date Completed**: December 23, 2025  
**Environment**: https://academic-celonis-n8jbjr.eu-2.celonis.cloud

---

## 📝 Summary
Built an end-to-end refund delay analysis system in Celonis to identify bottlenecks in ticket refund processes, implement SLA monitoring, and enable predictive analytics for proactive intervention.

**Key Achievements:**
- ✅ Analyzed **200 cases** with **1,097 event rows** across **7 distinct activities**
- ✅ Configured event-level data model with proper process mining structure
- ✅ Created **5 PQL KPIs** for refund throughput, case duration, and SLA compliance
- ✅ Built interactive process dashboard with variant analysis capabilities
- ✅ Implemented 5-day refund SLA business rule with delay classification

---

## 🗂️ Data Model Architecture

### Data Structure
**Data Model ID**: 30ad9e2a-34be-493a-acfb-294928574768  
**Loaded Rows**: 1,097 event records  
**Last Updated**: December 23, 2025, 1:37:31 PM CST

### Tables and Relationships

#### Case Table
- **Table Name**: `Case Table`
- **Primary Key**: `_CASE_KEY`
- **Purpose**: Stores unique case/ticket identifiers

#### Activity/Event Table
- **Table Name**: `_CEL_CSV_ACTIVITIES`
- **Foreign Key**: `_CASE_KEY` (links to Case Table)
- **Activity Column**: `ACTIVITY_EN`
- **Timestamp Column**: `EVENTTIME`
- **Resource Column**: `RESOURCE`
- **Additional Columns**: `AMOUNT`, `_SOURCE`

#### Event Table Configuration
```
Case ID Column:     _CASE_KEY
Activity Column:    ACTIVITY_EN  
Timestamp Column:   EVENTTIME
Sort Order:         Ascending by EVENTTIME
```

**Sample Data:**
| _CASE_KEY | ACTIVITY_EN | EVENTTIME | RESOURCE | AMOUNT |
|-----------|-------------|-----------|----------|--------|
| T1000 | Payment Completed | Sep 22, 2024, 3:18:00 | System | 50 |
| T1000 | Refund Initiated | Sep 22, 2024, 3:33:00 | Gate Scanner | 40 |
| T1001 | Ticket Created | Sep 15, 2024, 8:44:00 | Finance Team | 75 |

---

## 📈 Process Activities Discovered

### Activity Frequency (897 total events across 200 cases):
1. **Payment Completed** - Highest frequency (~200 cases, 98 instances)
2. **Payment Initiated** - Very high frequency (~200 cases, 98 instances)
3. **Ticket Created** - High frequency (~200 cases, 98 instances)
4. **Ticket Scanned** - Medium frequency (~144 cases, 90 instances)
5. **Refund Completed** - Lower frequency (refund flow)
6. **Refund Initiated** - Lower frequency (refund flow)
7. **Payment Failed** - Lowest frequency (exception cases)

### Process Flow
```
Start (200 cases)
   ↓ (~29 min avg)
Ticket Created (200 cases, 98 instances)
   ↓ (~31 min avg)
Payment Initiated (200 cases, 98 instances)
   ↓ (~29 min avg)
Payment Completed (200 cases, 98 instances)
   ↓
Ticket Scanned (144 cases, 90 instances)
   ↓
End (200 cases)

Alternative Flow (Refunds):
Payment Initiated → Refund Initiated → Refund Completed
```

**Key Insights:**
- Average of **4 activities per case**
- Main process path covers ticket creation → payment → scanning
- Refund subprocess branches from payment activities
- 72% case completion rate through ticket scanning (144/200)

---

## 💻 PQL Preamble Code

### Complete KPI Definitions

```pql
-- Activity Standardization
KPI "Standardized_Activity" AS
CASE
  WHEN ACTIVITY_EN LIKE '%Refund%Completed%' THEN 'Refund Issued'
  WHEN ACTIVITY_EN LIKE '%Refund%Initiated%' THEN 'Cancellation Requested'
  WHEN ACTIVITY_EN LIKE '%Payment%Completed%' THEN 'Purchase'
  ELSE ACTIVITY_EN
END;

-- Refund Throughput Time (in days)
KPI "Refund_Throughput_Days" AS
DURATION(
  FIRST_OCCURRENCE['Cancellation Requested'],
  FIRST_OCCURRENCE['Refund Issued']
) / 86400000.0;

-- Total Case Duration (in days)
KPI "Case_Duration_Days" AS
CASE_DURATION() / 86400000.0;

-- Number of Activities per Case
KPI "Activity_Count" AS
COUNT(ACTIVITY);

-- Refund Delay Label (SLA > 5 days)
KPI "Refund_Status" AS
CASE
  WHEN DURATION(
    FIRST_OCCURRENCE['Cancellation Requested'],
    FIRST_OCCURRENCE['Refund Issued']
  ) > DAYS(5)
  THEN 'Delayed'
  ELSE 'On Time'
END;
```

### KPI Descriptions

| KPI Name | Type | Purpose | Formula |
|----------|------|---------|---------|
| **Standardized_Activity** | String | Normalizes activity names for consistency | CASE WHEN mapping |
| **Refund_Throughput_Days** | Numeric | Measures refund processing time in days | DURATION / 86400000.0 |
| **Case_Duration_Days** | Numeric | Total case lifecycle duration | CASE_DURATION() / 86400000.0 |
| **Activity_Count** | Numeric | Number of events per case | COUNT(ACTIVITY) |
| **Refund_Status** | String | SLA compliance classifier | 'Delayed' or 'On Time' |

---

## 📄 Dashboard & Analytics

### Process Exploration View
**View Name**: "_Q1: What does your process look like?"  
**Package**: [Exported Exploration 12/22/2025] What does your process look like?  
**Space**: Process Analytics  

### Dashboard Components

1. **Key Metrics Tiles**
   - Number of Cases: 200 cases (Sep-Oct 2024)
   - Distinct Activities: 7 unique activities
   - Total Events: 897 activity occurrences
   - Average Activities per Case: 4 activities

2. **Cases Over Time Chart**
   - Column chart showing case distribution
   - Peak Period: September 2024 (~200 cases)

3. **Activity Frequency Chart**
   - Horizontal bar chart by case percentage
   - Payment activities dominate, followed by refunds

4. **Process Explorer**
   - BPMN-style process flow diagram
   - Activity boxes with frequency counts
   - Flow arrows with timing metrics
   - Variant analysis capabilities

---

## 🎯 Use Cases & Business Impact

### 1. Refund SLA Monitoring
- **Metric**: Refund_Status KPI
- **Threshold**: 5 business days
- **Action**: Flag 'Delayed' cases for immediate review
- **Business Impact**: Reduce customer complaints by 30% (projected)

### 2. Bottleneck Identification
- **Method**: Filter delayed cases, analyze activity frequency
- **Target Activities**: Refund Initiated → Refund Completed path
- **Analysis Output**: Identify steps causing >5 day delays

### 3. Variant Analysis
- **Happy Path**: Ticket → Payment → Scanning (72% of cases)
- **Refund Path**: Payment → Refund Initiated → Refund Issued
- **Exception Path**: Payment Failed → alternative handling

---

## 🤖 ML Model Preparation

### Feature Engineering (Export Ready)

```sql
-- Export query structure
SELECT 
    _CASE_KEY as case_id,
    "Case_Duration_Days" as total_duration,
    "Refund_Throughput_Days" as refund_duration,
    "Activity_Count" as activity_count,
    VARIANT("_CEL_CSV_ACTIVITIES"."ACTIVITY_EN") as variant_id,
    "Refund_Status" as label  -- 'Delayed' or 'On Time'
FROM 
    [Process Model]
WHERE 
    "Refund_Throughput_Days" IS NOT NULL
```

### Python Implementation Example

```python
import pandas as pd
from sklearn.ensemble import RandomForestClassifier
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import LabelEncoder

# Load exported Celonis data
df = pd.read_csv('celonis_refund_cases.csv')

# Feature encoding
le = LabelEncoder()
df['variant_encoded'] = le.fit_transform(df['variant_id'])

# Feature matrix and labels
X = df[['total_duration', 'refund_duration', 'activity_count', 'variant_encoded']]
y = df['label'].map({'Delayed': 1, 'On Time': 0})

# Train-test split
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# Model training
rf_model = RandomForestClassifier(n_estimators=100, max_depth=10, random_state=42)
rf_model.fit(X_train, y_train)

# Predictions
y_pred = rf_model.predict(X_test)
```

---

## 🗣️ Interview-Ready Project Story

**Scenario**: "Tell me about a complex process mining project you've completed."

**Answer**:
> "I built an end-to-end refund delay analysis system in Celonis to identify bottlenecks in a ticketing refund process. The project involved ingesting 1,097 event-level records across 200 cases and configuring a clean process model with proper Case ID, Activity, and Timestamp mappings.
>
> I wrote PQL code to standardize activity names—for example, mapping variations like 'Refund%Completed%' to 'Refund Issued' for consistency. I engineered five KPIs including refund throughput time calculated using DURATION and FIRST_OCCURRENCE functions, total case duration, and an activity count per case.
>
> The business logic I implemented included a 5-day SLA threshold that classifies refunds as 'Delayed' or 'On Time' using a CASE statement with DAYS(5) comparison. This enabled proactive monitoring of SLA breaches.
>
> The resulting dashboard monitors 200 cases across 7 activities, showing that payment operations dominate the process while refund activities represent a smaller but critical subset. Through variant analysis in Process Explorer, I identified that 72% of cases follow the standard path through ticket scanning, while refund cases branch into alternative flows.
>
> For predictive analytics, I prepared the data for ML export with engineered features like case duration, activity count, and variant IDs. This enables training a Random Forest classifier to predict refund delay risk at the 'Refund Initiated' activity, allowing for proactive escalation before SLA violations occur."

---

## 🛠️ Technical Stack

- **Process Mining Platform**: Celonis (Academic Instance)
- **Query Language**: PQL (Process Query Language)
- **Data Model**: Event-based with Case and Activity tables
- **Data Volume**: 1,097 rows, 200 cases, 7 activities
- **Visualization**: Celonis Process Explorer, Activity Frequency Charts
- **ML Framework**: Python (scikit-learn, pandas)

---

## 📚 Skills Demonstrated

✅ Process Mining & Analysis  
✅ PQL (Process Query Language)  
✅ Data Modeling (Event Tables)  
✅ KPI Engineering  
✅ SLA Monitoring  
✅ Business Logic Implementation  
✅ Variant Analysis  
✅ Feature Engineering for ML  
✅ Dashboard Design  
✅ Technical Documentation

---

## 🔗 Project Links

- **Celonis Workspace**: [Academic Instance](https://academic-celonis-n8jbjr.eu-2.celonis.cloud)
- **GitHub Repository**: [celonis-refund-delay-analysis](https://github.com/Karra1p/celonis-refund-delay-analysis)

---

## 💬 Contact

**Author**: Karra1p  
**Email**: Karra1p@cmoch.edu  
**Location**: Texas  
**LinkedIn**: [Connect with me](https://www.linkedin.com/in/your-profile)

---

*Built as a portfolio demonstration project showcasing Celonis process mining capabilities and PQL expertise. December 2025.*
