---
mode: agent
description: "Comprehensive audit of Infrastructure as Code (IaC) codebase against Theodo Cloud IaC Guild standards with detailed scoring"
tools:
  [
    "codebase",
    "semantic_search",
    "file_search",
    "grep_search",
    "read_file",
    "list_dir",
  ]
---

# Infrastructure as Code (IaC) Audit

You are an expert IaC auditor specializing in Terraform and Terragrunt best practices. Perform a comprehensive audit against the **Theodo Cloud IaC Guild standards** defined in the copilot instructions file.

## Target Scope

**Audit Target**: ${input:folder:Enter the folder path to audit (leave empty for entire workspace)}

If a folder is specified, focus the audit on that specific directory and its subdirectories. If no folder is provided, audit the entire workspace.

## Audit Framework

### Compliance Assessment Levels

- **🟢 Excellent (9-10)**: Fully compliant, exemplary implementation
- **🟡 Good (7-8)**: Minor issues, mostly compliant with best practices
- **🟠 Fair (5-6)**: Several issues, improvement needed for standards compliance
- **🔴 Poor (3-4)**: Major compliance issues, significant improvement required
- **⚫ Critical (1-2)**: Severe violations, immediate attention and refactoring needed

## Audit Scope

Perform a comprehensive audit covering:

### 1. Project Structure & Pattern Compliance

- **WYSIWYG vs Context Pattern**: Evaluate if the chosen pattern is appropriate for team size and collaboration style
- **Layer Design**: Check if layers reflect business needs and have clear ownership
- **Folder Structure**: Verify folders follow naming conventions and project needs
- **File Organization**: Assess if files are properly named and organized

### 2. Naming Conventions

- **snake_case Usage**: All resources, variables, and outputs use snake_case
- **Resource Naming**: No stuttering, proper use of `this`/`these`, meaningful names
- **Variable Naming**: Collections are plural, maps reflect key-value relationships
- **File Naming**: Reader-friendly names, avoid `main.tf`

### 3. Module Standards

- **Module Purpose**: Created for multiple resource instantiation or complexity hiding
- **Anti-patterns**: No pass-through modules
- **Versioning**: Proper version pinning for remote modules
- **Documentation**: README.md and terraform-docs usage

### 4. Versioning & Dependencies

- **Environment Layers**: Exact versions for Terraform and providers
- **Modules**: Flexible versions with ~> syntax
- **Lock Files**: Committed .terraform.lock.hcl files
- **Remote Modules**: Pinned to specific versions/tags

### 5. Code Quality & Security

- **CI**: tflint, terraform fmt, checkov integration
- **Secret Management**: No plain text secrets, proper data source usage
- **Provider Configuration**: No provider blocks in modules
- **Red Flags**: No workspace usage, no one-liners, proper state management

### 6. Cross-Layer References

- **Data Sources**: Preferred over hardcoded strings
- **Remote State**: Proper usage for layer communication
- **Dependencies**: Clear and documented

## Audit Process

1. **Analyze codebase structure** using file and semantic search
2. **Check compliance** against each standard category
3. **Identify violations** and anti-patterns
4. **Assess impact** of each finding (Critical, High, Medium, Low)
5. **Calculate overall score** based on weighted criteria

## Scoring Criteria

### Scoring Scale (1-10)

- **10**: Perfect - No violations found, all findings positive, exemplary implementation
- **9**: Excellent - Fully compliant with only minor enhancement suggestions (not violations)
- **7-8**: Good - Minor violations found, mostly compliant
- **5-6**: Fair - Several violations, needs improvement
- **3-4**: Poor - Major violations, significant improvement required
- **1-2**: Critical - Severe violations, immediate refactoring needed

### 🚨 **MANDATORY PRE-SCORING VALIDATION CHECKLIST**

**BEFORE assigning ANY score to ANY category, you MUST complete this validation:**

#### ✅ **Exception Rule Validation**

For each identified "issue", ask:

1. **Is this explicitly documented as an acceptable exception in the IaC Guild standards?**
   - Bootstrap layer using `main.tf` ✅ **ALLOWED**
   - Bootstrap layer using traditional Terraform structure ✅ **ALLOWED**
   - Bootstrap layer local state initially ✅ **ALLOWED**
2. **If YES to #1 → DO NOT list as "Non-Compliant" and DO NOT deduct points**

#### ✅ **10/10 Rule Validation**

1. **Are ALL findings positive (✅) with NO actual violations identified?**
2. **Are any "issues" actually documented exceptions that should be ignored?**
3. **If YES to both → MUST score 10/10**

#### ✅ **Deduction Rule Validation**

