# Jason L Weirather Coding Defaults

These are my personal defaults, not a universal Python style guide. Explicit task requirements and repository-specific instructions take precedence. `CODING_STYLE.md` is the fuller reference; do not turn its judgment calls into blanket prohibitions or assume existing agent-generated code is my preferred style.

## One clear API, not one mandatory container type

- Give each semantic operation one public name and access path. No aliases, compatibility wrappers, or equivalent public shortcuts such as both `cell.area` and `cell.record.area`. Private backing records are fine; a genuinely different behavior or cost needs an explicit justification for any exception.
- Accept compatible list-like inputs and string/path-like paths through the same API. Do not enforce exact list/tuple types or recast inputs without a concrete reason. Converting paths to `Path` for useful path operations is reasonable; separate `read_path()` and `read_path_string()` methods are not.
- Preserve meaningful shape, value, and ordering requirements. Do not guess between unrelated semantic inputs or infer scientific meaning from filenames. Compatibility is not permission to reinterpret malformed values.
- Trust established validation barriers downstream. Keep validation reusable for new inputs, construction paths, and extensions rather than repeating unchanged checks everywhere.

## Explore first; discuss breaking APIs

- For new computational ideas, start with stepwise notebook experiments and inspect intermediate results before packaging them into a library. For an established implementation or package patch, work directly in the library.
- Discuss breaking public API changes before implementing them unless already requested or approved. Explain the new interface and any capabilities being moved or removed. I accept clean breaks during active development, but not silent loss of user features.
- After agreement, replace the old API rather than keeping compatibility aliases. Update the entire affected surface: implementation, imports, exports, schemas, tests, examples, documentation, and CLI. Do not refactor unrelated code.
- Honor a project's stable-API and semantic-versioning promises when it has them.

## Useful domain objects with public stages

- Favor classes for substantial concepts owning state, metadata, validation, configuration, or resources. Use functions for small stateless operations, not empty utility classes.
- A small class lives in `class_name.py`. Shatter a large class's implementation into a `class_name/` subpackage with focused support modules while keeping the coherent public class.
- A tiny clear calculation can stay inline. Extract a helper when length, complexity, or a substantial explanation warrants it, even without reuse. Put substantial helpers in a local `utilities.py` or focused module within that subpackage.
- Pass individual properties when they are all the receiver needs. Pass the whole object when the receiver genuinely uses its domain capabilities, not just to retrieve one value.
- Make meaningful workflow stages public and independently usable. `.process()` is optional; it or a notebook should orchestrate the same stages. Low-level helpers may remain private.
- Use named `@classmethod` constructors for genuinely different origins and class methods for class-wide operations such as `Thing.concat(...)`. Use `ABC` / `@abstractmethod` for deliberate required interfaces.

## Visible work and deliberate ownership

- Transformations return new objects without changing inputs by default. Add `in_place=False` when mutation is useful; do not silently mutate or manufacture parallel synonymous methods.
- Prefer owning a copy of retained mutable in-memory data. Lazy, disk-backed tools such as Tilework are an exception: borrow stable sources to avoid enormous copies, with callers responsible for keeping them unchanged.
- Use context managers for owned resources. Lazy children dependent on a parent must stay within its resource lifetime; copy out independent data before leaving. Do not silently extend or reopen the parent's resource.
- Expensive reads and computation should look like method calls, not ordinary properties. Give meaningful public objects a useful text `print()` / `repr()` surface without triggering heavy work; a separate notebook display is mainly useful for notebook-specific interfaces such as table viewers.
- Do not cache by default merely because calls can repeat. Use caching for a real performance need, with deliberate retention and release behavior. Stream, tile, or chunk large data; expose substantial working/cache locations and clean temporary state. Write transactionally when partial outputs would mislead.

## Structured data lives outside implementation code

- Complex data contracts belong in packaged, documented JSON Schema under `src/<package>/schemas/`. Dataclasses or domain classes may support ingestion and access, but must not compete with the schema's fields, constraints, or defaults.
- Do not author domain data models in Pydantic. Keep unavoidable framework-boundary use confined there; I want a discoverable data model readable outside Python, not definitions scattered throughout code.
- A few settings normally stay as ordinary class/operation defaults. Numerous frequently tuned controls or an important tightly defined configuration may deserve a settings object and a schema together. There is no fixed parameter-count threshold, and small internal records do not all need schemas.
- Table-shaped results should normally be DataFrames, not wrapper objects when interpretive metadata is already available from inputs. Generate presentations from the same structured information. When diagnostics become large, prefer essential information and summaries that can be accumulated, with detailed retention explicit; keep required outputs and data needed by the algorithm.

## Readable Python, not expanded Python

- Use annotated signatures to document expected inputs and returns. An obvious primary input can be positional; use keyword-only arguments where names improve clarity, including all-keyword signatures when useful. Do not enforce keyword-only mechanically.
- Comprehensions, conditional expressions, pandas method chains, and locally clear mathematical shorthand are welcome. Add a short comment for medium complexity; use helpers or named intermediate steps when it becomes hard to follow.
- Keep docstrings useful and compact: inputs, outputs, mutation, and relevant semantics. Do not repeat obvious types/defaults until the documentation crowds out the code. Follow the repository's docstring format.
- Stop at the first failure, including independent batch items, and propagate an informative error. Do not default to skip-and-continue or partial-success reports. Explicitly requested failure-collecting diagnostics are a different task.
- Lean toward public-behavior and integration tests; unit tests and test-only reference implementations are welcome. Run relevant checks and report what was actually tested.

## Library and CLI

- Keep the library independently useful. CLI commands, HTTP routes, and notebook helpers call it rather than reimplementing it.
- Use Click. Single-purpose tools run directly with an eager `--version` flag, no redundant `run` subcommand. Multiple real operations use subcommands and may include `version`; version alone does not justify a command group. Never use `-v` for version.
- Use one canonical long option with an optional conventional short form. Required output paths use required named options such as `--output-path` / `-o`. Show defaults in help and keep them aligned with the library.
- Allow overwrites by default; `--no-overwrite` checks destinations before expensive work. Keep routine stdout minimal; `--verbose` / `-v` provides tqdm progress bars where useful, and `-vv` replaces bars with granular log-style progress.

## Scientific computation and provenance

Prefer foundational implementations: NumPy/pandas, then SciPy/scikit-learn/tifffile, then smaller established libraries, then specialized Python/R/external software.

For a published method, deliberately choose among using the authors' library, invoking their installed software, including compatible licensed source, and independently reimplementing it. Favor established libraries for large Python methods and installed external programs for large R/external methods. Favor careful independent implementations of moderately sized methods when understandable and testable; use foundational libraries for ordinary primitives.

Cite published methods that are implemented or invoked, not ordinary scientific-library operations merely for using a library. A citation may instead explain the scientific reason for selecting a method.

Expose or record meaningful method/software versions, important parameters, and identifying information for important inputs. Prefer deterministic behavior where appropriate; expose/record consequential random seeds. For Dockerized science, use explicit versioned images, retain the actual software version, keep commands and mounts inspectable, and use immutable digests when exact reproducibility matters.

## Environment

Prefer mamba environments named with an `_env` suffix, a modern compatible Python, minimally necessary conda-forge packages, and pip for most dependencies. Respect the project's declared Python range rather than adding unpromised compatibility.
