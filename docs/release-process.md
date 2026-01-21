# Release Process

This document defines how to cut an official AtlanticWave-SDX release from this repository.

## Definition
An AtlanticWave-SDX release is a Git tag in this repository that pins the exact versions of all required components.

## High-level steps
1. Select component commits/tags to include in the release.
2. Update this repository to pin those exact versions.
3. Validate the stack using the release artifacts (compose + images).
4. Tag the release in this repository.
5. Publish release notes and operator/developer instructions.

