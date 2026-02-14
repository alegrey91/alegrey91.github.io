# Environment-Aware Vulnerability Suppression Using Kubernetes Security Contexts and VEX

## Abstract

Software Bill of Materials (SBOM)-based vulnerability scanners frequently report vulnerabilities that are theoretically present but practically non-exploitable due to environmental constraints. This phenomenon contributes to alert fatigue and complicates vulnerability prioritization, particularly in large-scale Kubernetes environments.

This paper explores an environment-aware approach to vulnerability suppression based on Kubernetes Security Contexts and the Vulnerability Exploitability eXchange (VEX) specification. By leveraging Common Weakness Enumeration (CWE) classifications and statically analyzing workload deployment manifests, we propose a heuristic method for identifying vulnerabilities that are mitigated by configuration and expressing this information through automatically generated VEX documents. While exploratory in nature, this approach demonstrates how platform-level security controls can be incorporated into vulnerability assessment pipelines to improve signal quality and reduce noise.

## 1. Introduction

Vulnerability scanning has become a foundational practice in modern software supply chain security. Tools based on SBOM analysis enable organizations to detect known vulnerabilities across complex dependency graphs. However, these tools often lack contextual awareness of the runtime environment in which applications execute.

As a result, vulnerability reports frequently include findings that are not exploitable in practice. This disconnect increases the cognitive burden on security teams and shifts effort toward manual triage rather than remediation.

The Vulnerability Exploitability eXchange (VEX) specification addresses this issue by providing a standardized mechanism to communicate exploitability assessments. In practice, however, VEX documents are typically authored manually by security experts or developers with deep system knowledge.

This paper investigates whether exploitability assessments can be partially automated by leveraging declarative security constraints provided by Kubernetes.

## 2. Background

### 2.1 VEX

VEX is a machine-readable format for communicating the exploitability status of vulnerabilities in software products. It allows producers or operators to assert that a vulnerability is not exploitable due to specific conditions, such as unreachable code paths or mitigating controls.

VEX is supported by modern vulnerability scanners, including Trivy, enabling suppression of findings when appropriate justification is available.

### 2.2 Kubernetes Security Contexts

Kubernetes Security Contexts allow operators to constrain container execution through declarative configuration. Examples include disabling privilege escalation, enforcing read-only filesystems, and restricting Linux capabilities.

These controls are commonly applied to reduce attack surface but are not typically considered during vulnerability assessment.

### 2.3 CWE Classification

Each CVE is commonly associated with one or more Common Weakness Enumerations (CWEs), which describe abstract categories of software weaknesses. CWEs provide a potential bridge between individual vulnerabilities and generalized mitigation strategies.

## 3. Problem Statement

Current SBOM-based vulnerability scanning approaches treat exploitability primarily as a property of the software artifact itself. This ignores environmental constraints that may prevent exploitation entirely.

The central research question explored in this paper is:

> Can Kubernetes Security Contexts be used to infer, in a systematic way, that certain classes of vulnerabilities are mitigated by configuration, and can this information be expressed via automatically generated VEX documents?

## 4. Methodology

### 4.1 Overview

The proposed approach consists of three main steps:

1. Mapping CWE classes to configuration-level mitigations
2. Static analysis of Kubernetes deployment manifests
3. Automatic generation of VEX statements for mitigated vulnerabilities

### 4.2 CWE-to-Mitigation Mapping

For a selected set of CWE classes, we define heuristic mappings to Kubernetes Security Context constraints that are expected to mitigate exploitation.

For example, vulnerabilities classified as **CWE-732 (Incorrect Permission Assignment for Critical Resource)** often require write access to sensitive filesystem locations.

An illustrative case is **CVE-2022-42972**, which enables privilege escalation by modifying a webroot directory. If the following conditions hold:

- `readOnlyRootFilesystem: true`
- `volumeMounts[].readOnly: true`

then the vulnerability is considered mitigated with respect to that deployment.

The current implementation of these mappings is available in the prototype system [`vex8s`](https://github.com/alegrey91/vex8s).

### 4.3 Static Manifest Analysis

Kubernetes workload manifests are statically parsed to extract relevant Security Context settings at both Pod and container levels. These settings are evaluated against the mitigation rules associated with each CWE.

If all required conditions are satisfied, the corresponding CVE is marked as mitigated.

### 4.4 VEX Document Generation

For each mitigated CVE, a VEX statement is generated indicating that the vulnerability is not exploitable due to configuration-based mitigations. These statements can then be consumed by SBOM scanners to suppress findings in vulnerability reports.

## 5. Discussion

This approach treats deployment configuration as first-class security metadata. Rather than relying solely on software composition, it incorporates platform-level guarantees into exploitability assessment.

The method is particularly well-suited to Kubernetes environments, where security controls are declarative and amenable to static analysis. By operating at the CWE level, the approach generalizes across individual CVEs while maintaining a reasonable level of abstraction.

However, this generalization is also a source of imprecision, as discussed in the following section.

## 6. Limitations

The proposed method has several limitations:

1. **Heuristic Nature**  
   CWE classes are broad, and not all vulnerabilities within a class share identical exploitation requirements. This may result in false positives or false negatives.

2. **Static Analysis Only**  
   The approach does not account for runtime behavior, kernel-level vulnerabilities, or external attack vectors that bypass container-level constraints.

3. **Configuration Drift**  
   Generated VEX documents are only valid as long as the analyzed deployment configuration remains unchanged.

4. **Non-Substitutive**  
   This method does not replace expert vulnerability analysis but rather aims to reduce noise and improve prioritization.

## 7. Related Work (Informal)

Prior work on vulnerability prioritization has explored exploitability scoring (e.g., EPSS) and reachability analysis at the code level. This work differs by focusing on *platform-level mitigations* rather than application logic or probabilistic scoring.

To the author’s knowledge, limited prior research exists on automated VEX generation driven by orchestration-layer security semantics.

## 8. Conclusion

This paper presents an exploratory approach to environment-aware vulnerability suppression by combining Kubernetes Security Context analysis with CWE-based reasoning and VEX generation.

By encoding platform-level mitigations into structured exploitability metadata, vulnerability reports can more accurately reflect real-world risk. While the approach is inherently heuristic and configuration-dependent, it demonstrates the potential value of integrating orchestration semantics into software supply chain security workflows.

## 9. Future Work

Future research directions include:

- Empirical validation of CWE-to-mitigation mappings
- Expansion to additional Kubernetes controls (e.g., seccomp, SELinux, AppArmor)
- Integration with runtime security signals
- Extension to other orchestration platforms beyond Kubernetes
- Formalization of exploitability semantics and confidence levels in VEX statements