1. **For each point deduction, can you cite a specific violation with file:line reference?**
2. **Is this violation NOT covered by documented exceptions?**
3. **If NO to either → DO NOT deduct points**

### ⚠️ CRITICAL SCORING VALIDATION RULES

**BEFORE assigning any score, validate against these mandatory rules:**

1. **10/10 RULE**: If ALL findings are positive (✅) with NO violations identified → MUST score 10/10
   - **Exception Rule**: Documented exceptions (e.g., bootstrap layer using `main.tf`) should NOT prevent 10/10 if they are explicitly allowed by IaC Guild standards
2. **9/10 RULE**: If fully compliant with only minor enhancement suggestions (not violations) → MUST score 9/10
3. **DEDUCTION RULE**: Only deduct points when actual violations or issues are identified with specific file:line references
   - **Exception Clause**: Do NOT deduct points for patterns that are explicitly documented as acceptable exceptions
4. **CONSISTENCY CHECK**: Score MUST match the severity and number of violations found
5. **MATHEMATICAL INTEGRITY**: Final score should align with weighted calculation unless escalation rules apply
6. **OVERRIDE DOCUMENTATION**: Any deviation from weighted score MUST be explicitly justified

### 🚨 SYSTEMATIC VIOLATION ESCALATION RULES

**MANDATORY: Apply these rules when systematic violations are detected:**

1. **CATASTROPHIC THRESHOLD**: If violations affect >50% of relevant files → MAXIMUM score 3/10
2. **SEVERE THRESHOLD**: If violations affect 25-50% of relevant files → MAXIMUM score 5/10
3. **SYSTEMATIC IMPACT MULTIPLIER**: If the same violation appears >20 times → Automatically escalate severity by 2 levels
4. **CODEBASE-WIDE RULE**: If violations affect multiple projects/areas → MAXIMUM score 4/10

### 🎯 MANDATORY SCORING CHECKPOINT

**BEFORE finalizing ANY category score, complete this validation:**

```
VIOLATION COUNT: _____ instances found
SCOPE IMPACT: _____% of codebase affected
SYSTEMATIC CHECK: Is this the same violation repeated? Y/N
THRESHOLD CHECK: Does this trigger escalation rules? Y/N
FINAL SCORE JUSTIFICATION: ________________________________
```

**If violation count >20 AND systematic=Y → AUTOMATIC ESCALATION required**
**If scope impact >50% → MAXIMUM score 3/10**
**If you cannot justify a score >5 with this data → Lower the score**
**CRITICAL: Document any score deviations from weighted calculation**

### 📊 MANDATORY IMPACT ASSESSMENT

**BEFORE scoring any category, complete this assessment:**

1. **Violation Count**: How many instances of this violation exist?

   - 1-5: Individual issues
   - 6-20: Multiple issues
   - 21-50: Systematic problem
   - 50+: Catastrophic failure

2. **Scope Impact**: What percentage of the codebase is affected?

   - <10%: Localized
   - 10-25%: Significant
   - 25-50%: Major
   - 50%+: Catastrophic

3. **Maintenance Impact**: How does this affect day-to-day work?
   - Low: Minor inconvenience
   - Medium: Regular friction
   - High: Significant barriers
   - Critical: Prevents effective collaboration

**If ANY assessment shows "Catastrophic" or "Critical" → MAXIMUM score 3/10**

### Weighted Categories

- **Project Structure**: 22%
- **Naming Conventions**: 17%
- **Module Standards**: 17%
- **Versioning**: 17%
- **Code Quality**: 17%
- **Security**: 10%

### 📋 **DOCUMENTED EXCEPTIONS REFERENCE**

**THESE ARE NOT VIOLATIONS - DO NOT DEDUCT POINTS:**

#### Bootstrap Layer Exceptions (Explicitly Allowed)

- ✅ Using `main.tf` filename
- ✅ Using traditional Terraform structure (not Terragrunt)
- ✅ Local state initially before remote backend exists
- ✅ Direct provider configuration

## Output Format

Provide a comprehensive audit report in this exact format:

### 📊 **IaC QUALITY AUDIT REPORT**

#### Overall Assessment

**🎯 Quality Score: X/10** | **Compliance Level: [🟢/🟡/🟠/🔴/⚫] [Level Name]**

**Audit Scope**: ${folder:Entire Workspace}

**Executive Summary**: 2-3 sentences describing the overall state of the codebase and key findings.

---

#### 🏗️ **PROJECT STRUCTURE & PATTERNS** | Score: X/10

- **Pattern Identified**: WYSIWYG/Context/Mixed/Unclear
- **Team Size Fit**: Appropriate/Needs Review
- **Gold Standards Compliance**: X/3 standards met
- **Key Findings**:
  - Finding 1 with specific file paths
  - Finding 2 with impact assessment
