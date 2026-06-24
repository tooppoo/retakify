# retakify

[![License](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)

retakify is a minimal workflow tool for coordinating generation, inspection, and retake cycles between separate processes.

It is designed for workflows where one process produces an output, another process inspects that output, and the workflow may need to rerun the producer when the inspection result indicates that the output should be revised.

retakify does not try to replace the subject process or the inspector process.
Instead, it provides a small mediation layer that lets them cooperate through files.

## Concept

retakify organizes a workflow around three roles:

* **Subject**: the main process that produces the target output.
* **Inspector**: one or more processes that inspect the subject output.
* **Mediator**: retakify itself, which coordinates the subject, inspectors, and retake flow.

The basic flow is:

1. The subject produces output files.
2. retakify runs one or more inspectors against those files.
3. Inspectors produce inspection results.
4. retakify determines whether another subject run is needed.
5. The cycle continues until the output is accepted or the configured attempt limit is reached.

This keeps each process independent.
The subject does not need to know how many inspectors exist, and inspectors do not need to control the subject directly.

## Why use retakify?

Use retakify when you want to make review-and-retry workflows explicit, reproducible, and process-independent.

Typical motivations include:

* separating generation from inspection;
* running multiple inspectors over the same output;
* keeping subject and inspector implementations loosely coupled;
* making retake cycles visible instead of embedding them inside ad hoc scripts;
* preserving file-based interoperability between different tools, languages, or agents.

retakify is intentionally small.
It is not a general-purpose agent framework, task runner, CI system, or workflow engine. Its core responsibility is to mediate the subject-inspector-retake loop.

## Quick Start

A retakify workflow consists of a subject command and one or more inspector commands.

At a high level, the usage looks like this:

```sh
retakify run
```

The subject produces files.
The inspectors examine those files.
retakify coordinates whether the subject should be run again.

A minimal workflow usually follows this shape:

```text
subject
  -> output files

retakify
  -> run inspectors

inspectors
  -> inspection results

retakify
  -> accept or retake
```

For concrete command usage and configuration examples, see the documents below.

## Documentation

* [Workflow](docs/workflow.md)
  Explains how retakify coordinates the subject, inspectors, and retake cycle.

* [Commands](docs/commands.md)
  Describes the user-facing CLI commands.

* [Configuration](docs/configuration.md)
  Describes how to configure a retakify workflow.

* [Glossary](docs/glossary.md)
  Defines the main terms used by retakify.

## Non-goals

retakify does not aim to:

* define how a subject should generate its output;
* define how an inspector should evaluate output quality;
* replace existing linters, test runners, reviewers, or agents;
* prescribe a specific file schema in the README;
* expose detailed retake rules in the introductory documentation.

Those details belong in more focused documents when needed.

## Status

retakify is currently an early-stage project.
The public documentation focuses on the workflow model and user-facing behavior rather than internal implementation details.
