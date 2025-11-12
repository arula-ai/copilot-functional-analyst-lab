# RIFCC Prompt Library for Copilot Analyst Lab

This document contains ready-to-use prompts following the RIFCC framework (Role, Inputs, Format, Constraints, Checks) for all lab exercises.

---

## Lab 1 – Legacy Discovery & Assessment

### 1.1 Understanding Business Logic (fees_calc.cob)

**Role:** Act as a business analyst specializing in legacy system modernization.

**Inputs:** Analyze #fees_calc.cob to extract business rules and logic.

**Format:** Provide a business requirements narrative with:
- Executive summary of the program's purpose
- Numbered list of business rules extracted from the code
- Decision table showing fee calculations
- Examples with sample inputs and outputs

**Constraints:**
- Document all hard-coded values (rates, thresholds, dates)
- Identify any missing error handling paths
- Note implicit assumptions about data quality
- Exclude technical implementation details (focus on "what", not "how")

**Checks:**
- List any assumptions you made about unclear logic
- Flag edge cases not explicitly handled in the code
- Identify any business rules that conflict or overlap

---

### 1.2 Creating Data Mappings (copybook to SQL)

**Role:** Act as a data architect performing legacy-to-modern field mapping.

**Inputs:** Map fields from #copybook.cpy to target tables in #schema.sql.

**Format:** Create a markdown table with columns:
- Source Field (COBOL name)
- Source Type (PIC/COMP-3 definition)
- Target Table.Column
- Target SQL Type
- Transformation Logic
- Validation Rules

**Constraints:**
- Convert all COMP-3 types to appropriate SQL DECIMAL types
- Preserve precision for monetary values
- Document any truncation or rounding rules
- Flag fields present in copybook but missing in schema

**Checks:**
- Verify that decimal precision is not lost in conversion
- List any unmapped source fields and explain why
- Note potential data quality issues (nulls, invalid values)

---

### 1.3 Risk Assessment (fees_calc.cob + job_stream.jcl)

**Role:** Act as a modernization risk analyst evaluating migration complexity.

**Inputs:** Review #fees_calc.cob and #job_stream.jcl for modernization risks.

**Format:** Output a risk register table with columns:
- Risk ID
- Category (Logic/Data/Performance/Testing)
- Description
- Likelihood (High/Medium/Low)
- Impact (High/Medium/Low)
- Mitigation Strategy

**Constraints:**
- Identify at least five distinct risks
- Cover multiple categories (not all in one category)
- Focus on business continuity and accuracy risks
- Avoid generic risks; be specific to this codebase

**Checks:**
- Ensure each risk has a concrete mitigation plan
- Flag any risks that block go-live
- List assumptions about current system performance

---

### 1.4 Batch Flow Documentation (job_stream.jcl + fees_calc.cob)

**Role:** Act as a process analyst documenting batch workflows.

**Inputs:** Trace the end-to-end flow in #job_stream.jcl and processing steps in #fees_calc.cob.

**Format:** Provide:
1. Step-by-step narrative of the batch process
2. Mermaid flowchart syntax showing:
   - All job steps in sequence
   - Decision points (conditionals)
   - Input/output datasets
   - Error paths

**Constraints:**
- Map JCL job steps to COBOL program logic
- Show dependencies between steps
- Document error handling and retry logic
- Include timing/scheduling requirements if mentioned

**Checks:**
- Verify the flow handles all error scenarios
- List any parallel processing opportunities
- Note missing monitoring or logging points

---

### 1.5 SQL Translation (fees_calc.cob to SQL)

**Role:** Act as a SQL developer translating COBOL business logic to ANSI SQL.

**Inputs:** Convert the fee calculation logic in #fees_calc.cob using tables defined in #schema.sql.

**Format:** Provide:
- Complete SQL query (SELECT statement or stored procedure)
- Inline comments explaining each tier/discount/condition
- Test data examples showing input and expected output

**Constraints:**
- Use only tables and columns defined in #schema.sql
- Preserve exact calculation logic (no optimization yet)
- Handle all tier logic and discount rules
- Include error conditions as CASE statements or WHERE filters
- Use ANSI SQL (no vendor-specific syntax)

