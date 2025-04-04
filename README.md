# loopflow

**loopflow** is a workflow system for a graphs of structured llm requests for codegen.

## How it works

loopflow has two primary interfaces:
* the *loopflow.md*, which defines the prompt(s) sent to llms by defining Goals, Outputs, and other metadata
* the *loopflow CLI*, which executes commands.

loopflow is built around "mates": LLMs with pre-defined system prompts to act in specific roles (currently: infrastructure engineer, research scientist). Commands can be run either against a specific mate or against a whole "team" of "mates", in which multiple LLMs handle requests and those responses are concatenated.

### Commands

The loopflow commands:
- **Clarify**: A suite of LLMs with different system prompts all receieve a request to ask clarifying questions
- **Draft**: An LLM or team of LLMs will draft the specified output files to meet the specified goals.
  - *Team Mode* (longer): Each mate generates a draft, reviews others' work, and one synthesizes the final result
  - *Single Mate Mode* (faster): One mate directly drafts the file without review or synthesis
- **Review**: A current drafts of the output files and provides feed.
- **Rebase**: Loopflow is built to update files in-place and auto-commit in git so that git diff shows the changes in the output files
made by the LLMs responding to loopflow requests. Rebase rebases to the latest non-loopflow commit.

To invoke:
```bash
# Initialize a new loopflow project
loopflow init [project_dir]
# Generate questions to clarify requirements
loopflow clarify [project_dir]
# Draft files (uses full team by default with a draft->review->synthesize subpipeline)
loopflow draft [project_dir] [--mate <mate_name>]
# Review existing files and append feedback
loopflow review [project_dir]
# Reset git history to before loopflow checkpoints
loopflow rebase [project_dir]
```

### Team Members

loopflow is built around the general idea of "mates". Many different mates could be defined for different contexts.

The following two are the defaults currently available:
- maya: infrastructure engineer at a large tech company, focused on simplicity and robustness
- merlin: ML researcher specializing in novel architectures and mathematical abstractions

You can add mates in `templates/mates`; simply creating the file is sufficient for it be available.

### `code-context`: Smart Context Management

Key to loopflow is submitting the right context, i.e. subset of the codebase as a reference. This is
a part of how loopflow works, but installing loopflow also installs a `code-context` binary that is
used to copy codebase subsets to the clipboard or other output streams.

code-context is a spiritual descendent of [files-to-prompt](https://github.com/simonw/files-to-prompt), 
a tool for specifying and formatting a subset of a codebase to submit as context to an LLM.

code-context builds on files-to-prompt with additional features:
- Automatic detection and prioritization of parent READMEs
- Smart path resolution based on common codebase structures

Perhaps the biggest area of current development is exploring how to best filter and compress large codebases. 
Right now the burden is on the user to identify the right subset via the code-context tool.
I believe that empowering the user to quickly express intent is essential, but it is not enough.
Work in ideas like summarization or indexing or RAG are on-going.

## Installation

This is still very early in development. Contact jack@loopflow.studio with any questions.

Requires Python 3.11+
```bash
pip install -e .
```

You must set `ANTHROPIC_API_KEY` and/or `OPENAI_API_KEY` in your environment to use those providers.

## Usage

### Structured Code Generation

Create a prompt file defining your requirements:

```markdown
# Loopflow LLM Implementation

## Goal
Implement the LLM API for loopflow, a tool for generating code with teams of LLMs.

## Output
loopflow/llm/llm.py
loopflow/llm/anthropic.py
loopflow/tests/test_llm.py

## Context
loopflow/

## Team
maya
merlin
```

Run the workflow:
```bash
# Clarify requirements
loopflow clarify

# Draft with the full team (default)
loopflow draft

# Or draft with a specific mate
loopflow draft --mate maya
```

### Context Management

Copy specific paths to clipboard (OS X):
```bash
code-context -p myproject/src,myproject/tests
```

Filter to specific file types:
```bash
code-context -e .py -e .js myproject
```

Generate raw output:
```bash
code-context -r myproject
```

#### Path Resolution

The code-context tool uses smart defaults for finding files:

- Paths are relative to $CODE_CONTEXT_ROOT (default: ~/src)
- Direct paths are tried first: "myproj/env" -> "$CODE_CONTEXT_ROOT/myproj/env"
- Auto-prefixed paths are used as fallback: "myproj/env" -> "$CODE_CONTEXT_ROOT/myproj/myproj/env"
- Test directories are always at "myproj/tests"

## License

MIT