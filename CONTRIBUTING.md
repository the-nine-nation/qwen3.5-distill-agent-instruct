# Contributing

Thank you for helping make fast, reliable Instruct models better.

## High-value contributions

- Reproduce an existing evaluation and report the exact environment.
- Add a deployment or quantization recipe with measured memory and latency.
- Submit a minimal tool-calling failure case, including tool schemas and expected behavior.
- Propose an evaluation that measures real instruction following, structured output, or agent execution.
- Contribute carefully licensed data with provenance and a clear cleaning description.

## Issues

For bugs or behavior reports, include:

1. model revision and serving engine;
2. hardware and relevant generation parameters;
3. the complete prompt or message sequence;
4. tool definitions, if applicable;
5. actual and expected output;
6. a minimal reproduction whenever possible.

Never include credentials, private datasets, personal information, or unsafe tool access in a reproduction.

## Pull requests

Keep each pull request focused. Explain what changed, why it matters, how it was validated, and any limitations. New benchmark claims should include raw counts or reproducible artifacts—not only a rounded headline score.

By contributing, you agree that your contribution is licensed under the Apache License 2.0.
