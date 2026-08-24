# Jason Weirather Coding Defaults

These are the preferences most likely to differ from a coding agent's normal defaults. Repository-specific instructions still take precedence.

## One clear API

- Give each concept one canonical representation, spelling, and API.
- Do not add aliases, fuzzy inputs, compatibility wrappers, or multiple equivalent ways to do the same thing.
- Do not guess between unrelated input types or infer scientific meaning from filenames when the caller or file contents can be explicit.
- Reject violated assumptions with informative errors.

## Prefer the better design over legacy compatibility

- During active development, a clean breaking change is preferable to preserving a bad API or obsolete implementation.
- When replacing something, remove the old path rather than supporting old and new forms indefinitely.
- Update the complete affected surface together: implementation, imports, exports, schemas, tests, examples, documentation, and CLI.
- Once a project explicitly promises a stable API, honor its semantic-versioning contract.

## Organize around useful domain objects

- Favor classes for substantial concepts that own state, metadata, validation, configuration, or a resource lifecycle; use functions for small stateless transformations.
- A small class normally lives in `class_name.py`.
- When a class becomes large, keep the coherent public class but shatter its implementation into a `class_name/` subpackage with focused supporting modules.
- Keep high-level orchestration readable as explicit, well-named steps.
- Use named `@classmethod` constructors such as `Thing.from_file(...)` for genuinely different origins, and class methods such as `Thing.concat(...)` for operations that belong to the class as a whole.
- Use `ABC` / `@abstractmethod` when child classes must implement a deliberate interface.

## Keep one authoritative structured model

- For complex exchanged data, use packaged JSON Schema as the authoritative contract.
- Dataclasses and domain classes may make schema-defined data easier to use internally, but they do not become a competing authority.
- Do not use Pydantic as the default authority; confine it to framework boundaries when unavoidable.
- Generate text, JSON, HTML, notebook, or CLI presentations from one structured model rather than maintaining parallel representations.

## Keep the library real

- The Python library should be useful independently of its CLI, HTTP routes, or notebook helpers.
- Use Click for CLIs. Prefer explicit subcommands/options, one canonical long option name, and show defaults in `--help`.
- CLI behavior should call the underlying library rather than reimplement it.

## Handle large data and resources deliberately

- Use context managers for readers, writers, temporary resources, subprocesses, and classes that own open resources.
- Prefer streaming, generators, iteration, tiling, and chunking when data need not be loaded entirely into memory.
- Allow callers to choose working/cache directories when substantial temporary data may be produced.
- Clean temporary state when practical and write transactionally when partial output would be misleading.

## Scientific computation

Prefer the most basic well-established implementation that cleanly performs the job:

```text
numpy / pandas
    >
scipy / scikit-learn / tifffile
    >
smaller well-established Python libraries
    >
specialized or eclectic Python/R/external scientific software
```

For a published scientific method, deliberately choose among:

1. use the authors' library
2. invoke their installed software
3. include compatible licensed source
4. independently reimplement the method

For a large sophisticated Python method, favor the established library. For a large R or external tool, favor invoking the installed implementation. For a moderately sized method, favor a careful independent reimplementation when it remains understandable and testable. For ordinary computational primitives, use NumPy/SciPy/scikit-learn rather than recreating them.

Cite published scientific methods that are implemented or invoked. Do not add scientific-method citations merely for ordinary NumPy, pandas, SciPy, scikit-learn, or similar library operations. A paper may instead be cited as the scientific reason for choosing a particular method.

## Scientific provenance

Important scientific outputs should not be black boxes. When practical, expose or record the method/software and version, important parameters, and identifying information for important inputs.

For Dockerized scientific software, use explicit versioned images rather than floating `latest` tags, preserve the actual scientific software version, keep commands and mounts inspectable, and use an immutable digest when exact reproducibility matters.

Prefer deterministic behavior when scientific meaning does not require otherwise, and expose/record random seeds when stochastic behavior materially affects results.