**Checks:**
- Verify query matches all business rules from the COBOL
- List any COBOL logic that cannot be directly translated
- Flag assumptions about null handling or data types

---

## Lab 2 – Automating Data Tasks

### 2.1 CSV Data Profiling

**Role:** Act as a data quality analyst profiling CSV datasets.

**Inputs:** Analyze #transactions.csv and #customers.csv against the schema in #schema.sql.

**Format:** For each file, provide a markdown table with:
- Total row count
- Column names and inferred types
- Null counts per column
- Duplicate record count
- Value range/distribution for key fields
- Data quality issues detected

**Constraints:**
- Do not execute any database queries (work with CSV content only)
- Compare CSV structure to #schema.sql and note mismatches
- Identify primary/foreign key candidates
- Flag any referential integrity violations

**Checks:**
- List assumptions about data types
- Note columns that may need cleansing before load
- Identify any suspicious patterns (outliers, impossible values)

---

### 2.2 Excel-to-SQL: Monthly Revenue Trend

**Role:** Act as a reporting analyst converting Excel formulas to SQL.

**Inputs:**
- Reference #transactions.csv structure and #schema.sql
- Excel formula: `=SUMIFS($B:$B, $A:$A, ">=​" & EOMONTH(TODAY(),-1)+1, $A:$A, "<=​" & EOMONTH(TODAY(),0), $C:$C, "Completed")`

**Format:** Provide:
- ANSI SQL query producing the same result
- Explanation of how Excel logic maps to SQL
- Sample output format

**Constraints:**
- Use DATE functions equivalent to EOMONTH and TODAY
- Filter only "Completed" status transactions
- Sum amounts for the current month (last month-end + 1 to current month-end)
- No vendor-specific functions (pure ANSI SQL)

**Checks:**
- Verify date range logic matches Excel exactly
- List any edge cases (partial months, timezone handling)
- Note assumptions about the "amount" and "date" column names

---

### 2.3 Excel-to-SQL: Customer 80/20 Analysis

**Role:** Act as an analytics engineer translating Excel array formulas to SQL.

**Inputs:**
- Reference #transactions.csv, #customers.csv, and #schema.sql
- Excel formula: `=LET(sorted, SORT(by_total, -1), running, SCAN(0, sorted, LAMBDA(a,b, a+b)), running/total)`

**Format:** Provide:
- SQL query with window functions to calculate cumulative revenue percentage
- Customer ranking by total revenue
- Identification of the top 20% of customers by revenue contribution

**Constraints:**
- Use window functions (SUM() OVER, ROW_NUMBER, etc.)
- Sort customers by descending total revenue
- Calculate running total as percentage of grand total
- Output customer ID, total revenue, and cumulative percentage

**Checks:**
- Confirm that percentages sum to 100%
- List any assumptions about how to aggregate transactions per customer
- Note handling of customers with zero or negative revenue

---

### 2.4 Excel-to-SQL: Failed Transaction Alerts

**Role:** Act as an operations analyst building alert queries.

**Inputs:**
- Reference #transactions.csv and #schema.sql
- Excel formula: `=COUNTIFS(Status,"Failed",Amount,">=500",Date,">="&TODAY()-7)`

**Format:** Provide:
- SQL query counting failed transactions
- Filter criteria clearly commented
- Example output showing count and sample transactions

**Constraints:**
- Filter for Status = "Failed"
- Filter for Amount >= 500
- Filter for Date within last 7 days from today
- Return count plus optional detail rows for investigation

**Checks:**
- Verify date arithmetic matches Excel's TODAY()-7 logic
- List assumptions about date column format
- Note how to parameterize this query for daily execution

---

## Lab 3 – Governance & Accuracy

### 3.1 SQL Error Detection and Correction

**Role:** Act as a senior database developer conducting code review.

**Inputs:** Review #flawed_sql_example.sql for logical errors, syntax issues, and best practice violations.

