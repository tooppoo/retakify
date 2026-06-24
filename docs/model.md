# Retakify Core Model

Retakify is a minimal mediator for a file-based retake loop.

It connects one subject process and one or more inspector processes through files. Retakify does not define what those processes do internally. It only defines how their outputs are passed, inspected, and used to decide whether the subject should be run again.

## Purpose

Retakify exists to support workflows where one process produces a result and other processes inspect that result.

If the inspection results indicate that the subject should be retried, Retakify passes the inspection results back to the subject and runs it again.

The primary purpose is not to provide an agent framework, review framework, CI system, or task manager. Retakify only mediates the retake loop between external processes.

## Core Roles

Retakify has three core roles:

* Subject
* Inspector
* Mediator

### Subject

The subject is the primary process.

It performs the main work and writes result files.

Examples include:

* an implementation agent
* a document generator
* a code transformation tool
* a shell script
* any other process that produces files to be inspected

The subject does not need to know which inspectors will run after it.

### Inspector

An inspector is a process that examines the subject's result files.

It writes inspection result files.

Examples include:

* a review agent
* a test wrapper
* a linter wrapper
* a policy checker
* a consistency checker

An inspector does not need to know how the subject works internally. It only needs to understand the files it receives.

### Mediator

The mediator is Retakify itself.

The mediator runs the subject, runs the inspectors, reads the inspection results, and decides whether another subject attempt is required.

The mediator is the only component that knows the overall loop.

## Process Relationship

The subject and inspectors do not communicate directly.

They are connected only through files managed by the mediator.

```text
subject -> files -> mediator -> inspector
subject <- files <- mediator <- inspector
```

This separation is the central design principle.

Each process may be implemented in any language, use any tool, or wrap any LLM agent, as long as it participates through the expected files.

## Retake Loop

A Retakify run follows this conceptual flow:

1. The mediator runs the subject.
2. The subject writes result files.
3. The mediator runs one or more inspectors.
4. Inspectors write inspection result files.
5. The mediator checks the inspection results.
6. If retake is required, the mediator passes relevant inspection results back to the subject.
7. The subject runs again.
8. The loop stops when retake is no longer required or when the configured limit is reached.

The loop can be summarized as:

```text
subject
  -> inspection
  -> retake decision
  -> subject again if needed
```

## Attempts

Each subject execution is an attempt.

A run may contain one or more attempts.

Retakify should preserve the distinction between attempts because each attempt represents a specific subject output and its corresponding inspection results.

## Boundary

Retakify treats subject and inspector processes as opaque.

Retakify does not control:

* how the subject performs its work
* how inspectors evaluate results
* which LLM, tool, framework, skill, plugin, or script is used
* how findings are produced internally

Retakify only controls:

* when each process is run
* which files are passed between processes
* when the retake loop continues or stops

## Design Principles

### File-based coordination

Files are the interface between processes.

This keeps Retakify independent of specific runtimes, languages, LLM providers, and agent frameworks.

### Process opacity

Subject and inspector internals are outside Retakify's concern.

Retakify should not require special knowledge of Claude, Codex, Gemini, shell scripts, linters, or other tools.

### Mediated interaction

Subject and inspectors should not call each other directly.

The mediator coordinates all interaction.

### Inspection and decision are separate

Inspectors produce inspection results.

The mediator determines whether those results require retake.

### Bounded retry

The retake loop must be bounded.

Retakify should not allow unbounded retries.

## Non-Goals

Retakify is not:

* an LLM agent framework
* a subagent system
* a review methodology
* a prompt management tool
* a CI system
* a GitHub Issue or Pull Request manager
* a plugin package manager

Retakify may run tools that provide those capabilities, but it does not define them.

## Summary

Retakify is a mediator for a file-based subject-inspector retake loop.

Its core model is:

```text
subject   = process that produces results
inspector = process that inspects results
mediator  = process that coordinates execution and retake
```

Its core flow is:

```text
run subject
inspect result
decide retake
repeat if needed
```

Retakify is intentionally small. Its value comes from keeping different processes loosely coupled while still enabling repeated improvement through inspection.
