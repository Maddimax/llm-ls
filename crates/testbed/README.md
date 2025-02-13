# testbed

testbed is a framework to evaluate the efficiency of the completions generated by llm-ls and the underlying model.

It works by first making holes in files, then generates completions for a given list of repositories and finally runs the associated unit tests.

The result is a table containing a line for each repository and the total with the average percentage of successful unit tests.

Here is a simplified pseudo code algorithm for testbed:
```
read the repositories file
read the holes file(s)
for each repository
  spawn a thread
  setup the repository
  for each hole
    make the hole as specified by the file
    generate completions
    build the code
    run the tests
print results
```

## Running testbed

Before running testbed you will need to create a repositories file. It is a YAML file containing a list of repositories to test.

It also contains the parameters to the `llm-ls/getCompletions` request.

Repositories can either be sourced from your local storage or Github.

You can check the repositories files at the root of the crate to see the full structure.

### Generating holes

Before running testbed, you will need to generate a holes file for each repository. To generate a holes file run testbed with the `-g` option. You can specify the number of holes to make with `-n <number>`. It will take the list of repositories in your YAML file and create the associated files at the defined path.

### Setup

testbed runs completions for each repository in parallel. It will first create a temporary directory, then copy or download the repository's source files to that location and finally run the setup commands.

Setup commands are useful to install dependencies.

```yaml
setup_commands:
  - ["python3", ["-m", "venv", "huggingface_hub-venv"]]
  - ["huggingface_hub-venv/bin/python3", ["-m", "pip", "install", ".[dev]"]]
```

### Build

Before running the tests, testbed will run a build command to check if the code is valid.

To configure the commands, you can do the following:

```yaml
build_command: huggingface_hub-venv/bin/python3
build_args: ["-m", "compileall", "-q", "."]
```

### Runners

testbed supports two test runners at the moment:
- cargo
- pytest

To configure your runner, you have the following options:
```yaml
runner: pytest
runner_command: huggingface_hub-venv/bin/python3
runner_extra_args:
  - "-k"
  - "_utils_ and not _utils_cache and not _utils_http and not paginate and not git"
```

You can override the runners command with `runner_command`, which is useful when setting up dependencies in a venv.

## References

testbed was inspired by [human-eval](https://github.com/openai/human-eval) and [RepoEval](https://arxiv.org/abs/2303.12570).