**Format:** Provide:
- Line-by-line error catalog with severity (Critical/Warning/Info)
- Corrected SQL query with explanatory comments
- Explanation of what each error would cause in production

**Constraints:**
- Identify logic errors (wrong joins, missing filters, incorrect aggregations)
- Catch syntax errors or non-standard SQL
- Flag performance anti-patterns
- Note any security issues (SQL injection risks, missing access controls)

**Checks:**
- Verify the corrected query produces expected results
- List any assumptions made during correction
- Suggest testing steps to validate the fix

---

### 3.2 Analysis Document Review

**Role:** Act as a quality assurance analyst reviewing business requirements documentation.

**Inputs:** Audit #flawed_analysis.md for accuracy, completeness, and logical consistency.

**Format:** Provide:
- Categorized list of issues:
  - Factual errors (incorrect statements)
  - Missing information (gaps in coverage)
  - Logical inconsistencies (contradictions)
  - Unsupported assumptions
- Corrected version of the document with tracked changes explained

**Constraints:**
- Cross-reference claims against source files (COBOL, schema, CSVs)
- Flag any requirements that are ambiguous or untestable
- Identify missing edge cases or error handling
- Note areas where validation is required

**Checks:**
- Ensure all corrections are traceable to source evidence
- List any areas where additional research is needed
- Verify that corrected document is internally consistent

---

### 3.3 Python Join Logic Verification

**Role:** Act as a data engineer reviewing ETL transformation code.

**Inputs:** Analyze #flawed_join_logic.py for correctness and efficiency.

**Format:** Provide:
- Bug report listing each defect with:
  - Line number
  - Issue description
  - Expected vs. actual behavior
  - Severity
- Corrected Python code with comments
- Test cases demonstrating the bug and the fix

**Constraints:**
- Focus on join logic (inner/outer/cross, key matching)
- Check for off-by-one errors, null handling, duplicate rows
- Verify datatype compatibility
- Review performance implications

**Checks:**
- Confirm corrected logic produces expected row counts
- List assumptions about input data structure
- Suggest unit tests to prevent regression

---

## General Multi-Purpose Prompts

### Code Explanation Request

**Role:** Act as a technical documentation specialist.

**Inputs:** Explain the purpose and logic of #[filename].

**Format:** Provide a plain-English summary including:
- Purpose statement (one sentence)
- Key functionality (bullet list)
- Important variables/fields
- Dependencies and assumptions

**Constraints:**
- Avoid jargon; write for non-technical stakeholders
- Focus on business impact, not implementation details
- Highlight any hard-coded values or configuration

**Checks:**
- Verify explanation matches actual code behavior
- List any unclear or ambiguous sections requiring clarification

---

### Validation Plan Generation

**Role:** Act as a QA lead designing test scenarios.

**Inputs:** Based on #[filename], create a validation plan for modernization testing.

**Format:** Provide a test plan with:
- Test case ID, description, inputs, expected outputs
- Coverage of happy path, edge cases, and error conditions
- Data setup requirements

**Constraints:**
- Ensure at least 80% path coverage
- Include boundary value tests
- Test both functional correctness and performance
- Document any test data dependencies

**Checks:**
- Verify test cases are executable without missing data
- List any assumptions about test environment
- Flag high-risk scenarios requiring manual verification

---

## Usage Tips

1. **Copy the full prompt** including all five sections (Role, Inputs, Format, Constraints, Checks)
2. **Replace #[filename]** placeholders with actual file references using `#` in Copilot Chat
3. **Customize constraints** based on your specific project requirements
4. **Review outputs** against the source files yourself—AI makes mistakes
5. **Document assumptions** made by Copilot in your deliverables

---

## RIFCC Framework Quick Reference

- **Role:** "Act as a [specific expertise]..."
- **Inputs:** "Analyze #file1, #file2..." (use `#` for file attachment)
- **Format:** "Provide [table/narrative/diagram/code]..."
- **Constraints:** "Rules: [preserve precision, use only ANSI SQL, document hard-coded values]..."
- **Checks:** "Verify [accuracy, completeness], list [assumptions, edge cases]..."
