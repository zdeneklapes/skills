---
name: vertical-slice-architecture
description: Design, review, refactor, or implement systems using Vertical Slice Architecture. Use when the user asks about feature folders, slice boundaries, layered-to-feature refactors, CQRS, handlers, validators, DTO placement, server/backend architecture, full-stack feature organization, or applying Vertical Slice Architecture to machine learning training, inference, model evaluation, model serving, MLOps, batch scoring, or ML platform codebases.
---

# Vertical Slice Architecture

Use this skill to help design, review, refactor, or implement software around cohesive business capabilities instead of horizontal technical layers.

Core rule:

> Group things that change together. Keep feature-specific behavior close. Share only what is stable, intentional, and truly cross-cutting.

## Choose the Reference

- For backend, server, full-stack, CQRS, API, modular monolith, microservice, serverless, or general application architecture, read [server-systems.md](references/server-systems.md).
- For machine learning, MLOps, model training, inference, evaluation, feature stores, model registries, batch scoring, online serving, or ML platform work, read [machine-learning-systems.md](references/machine-learning-systems.md).

## Default Workflow

1. Identify the behavior, capability, workflow, or model responsibility the user is changing.
2. Determine whether the current organization follows technical layers or cohesive slices.
3. Keep request contracts, validation, authorization, handler/use-case logic, local data access, mapping, tests, and documentation near the behavior when they change together.
4. Move only stable cross-cutting concerns into shared infrastructure or a shared kernel.
5. Avoid forcing every slice into the same implementation style. Use the simplest implementation that fits each slice.
6. Verify boundaries: one slice should not depend on another slice's private request, handler, validator, DTO, feature code, model policy, or internal test fixtures.

## Review Checklist

- Does the proposed slice represent a real capability or workflow, not a technical artifact?
- Can a developer understand and test the behavior without navigating unrelated layers?
- Are shared abstractions stable and genuinely cross-cutting?
- Is duplication local and intentional rather than prematurely abstracted?
- Are tests organized around behavior, not implementation layers?
- For ML systems, do training, inference, evaluation, model contract, monitoring, and deployment policy live with the capability when they change together?
