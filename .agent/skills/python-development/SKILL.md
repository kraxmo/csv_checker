---
name: python-development
description: >-
  Instructions for how to develop Python programs, the best practices
  and guidelines to follow. Applies to all programs written in Python.
---

# General Guidelines

Near the top of each Python file, insert a comment that contains this
text: "PYTHON SKILL APPLIED".

Write unit tests when they are beneficial. Use the unittest library
for this. In the tests, avoid mocks unless they are necessary. Prefer
end-to-end tests instead of overly pinpointed unit tests.

Create a Makefile with a 'test' target that executes all tests.

Follow PEP8 guidelines, except the maximum line length is 132 chars,
not 80.

Adhere to SOLID design principles.

Adhere to DRY design principles.

Create a virtual environment in the '.venv' folder in the project
root.

Use pyproject.toml to store dependency information.

Make it so that the program can be executed with uv .

For every program created, devise a rubric scorecard to measure how successful
the generated solution matches the original prompt. If rubric score is less than 95%,
re-evaluate and regenerate solution until either rubric score >=95% or you have
repeated the process 3 times without success.

When in doubt, ask questions. Do not assume or guess. 

# Command-Line Programs

Use the library called 'click' to define the command-line interface.

Always define help output, that can be accessed with either -h or
--help.