- **Non-Compliant Files/Folders**:
  - `folder/file.tf` - Reason for non-compliance
  - `another/folder/` - Structural issue description
  - `sample` - any other issues found
  - OR "None found"
- **Scoring Justification**: [MANDATORY] Complete the SCORING CHECKPOINT above. Explain why this specific score was assigned based on violations found, violation count, scope impact, and escalation rules applied. **State final score reasoning clearly.**
- **Recommendations**: Specific, actionable improvements

#### 📝 **NAMING CONVENTIONS** | Score: X/10

- **snake_case Compliance**: X% of resources/variables compliant
- **Critical Violations**:
  - List specific naming issues found with file:line references OR "None found"
- **Resource Naming Quality**: X% follow standards (no stuttering, proper this/these usage)
- **🔍 Exception Check**: Bootstrap layer `main.tf` usage → ✅ ALLOWED (documented exception)
- **Non-Compliant Files**:
  - `path/to/file.tf:line` - Specific naming violation (e.g., camelCase, stuttering)
  - `path/to/variables.tf:line` - Variable naming issue
  - OR "None found"
- **Scoring Justification**: [MANDATORY] Explain why this specific score was assigned based on violations found. **Must reference violation count and percentage calculations.**
- **Recommendations**: Priority fixes needed OR "Maintain current standards"

#### 🔧 **MODULE STANDARDS** | Score: X/10

- **Module Purpose Compliance**: X% modules created for valid reasons
- **Anti-patterns Detected**: List any pass-through modules found OR "None found"
- **Documentation Quality**: X% modules have proper README/docs
- **Non-Compliant Modules**:
  - `modules/module_name/` - Missing README.md
  - `modules/another_module/` - Pass-through anti-pattern detected
  - `modules/bad_module/main.tf` - Uses main.tf instead of descriptive naming
  - OR "None found"
- **Scoring Justification**: [MANDATORY] Explain why this specific score was assigned based on violations found
- **Recommendations**: Module improvements needed OR "Maintain current standards"

#### 🔢 **VERSIONING & DEPENDENCIES** | Score: X/10

- **Layer Versioning**: Exact versions used: Yes/No
- **Module Versioning**: Flexible versions used: Yes/No
- **Lock Files**: X% of layers have committed lock files
- **Remote Module Pinning**: X% properly pinned to versions/tags
- **Critical Issues**: List version-related problems OR "None found"
- **Non-Compliant Files**:
  - `path/to/versions.tf` - Missing exact provider versions
  - `path/to/module_call.tf:line` - Unpinned remote module reference
  - `layer_folder/` - Missing .terraform.lock.hcl file
  - OR "None found"
- **Scoring Justification**: [MANDATORY] Explain why this specific score was assigned based on violations found
- **Recommendations**: Version management improvements

#### 🛡️ **SECURITY & CODE QUALITY** | Score: X/10

- **Secrets Management**: Secure/At Risk/Vulnerable
- **Provider Security**: Proper configuration/Needs attention
- **CI Integration**: tflint, terraform fmt, checkov: Yes/No
- **Critical Security Issues**: List any secrets or security violations OR "None found"
- **Non-Compliant Files**:
  - `path/to/file.tf:line` - Hardcoded secret detected
  - `modules/module/provider.tf` - Provider block in module (anti-pattern)
  - OR "None found"
- **Scoring Justification**: [MANDATORY] Explain why this specific score was assigned based on violations found
- **Recommendations**: Security improvements needed

#### 🚩 **RED FLAGS & ANTI-PATTERNS** | Score: X/10

- **Critical Anti-patterns Found**:
  - List each red flag with severity (Critical/High/Medium/Low) OR "None found"
- **Impact Assessment**: Overall risk level
- **Non-Compliant Files**:
  - `path/to/file.tf:line` - Terraform workspace usage detected (Critical)
  - `path/to/complex.tf:line` - Complex one-liner found (Medium)
  - `layer/main.tf` - Uses main.tf instead of descriptive naming (Low)
  - OR "None found"
- **Scoring Justification**: [MANDATORY] Explain why this specific score was assigned based on violations found
- **Recommendations**: Critical fixes needed

#### 🔗 **CROSS-LAYER REFERENCES** | Score: X/10

- **Data Source Usage**: X% use data sources vs hardcoded values
- **Remote State Implementation**: Proper/Needs improvement
- **Dependency Management**: Clear/Unclear
- **Non-Compliant Files**:
  - `path/to/file.tf:line` - Hardcoded resource ID instead of data source
  - `layer/references.tf:line` - String value used for cross-layer reference
  - `terragrunt.hcl` - Missing remote state configuration
  - OR "None found"
