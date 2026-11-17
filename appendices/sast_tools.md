---
title: Assessing SAST Tools:Criteria for Quality
short_title: The State of Python SAST Tools
---

## Introduction

Python applications have become an increasingly attractive target for malicious actors. Anyone who runs, integrates, or develops Python software should employ a Static Application Security Testing (SAST) tool. SAST tools are essential for keeping systems secure: they proactively identify weaknesses directly in the source code, making them a cost-effective way to reduce cybersecurity risk.

Python SAST tools perform an in-depth, automated review of source code to detect security weaknesses that, depending on the context, can lead to vulnerabilities if left unaddressed.

## The Python SAST Magic Quadrant: Choosing a Python SAST Tool


Static Application Security Testing (SAST) is a crowded market. So how do you choose a good SAST product for Python code that can be used for:

* Testing the Python applications or scripts of others.  
* Testing your own Python code. So to prevent weaknesses from being discovered later, or to ensure your application conforms to a minimum [security baseline](https://nocomplexity.github.io/pythonsecurity/guidelines-intro/) that can be expected of Python applications.

Or are there good products that excel in both areas?

A good question to ask is: is there something like a "Magic Quadrant" for choosing a Python SAST scanner? A well-known global IT advisory company regularly publishes reports featuring a selection matrix designed to help executives choose IT services. These reports appear objective, but they fail against every criterion required for open research. This fuels persistent rumours that companies pay to achieve a better outcome in these assessments.

This report is not that kind of advice. Everyone is invited to discuss the criteria and methodology used here. This is an open advisory report that will be improved with your input, so [contribute](../contribute.md)!

The cybersecurity industry is filled with companies that sell trivial advice at premium prices.

The SAST "Magic Quadrant" below has been created to help you ask the right questions. The core questions for any cybersecurity product should be:

- **Is this product trustworthy?** Cybersecurity is complex enough already. You should be able to trust a security tool and understand its limitations.

:::{image} ../images/magic_quadrant.png
:alt: The Python SAST Magic Quadrant
:align: center
:::




Use this SAST Magic Quadrant to make selecting your SAST scanner easier:

1. Aim for the quadrant: Simple \+ Trustworthy. These scanners are easy to run, produce little noise, and deliver results you can actually trust.  
2. Acceptable second choice: Complex \+ Trustworthy. These require more setup and have a steeper learning curve, but the findings are reliable.  
3. Avoid the left-hand side: a false sense of security, whether simple or complex. These tools look impressive but will miss security issues you need to know about. Many commercial SAST scanners for Python unfortunately fall into the "Complex \+ False Sense of Security" quadrant — the worst of both worlds: hard to use and untrustworthy, and often very expensive to licence as well.

With this Magic Quadrant, you can score SAST scanners for Python against proven evaluation criteria.

## Evaluation criteria

The most important criteria for selecting a Static Application Security Testing (SAST) tool from a security perspective are summarised in the table below.



| Metric | Why this criteria matters |
| :---- | :---- |
| Open Source ([FOSS](https://fsfe.org/freesoftware/comparison.en.html)) | The product must be released under a valid open source software licence, ideally one approved by the [Open Source Initiative (OSI)](https://opensource.org/licenses). This of course means that the source code must be publicly available in a version-controlled repository. |
| No AI use for scanning code | SAST tools that employ AI agents connected to a large language model (LLM) pose a security risk in use, as valuable source code and other artefacts may be retained within an untrusted remote environment. |
| Local-first deployment | The SAST tool can be used without internet connectivity and deployed on premise.&nbsp; |
| Dedicated for Python | Every programming language is unique, so multi-language SAST tools usually have hidden weaknesses that create a false sense of security. Each language needs its own Abstract Syntax Tree(AST) for accurate scanning. Dedicated Python tools consistently outperform “all-in-one” solutions.&nbsp; |
| Method to find weaknesses | Common methods are based on Abstract Syntax Tree (AST) checking or [Taint analysis](https://nocomplexity.com/taint-analysis-vs-ast/) to discover insecure patterns with a regex method. It is crucial, however, that the code under test is never executed in any way since it may contain malware on purpose.. |
| Trustworthy default configuration | The SAST tool can be used without configuration and delivers excellent results without manually configuring scanning rules. |
| No Telemetry | While many telemetry services claim anonymity, privacy risks persist. If the backend systems are closed-source, they rely on security by obscurity, violating a core security principle. |
| Security Policy&nbsp; | A SAST tool must have a security policy to ensure that potential security issues with the tool can be reported effectively. |
| [OpenSSF Best Practices Badge](https://www.bestpractices.dev/en) | This badge demonstrates that the project maintainers take security seriously.&nbsp; |
| Transparent architecture and design | Designing systems requires choices. Some choices have impact which can influence the long term stability and security of the SAST tool. A good SAST tool has an open architecture that is available and makes choices clear. |
| [Reproducible Builds](https://reproducible-builds.org/) | The SAST software package can be verified bit-by-bit from the source.&nbsp; |
| Principle of Least Privilege | No extra access rights are needed to run the SAST tool. |
| SAST Scanning | SAST scanning is applied to the tool itself. Possible weaknesses in the code are marked.. |
| Documentation | The SAST tool has documentation on usage and outlines how the tool works, including limitations. |
| Easy and simple to use | Good security tools should be usable for everyone. Not only for experts that understand unix CLI use. By default no configuration should be needed for good SAST scan results. |



Additional important capabilities for Python SAST scanners:

| Additional SAST tool capabilities | Explanation |
| :---- | :---- |
| Simple CI integration | Integrates easily into CI/CD pipelines and standard code-quality workflows. |
| Scans directly packages on PyPI.org | This feature saves time when deciding whether to adopt a new package from PyPI.&nbsp; |
| Honours suppression markers in the code | Markers in the code indicate that the author was aware of the risks associated with a particular Python construct that may introduce a vulnerability. A good SAST scanner respects markers by default, but users can override this behaviour to review the findings and judge the risks for themselves. |
| Scans [tar.gz](http://tar.gz) files (local or remote) | Many Python packages are not fully open source and a SAST scanner that can directly scan any `sdist` ([tar.gz](http://tar.gz)) file has numerous advantages. |
| Complexity check | Complexity directly impacts security. Scanning the code base and determining a cyclomatic complexity score is a proven and quick way to predict future security misery.&nbsp; |
| Determining used modules | One of the biggest security challenges in modern Python development lies not in your own code, but in the dependencies you use. Determining the [direct and indirect dependencies](https://nocomplexity.github.io/pythonsecurity/dependencies/) used is not simple, but vital for a good SAST scanner. |
| Covers a broad range of validation rules.&nbsp; | Good Python SAST scanners check Python code against constructs that are directly available in the Python Standard Library (PSL). There are many default library calls possible in Python that can cause security weaknesses.&nbsp; |



## Python SAST Landscape overview

For this Python SAST benchmark, I have selected as many FOSS SAST scanners as possible for the comparison. However, since the number of well-maintained Python SAST scanners is very limited, popular SAST scanners that claim to support Python are also included.

More is certainly not always better, so I have set a limit on the number of scanners in this list. Note, however, that this selection is not random: it is an opinionated selection of SAST scanners for Python that are used around the world.



### Open Source SAST Scanners


An open source SAST scanner is one that can be used and deployed on-premises easily, without built-in barriers or limitations on what you can or cannot do with it.

| Name | Documentation | Short description |
| :---- | :---- | :---- |
| [Python Code Audit](https://codeaudit.nocomplexity.com) | [Documentation](https://codeaudit.nocomplexity.com) | A modern Python source code security analyser based on distrust. It offers over 95 validation rules and can be used to check your own code or to scan code written by others. |
| [Opengrep](https://www.opengrep.dev/) | [Documentation](https://github.com/opengrep/opengrep) | FOSS version of Semgrep. Development is a bit staggered. |
| [Bandit](https://github.com/PyCQA/bandit) | [Documentation](https://bandit.readthedocs.io/en/latest/) | One of the few other FOSS Python SAST scanners. |
| [Pysa( Meta Open Source)](https://pyre-check.org/docs/pysa-basics/) | [Documentation](https://pyre-check.org/docs/pysa-basics/) | Type checker with a SAST module (Pysa) that only works with Pyre. Meta FOSS software ([code](https://github.com/facebook/pyre-check)). |



This list is short, because I only consider Python SAST scanners that meet the following criteria:

* Have a valid [Open Source Initiative (OSI)](https://opensource.org/licenses)\-approved licence. For example, a product under the Elastic License 2.0 is not considered Free and Open Source Software (FOSS).  
* Are actively maintained. The code must still be maintained.  
* Primarily aim to detect weaknesses in code. I only include static analysis tools with this focus. Linters are sometimes misused as SAST tools, but the aim of a code linter is entirely different.  
* Are not a security threat in themselves. SAST scanners that are 100% vibe-coded, contain many weaknesses that compromise your security when used, or make use of LLM agents are not included in this list.

I detest products that claim to be open source but use the label 'open source' purely as a marketing tactic, while in reality it is impossible to actually use the code locally after cloning it.



### Closed source SAST scanners

Closed source Python SAST scanners are mostly black-box propriety solutions. Most commercial SAST scanners fall in this category.

Commercial Python SAST scanners are more numerous, but I only list those that are widely known and used, and that do not rely solely on LLMs for the core analysis.



| Name | Documentation | Short description |
| :---- | :---- | :---- |
| [Aikido](https://www.aikido.dev/scanners/static-code-analysis-sast) | [Documentation](https://help.aikido.dev/) | Developer-focused SAST with strong false-positive reduction (built on Opengrep) and AI-assisted triage/autofix. |
| [Bearer](https://github.com/bearer/bearer) | [Documentation](https://docs.bearer.com/) | SAST scanner that prioritizes findings by sensitive data flows (PII/PHI); now part of Cycode. |
| [Black Duck (Coverity)](https://www.blackduck.com/static-analysis-tools-sast/coverity.html) | [Documentation](https://www.blackduck.com/static-analysis-tools-sast/coverity.html) | Enterprise-grade SAST for large, complex codebases with strong C/C++ and compliance support. |
| [Checkmarx One](https://checkmarx.com/cxsast-source-code-scanning/) | [Documentation](https://checkmarx.com/) | Unified AppSec platform with hybrid AI \+ deterministic SAST engine and broad language/framework coverage. |
| [CodeAnt](https://www.codeant.ai/) | [Documentation](https://docs.codeant.ai/) | Agentic platform combining AI SAST with offensive security (pentesting agents) and PR/IDE integration. |
| [CodeQL(Github.com)](https://codeql.github.com/) | [Documentation](https://codeql.github.com/docs/) | Semantic query-based SAST engine from GitHub; treats code as a database for precise taint and variant analysis. |
| [CodeValor](https://www.sentar.com/codevalor/) | [Documentation](https://www.sentar.com/codevalor/) | SAST focused on legacy and specialized languages (including Fortran, Ada); used in DoD environments. |
| [Codex Security(OpenAI)](https://learn.chatgpt.com/docs/security) | [Documentation](https://learn.chatgpt.com/docs/security) | OpenAI’s agentic security tool that reasons over code and validates findings (research preview). |
| [Codiga(DataDog)](https://www.codiga.io/) | [Documentation](https://www.codiga.io/) | Real-time static analysis and automated code reviews with OWASP/CWE rules across IDEs and PRs. |
| [Corgea](https://corgea.com/) | [Documentation](https://docs.corgea.app/introduction) | AI-native SAST that detects complex/business-logic issues and auto-generates fixes (also works on top of other scanners). |
| [DeepSource(Harness)](https://deepsource.com/platform/sast) | [Documentation](https://deepsource.com/) | SAST with very low false-positive claims, autofix, and strong PR integration. |
| [DerScanner](https://derscanner.com/) | [Documentation](https://derscanner.com/) | Full-cycle platform (SAST \+ DAST \+ SCA \+ MAST) with AI triage/fix and broad (including legacy) language support. |
| [Flawnter](https://www.flawnter.com/) | [Documentation](https://www.flawnter.com/) | SAST/DAST/SCA tool with zero-trust design and support for 25+ languages. |
| [Fluid Attacks](https://fluidattacks.com/products/sast) | [Documentation](https://docs.fluidattacks.com/) | SAST tool which claims very high true-positive / low false-positive rates on OWASP Benchmark; part of broader AppSec platform. |
| [OpenText Fortify SAST](https://www.opentext.com/products/static-application-security-testing) | [Documentation](https://www.opentext.com/products/static-application-security-testing) | Enterprise SAST tool with extensive language/framework coverage and strong compliance focus. |
| [HCL AppScan Source](https://www.hcl-software.com/appscan/products/appscan-source) | [Documentation](https://www.hcl-software.com/appscan/products/appscan-source) | On-premises enterprise SAST with AI-assisted analytics and broad language support. |
| [JFrog SAST](https://jfrog.com/sast/) | [Documentation](https://docs.jfrog.com/security/docs/sast-1) | Fast, local, developer-focused SAST integrated into the JFrog platform and Frogbot. |
| [Lucent Sky AVM](https://www.lucentsky.com/en/avm) | [Documentation](https://www.lucentsky.com/en/avm) | Kind of a SAST tool that automatically generates and applies “Instant Fixes” for many vulnerability classes. |
| [Mend SAST](https://www.mend.io/sast/) | [Documentation](https://docs.mend.io/platform/latest/mend-sast) | SAST tool with AI remediation, integrated into the Mend AppSec platform. |
| [Ox Security](https://www.ox.security/) | [Documentation](https://docs.ox.security/) | Context-aware AppSec platform that correlates SAST findings with runtime, reachability and business context. |
| [Perforce Klocwork](https://www.perforce.com/products/klocwork) | [Documentation](https://www.perforce.com/products/klocwork) | Scalable SAST for large codebases (strong C/C++/Java etc.) with compliance standards support. |
| [Semgrep](https://semgrep.dev/) | [Documentation](https://semgrep.dev/docs/) | Flexible SAST (plus SCA & secrets) with a rule-driven scanner that matches patterns over a parsed tree. |
| [Snyk Code](https://snyk.io/product/snyk-code/) | [Documentation](https://docs.snyk.io/scan-with-snyk/snyk-code) | Developer-first AI-powered SAST with real-time IDE/PR results and autofix suggestions. |
| [Sonar (SonarQube / SonarCloud)](https://www.sonarsource.com/) | [Documentation](https://docs.sonarsource.com/) | Tool for code verification and automated code review.&nbsp; |



## Python Benchmarking Suite

A good question for anyone who places trust in cybersecurity tools is: "How good is this tool?" Of course, no tool is perfect. Even AI-powered cybersecurity tools fail in many ways, so human critical thinking about weaknesses found, context, and any necessary safety improvements should never be outsourced to a tool.

This question is especially relevant for Python SAST scanners. Most scientific benchmarking suites, such as [PySASTBench](https://github.com/Victor725/PySASTBench), comprise a collection of real-world Python programs with known CVEs. However, this approach has two fundamental problems.

1. The majority of weaknesses and real-world vulnerabilities in Python programs and scripts are never reported or collected as CVEs, mainly due to the complexity of registering a vulnerability as a CVE and the time this requires.  
2. The number of CVEs relating to Python software is extremely low, given the enormous number of weaknesses and vulnerabilities present in Python packages.

Consequently, no simple, reusable, and broadly applicable validation suite exists to check the correctness of various Python SAST scanners. I therefore created a standard validation suite to assess their quality. The results are striking.

You can and should validate your current Python SAST scanner against this benchmark. If you are in the process of choosing a new or additional Python SAST scanner, you should definitely use these easy-to-use test suites. You can find the suite on [GitHub](https://github.com/nocomplexity/codeaudit/tree/main/tests/benchmarkfiles); for more information, see the [documentation](https://nocomplexity.com/documents/codeaudit/benchmarksuite.html#benchmark-suite).



## Benchmarking results

This section presents observations drawn from evaluating open source and commercial Python SAST tools against:

* our evaluation criteria, and  
* our simple Python-specific benchmarking suite.

We make the following observations:

* The number of open source Python SAST scanners with an [OSI-approved licence](https://opensource.org/licenses) is very limited.  
* Almost all Python SAST scanners fail on our benchmark suite when faced with obfuscation, homoglyphs used to mask dangerous commands, import aliasing, and obfuscated dynamic code execution via built-in aliasing. This is worrying, to put it mildly.  
* AI agent-based SAST scanners are rapidly gaining ground, but besides their reproducibility disadvantages, these products pose significant privacy risks and should not be trusted with non-public code.  
* Most SAST scanners available for Python claim to support many programming languages. Using a multi-language SAST scanner for Python runs the risk of producing a tool that is mediocre at best for Python.  
* Most Python SAST scanners are designed to scan code you work on or 'own'. Very few make it easy for you to validate code directly from PyPI or from `sdist` files at a remote location.  
* Most SAST tools for Python are SaaS-based. From a security and privacy perspective, this poses severe risks.  
* The number of validation rules available without configuration is very limited for most Python SAST tools.  
* Most Python SAST tools offer no simple way to assess the trustworthiness and quality of their weakness detection. Using a [benchmark collection](https://nocomplexity.com/documents/codeaudit/benchmarksuite.html#how-to-use) of Python files with known weaknesses should be straightforward, but in reality, commercial tools impose many barriers that prevent their quality from being evaluated before purchase.



## Discussion and conclusions

Despite the abundance of static application security testing (SAST) scanners available for Python, the overall quality of most of these tools is mediocre at best. Selecting a suitable Python SAST scanner is therefore far from trivial. For practitioners who are not experts in technical security quality metrics, it is particularly difficult to look beyond the marketing rhetoric used by cybersecurity vendors. To support informed decision-making, we created a simple [benchmark](https://nocomplexity.com/documents/codeaudit/benchmarksuite.html#benchmark-suite) that measures how different Python SAST scanners perform on a set of deliberately trivial Python weaknesses. Even such elementary checks prove discriminative: scanners that fail on trivial weaknesses are unlikely to perform reliably on complex ones.



The methods employed by Python SAST scanners differ considerably, and these differences matter for ease of use, performance, and default coverage. AST-based checking is simple, fast, and highly effective for pattern-like weaknesses, whereas some scanners additionally employ taint analysis to detect data-flow vulnerabilities such as injection flaws. [Methods matter:](https://nocomplexity.github.io/pythonsecurity/ast-vs-taint/#ast-based-analysis-vs-taint-analysis-for-sast) when a scanner is open source, its weaknesses can be inspected directly, which facilitates informed evaluation. A "holy grail" tool does not exist. Nevertheless, we recommend prioritising open source tools that are specifically engineered for the Python programming language, rather than generic multi-language scanners with shallow Python support.



When selecting a Python SAST tool, focus on its core purpose: code scanning. The broader domain of cybersecurity encompasses cross-site scripting (XSS), SQL injection, secrets scanning, correct cryptographic usage, and many other concerns. Each of these areas requires specialised expertise, and some are better addressed with dynamic application security testing (DAST), which evaluates a Python application while it is running in order to identify exploitable vulnerabilities. DAST is typically required for classes of threats that arise in internet-facing applications, where runtime behaviour — not source code alone — determines the security posture. A sound [security testing strategy](https://securitytesting.nocomplexity.com/) therefore combines SAST and DAST rather than relying on either exclusively.



Finally, our results warrant a critical note on the recent wave of AI-based SAST products, many of which overpromise and underdeliver:

1. Large language models (LLMs) are trained on yesterday's attack patterns and remediations, not on today's challenges. Their knowledge is inherently retrospective.  
2. Most threat models and security architectures are not published as open-access material on the internet, and are therefore invisible to models trained on public data.  
3. Security is a process, not a product. No security tool can replace the human factor, which remains crucial in cyber defence.  
4. Risk assessment cannot be outsourced to an AI agent or tool. Risk assessment is context-specific and involves humans, organisational processes, business systems, and domain knowledge that no model possesses.  
5. Many AI-based solutions impose questionable network requirements — frequently amounting to "it only requires our private, trusted VPN tunnel" — which themselves introduce additional attack surface and trust dependencies.

In conclusion, organisations should evaluate Python SAST scanners empirically, using benchmarks that expose both methodological strengths and blind spots, and should treat AI-assisted analysis as a complement to expert human judgement.

