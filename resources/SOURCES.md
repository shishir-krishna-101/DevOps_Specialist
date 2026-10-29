# Source Map

This file records the sources that inform the learning paths in this repository. It is a source map, not a claim that any source defines an official DevOps curriculum.

## Source priority

For technical claims, the repository follows this order of preference:

1. Official documentation
2. Official GitHub repositories
3. Official specifications, RFCs, and standards
4. Official project or vendor documentation
5. Official engineering blogs
6. Other authoritative technical sources

When an authoritative source is available, it takes precedence over secondary tutorials or summaries.

## Python

### Python documentation

- **Source:** Python Documentation
- **URL:** https://docs.python.org/3/
- **Use:** Language reference, standard library behavior, tutorials, and Python-specific technical verification.
- **Repository role:** Primary authority for Python language and standard-library facts.

### Python for DevOps learning sequence

- **Source:** iam-veeramalla/python-for-devops
- **URL:** https://github.com/iam-veeramalla/python-for-devops
- **Use:** Practical topic sequence for learning Python with a DevOps focus.
- **Repository role:** Secondary curriculum reference for organizing the Python-for-DevOps path. Individual tool behavior should be verified against the relevant official documentation.

### Python examples and scripts

- **Source:** geekcomputers/Python
- **URL:** https://github.com/geekcomputers/Python
- **Use:** Examples of small Python programs and scripts.
- **Repository role:** Supplementary inspiration only. Code is not copied automatically; examples are reviewed individually before reuse or adaptation, including licensing requirements.

## Data Structures & Algorithms

### The Algorithms — Python

- **Source:** TheAlgorithms/Python
- **URL:** https://github.com/TheAlgorithms/Python
- **Use:** Educational implementations of algorithms and data structures in Python.
- **Repository role:** Primary implementation reference for the DSA learning path, with the repository selecting a focused beginner-friendly subset rather than reproducing the entire project.

### Tech Interview Handbook

- **Source:** Tech Interview Handbook
- **URL:** https://www.techinterviewhandbook.org/
- **Algorithms study material:** https://www.techinterviewhandbook.org/algorithms/study-cheatsheet/
- **Study plan:** https://www.techinterviewhandbook.org/coding-interview-study-plan/
- **Use:** Algorithm study guidance, practical interview preparation, and curated problem-solving direction.
- **Repository role:** Supplementary study-planning and practice reference. The previously supplied `/algorithms` URL is no longer the current algorithms landing page; the current site links to the study cheatsheet and study plan above.

### LeetCode Python solutions

- **Source:** Garvit244/Leetcode
- **URL:** https://github.com/Garvit244/Leetcode
- **Use:** A catalog of Python solutions to LeetCode problems.
- **Repository role:** Supplementary practice reference. Problems may inform a curated practice index, but the repository should avoid copying solutions wholesale.

## Kubernetes and DevOps applications

### Kubernetes Python client

- **Source:** kubernetes-client/python
- **URL:** https://github.com/kubernetes-client/python
- **Use:** Official Python client for interacting with the Kubernetes API.
- **Repository role:** Optional applied reference for advanced Python-for-DevOps material. Kubernetes API behavior and client usage should be checked against the project documentation and relevant Kubernetes documentation before examples are added.

### Python for DevOps secondary topic reference

- **Source:** GeeksforGeeks — Python for DevOps
- **URL:** https://www.geeksforgeeks.org/devops/python-for-devops/
- **Use:** Secondary checklist of Python applications in configuration management, testing, cloud automation, deployment, and monitoring/operations.
- **Repository role:** Topic-discovery reference only. Product-specific commands, APIs, and configuration are not treated as authoritative from this source when official documentation exists.

## How sources are used

### Verified fact

A technical statement about Python, Kubernetes, an API, a command, a configuration field, or another product behavior should be supported by an authoritative source when one is available.

### Repository design decision

The ordering of topics, the split between DevOps-focused and complete paths, the amount of practice, and the choice to omit or defer a topic are decisions made for this repository. They are not presented as official requirements for becoming a DevOps engineer.

### Demonstration or experiment

A script or example created to demonstrate a concept is a repository learning artifact. Its behavior should not be presented as proof of how an unrelated production system works unless that behavior has been independently verified.

## Deduplication policy

The repository intentionally separates **learning sequence** from **source ownership**:

- Core Python concepts needed by DevOps are introduced in **Python for DevOps**.
- **Complete Python** continues from that foundation instead of repeating the same lessons.
- DSA concepts needed by DevOps are introduced in **DSA for DevOps**.
- **Complete DSA** continues from that foundation instead of duplicating the focused path.
- A practice problem should link to a canonical explanation or implementation when possible rather than maintaining multiple copies of the same solution.

## External code and licensing

A source being useful for learning does not automatically grant permission to copy its code into this repository. Before reusing external code, check the source repository's license and attribution requirements and keep copied material within those terms.

## Verification boundary

This source map records the sources researched for the initial repository architecture. It does not establish technical details that have not yet been researched. Future topic-specific files should add or update source references when they introduce claims requiring additional authoritative verification.