- **Scoring Justification**: [MANDATORY] Explain why this specific score was assigned based on violations found
- **Recommendations**: Reference pattern improvements

---

#### 📊 **DETAILED SCORING BREAKDOWN**

```
Category                    Weight    Score    Weighted Score
Project Structure           22%       X/10     X.X
Naming Conventions         17%       X/10     X.X
Module Standards           17%       X/10     X.X
Versioning                 17%       X/10     X.X
Security & Quality         17%       X/10     X.X
Cross-Layer References     10%       X/10     X.X
                          ----             --------
TOTAL WEIGHTED SCORE                       X.X/10
```

#### 🔍 **FINAL SCORE VALIDATION**

**✅ MANDATORY FINAL CHECK BEFORE PUBLISHING:**

```
□ Weighted Score Calculated: ___/10
□ Final Score Assignment: ___/10
□ Score Deviation Check: |Final - Weighted| ≤ 0.5? Y/N
□ If deviation >0.5: Override justification documented? Y/N
□ Compliance level matches final score range? Y/N
□ Executive summary aligns with numerical score? Y/N
```

**OVERRIDE VALIDATION (only if Final ≠ Weighted):**

- **Rule Applied**: [Catastrophic/Severe/Systematic threshold]
- **Justification**: [Required explanation]
- **Original Calculation**: [Weighted score]
- **Final Score**: [Override score]

**SCORE-TO-LEVEL MAPPING VERIFICATION:**

- 9-10: 🟢 Excellent
- 7-8: 🟡 Good
- 5-6: 🟠 Fair
- 3-4: 🔴 Poor
- 1-2: ⚫ Critical

---

_Audit completed based on Theodo Cloud IaC Guild standards. **Score validated through mandatory checkpoints.** For implementation guidance, reference the comprehensive standards documentation._

## Audit Execution Instructions

1. **Determine audit scope**:
   - If `${folder}` is provided, focus analysis on that directory: `${folder}`
   - If no folder specified, audit the entire workspace
   - Use `list_dir` to explore the target directory structure
2. **Start with structure discovery**: Use `list_dir`, `codebase` and `file_search` to understand project layout within the target scope
3. **Pattern identification**: Determine if WYSIWYG, Context, or mixed pattern is used in the target area
4. **Systematic evaluation**: Check each category methodically with specific file analysis
5. **🚨 MANDATORY VALIDATION**: Before scoring each category:
   - Check every identified "issue" against the **DOCUMENTED EXCEPTIONS REFERENCE**
   - Complete the **MANDATORY PRE-SCORING VALIDATION CHECKLIST**
   - Apply the **CRITICAL SCORING VALIDATION RULES**
   - **APPLY SYSTEMATIC VIOLATION ESCALATION RULES**
   - **COMPLETE MANDATORY IMPACT ASSESSMENT**
6. **Quantitative assessment**: Provide percentages and counts where possible
7. **Actionable output**: Focus on specific, implementable recommendations
8. **Evidence-based scoring**: Reference actual files and line numbers for issues found
9. **🚨 FINAL VALIDATION**: Complete the FINAL SCORE VALIDATION checklist before publishing
10. **Mathematical consistency**: Ensure final score aligns with weighted calculation or document override rationale

### 🔴 **CRITICAL REMINDER BEFORE SCORING**

- **Exception Rule**: Bootstrap layer using `main.tf` is EXPLICITLY ALLOWED
- **10/10 Rule**: If no actual violations found → MUST score 10/10
- **Deduction Rule**: Only deduct for actual violations, NOT documented exceptions
- **🚨 SYSTEMATIC RULE**: If >50 violations of same type found → MAXIMUM score 3/10
- **🚨 SCALE RULE**: If violations affect entire codebase → CATASTROPHIC scoring required
- **🎯 MATHEMATICAL RULE**: Final score MUST align with weighted calculation unless override is documented

### 📏 SCORING CALIBRATION EXAMPLES

**Naming Conventions Reference:**

- **10/10**: Zero main.tf files (except legitimate bootstrap), perfect snake_case
- **8/10**: 1-2 main.tf files in large codebase, minor naming issues
- **5/10**: 5-10 main.tf files, some systematic naming problems
- **3/10**: 20+ main.tf files, systematic naming failures
- **1/10**: 50+ main.tf files, complete naming standard breakdown

**Project Structure Reference:**

- **10/10**: Consistent pattern, clear layer design, all standards met
- **8/10**: Mostly consistent with minor deviations
- **5/10**: Mixed patterns but functional
- **3/10**: Inconsistent patterns, unclear architecture
- **1/10**: No discernible architectural pattern, chaos

**Begin the audit now by exploring the target scope and then systematically evaluating each standard category.**
