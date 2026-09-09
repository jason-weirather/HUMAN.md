# Coding Style and Agent Expectations for Jason L Weirather

This document describes my preferred defaults for Python code and repository design. It is intended both as a personal style guide and as guidance for coding agents working in my repositories.

These are defaults, not sacred laws. Repository-specific requirements and explicit task instructions may override them.

## Applying this guide

* Explicit task requirements and repository-specific instructions take precedence.
* New code should follow this guide even when nearby legacy code does not. Existing code may reflect older habits, agent choices, or compromises rather than my current preferences.
* Do not refactor unrelated code merely to conform. Improve the affected structure when it obstructs the requested work.
* Treat firm preferences as defaults to preserve, and judgment calls as judgment calls. Do not turn an unanswered question or an absence of preference into a new prohibition.

The strongest distinctions here concern public APIs without aliases, ordinary compatible inputs without needless coercion, useful public workflow stages, schema-owned complex data models, non-mutating transformations, and clear failure behavior. They are not a mandate to make all code maximally strict or maximally elaborate.

### Notebook first when exploring

I normally develop a new computational idea in a notebook, checking intermediate results along the way, and then package the useful code into a library. This is a strong workflow preference, not a departure from library-first design.

For an exploratory request, start with visible, stepwise computation using existing library components where useful. Do not require a settings hierarchy, finished package structure, or complete production interface before we have tested the idea.

When the task is to implement an established idea or update an existing package, work in the library directly. Do not turn every requested patch into another notebook experiment. Once an experiment becomes library code, preserve its useful steps as public operations rather than burying them in a single entry point.

## Core philosophy

### Prefer one clear way to do something

Give each semantic concept one intentional public API and spelling. I feel strongly about avoiding aliases, not about policing equivalent Python container types.

Avoid multiple names for the same argument, several long-form CLI aliases, legacy wrappers, and duplicate public access paths to the same result. Do not make a constructor guess between unrelated semantic origins.

For example, do not publicly expose both of these as interchangeable access paths:

```python
cell.area
cell.record.area
```

Choose the appropriate public surface. A private backing record is fine; public forwarding properties are not automatically justified just because they avoid duplicating storage. A genuine difference in behavior or cost may justify an exception, but it needs an explanation rather than a blanket convenience exemption.

Use different names or explicit constructors for genuinely different origins, such as a file versus an array, not a string path versus a `Path`.

### Accept compatible inputs without needless coercion

A list, tuple, or another list-like object is fine when it supplies the behavior the operation actually needs. Accept compatible values without requiring an exact concrete class, copying them, or recasting them solely to enforce a house container type.

```python
# The same selection API accepts either sequence.
image.select_channels([0, 2, 4])
image.select_channels((0, 2, 4))
```

This does not make every iterable interchangeable or remove meaningful requirements on shape, element values, ordering, or repeatable access. Validate the requirements that matter to the operation, not incidental differences that do not affect it.

Similarly, use one path-reading API that accepts a string or a path-like object:

```python
from os import PathLike
from pathlib import Path


def read(path: str | PathLike[str]) -> Image:
    path = Path(path)
    ...
```

Do not add `read_path()` and `read_path_string()`. Converting to `Path` at entry is reasonable because it gives the implementation useful path operations. Check the filesystem requirements the operation actually needs; constructing a `Path` alone is not a guarantee that a file exists or is readable.

### Normalize only for a reason

Normalize external or vendor representations when their differences complicate the internal computation, not merely because an input crossed a boundary. Reuse an external representation when it is already the clearest or most faithful model.

A conversion should earn its place through correctness, a downstream requirement, meaningful ownership, or simpler computation. Do not automatically freeze every sequence into a tuple or eagerly materialize a lazy source.

Accepting compatible inputs is separate from retaining mutable state: copying data to own an independent value is a real reason to copy; copying only to change an otherwise suitable container type is not.

### Use explicit types when they improve the code

When a concept has meaningful rules, metadata, operations, or validation requirements, consider representing it with a class, dataclass, `NamedTuple`, enum, or another explicit type rather than passing loose dictionaries and tuples throughout the code.

Do this when the type makes the API clearer or helps organize and enforce the concept. Do not define a formal type for every temporary intermediate value.

### Build for current requirements

Build abstractions for requirements that exist now; do not add compatibility machinery, aliases, plugin systems, extension points, configuration options, or generalized abstractions solely because they might become useful later.

A small amount of structure that makes the current implementation substantially cleaner is welcome. Architecture erected around imaginary future users is not.

### Maintain one authoritative model

Keep one authoritative structured model of the information represented by a major class or process.

Generate text, JSON, HTML, notebook, and CLI presentations from that structured information rather than maintaining separate copies for each display.

Do not store important information only inside a human-readable display string when it can remain available as structured data. Generate the display string from the structured data rather than later parsing the string to recover the information.

## API evolution and compatibility

Backward compatibility is not automatically more important than the quality of the current design.

Discuss a proposed breaking public API change with me before implementing it unless I have already requested or approved that change. Show the intended interface and explain which capabilities are retained, moved, or removed. I want to keep track of how users should call the library, and I may reject a change that takes functionality away from them. Willingness to break an API is not permission to silently remove features.

Routine private refactors do not require this discussion, and an interface change I have already approved does not need another approval round.

Do not make breaking changes without a reason. However, do not retain a poor API, confusing name, redundant representation, bad abstraction, or obsolete implementation merely to avoid breaking existing callers.

During active development, especially in early development projects, prefer a clean breaking change over accumulating:

* compatibility shims
* deprecated wrapper functions
* old and new APIs that perform the same operation
* aliases for removed names
* branches supporting obsolete representations
* tests whose only purpose is preserving behavior that was intentionally replaced

When replacing an API or internal representation, remove the old form and update the affected implementation, exports, imports, tests, schemas, examples, documentation, and CLI together.

The repository should have one current truth after the change. Do not leave fragments of the former design scattered through the codebase.

Once a project explicitly promises a stable public API in its README.md, or has a significant external user base, follow the promise of the semantic versioning contract. Under semantic versioning, breaking public API changes belong in a major release, additive compatible features in a minor release, and compatible fixes in a patch release.

## Repository and package structure

Start new Python projects from a contemporary Python version unless otherwise specified.

The Python range declared by `[project].requires-python` is authoritative. Use the modern syntax and standard-library functionality available in that range.

Do not add backports, compatibility branches, or dependencies solely to support Python versions the project does not claim to support.

A normal repository should follow this structure:

```text
repository-name/
    pyproject.toml
    src/
        import_package/
            __init__.py
            ...
```

By default:

* the distribution name matches the Git repository name
* the import package matches the distribution name
* dashes in a distribution name become underscores in the Python import package

For example:

```text
repo-riposte/
    pyproject.toml
    src/
        repo_riposte/
            __init__.py
```

Any less obvious relationship between repository name, distribution name, and import package should be documented in `pyproject.toml` and the repository documentation.

Use modules and subpackages whenever they improve the logical organization of the implementation.

### Package namespaces and `__init__.py`

Keep `__init__.py` files minimal.

They are mainly used for:

* intentionally exposing public classes, functions, and variables
* arranging convenient package namespaces
* exposing package metadata such as `__version__`

Do not place substantial implementation logic in `__init__.py`.

The public namespace does not have to reproduce the physical file hierarchy exactly. A deeply implemented class may be re-exported at a higher package level when that makes the package substantially easier or more natural to use.

A re-export may establish the documented public entry point while its implementation module remains internal. Do not advertise several equivalent import paths merely because they happen to work.

### Version handling

`pyproject.toml` is the sole authored source of the package version.

Place version-resolution code in:

```text
src/<package>/_version.py
```

Expose `__version__` from the top-level import package.

When running from a source checkout or editable source tree, `_version.py` may read the expected repository-root `pyproject.toml`. Confirm that `[project].name` identifies the expected distribution before accepting its version.

When the source `pyproject.toml` is unavailable, such as in an installed wheel, obtain the version from installed distribution metadata using `importlib.metadata.version()`.

Do not maintain another hard-coded version that can silently drift away from `pyproject.toml`.

A constant fallback such as:

```python
UNKNOWN_VERSION = "0+unknown"
```

is acceptable because it explicitly communicates that no authoritative version was found. It is not a second version source.

### Naming

Use ordinary Python naming conventions: `snake_case` for modules, functions, and variables; `UpperCamelCase` for classes; a leading underscore for private implementation details; and `UPPER_CASE` for constants.

Prefer descriptive names in orchestration and public interfaces. Domain-standard abbreviations and mathematical shorthand are fine inside small, understandable calculations. I do not have a strong preference between `x, w` and `values, weights` when both versions are clear in context.

## Classes, functions, and implementation organization

### Classes for substantial concepts

Major domain concepts and substantial library APIs will often be represented as classes.

Use a class when related behavior benefits from owning:

* state
* identity
* metadata
* validation rules
* configuration
* dependencies
* a resource lifecycle
* a coherent domain concept

Use ordinary functions for stateless transformations and small independent operations.

Do not create an empty utility class merely to provide a namespace around unrelated functions. A module is generally better for that purpose.

Dataclasses and `NamedTuple` types are different. They are useful record types and may be appropriate when they substantially improve organization.

### Small classes

When a class is reasonably small, place it in a module named after the class using `snake_case`.

For example:

```text
number_calculator.py
```

contains:

```python
class NumberCalculator:
    ...
```

### Large classes

A class becoming large is itself a reason to reorganize its implementation.

Do not allow one major class to expand indefinitely inside a giant source file merely because it remains one conceptual public object.

When a class becomes large, complicated, or naturally separates into groups of implementation, create a subpackage named for the class:

```text
number_calculator/
    __init__.py
    number_calculator.py
    parsing.py
    operations.py
    validation.py
```

Keep the main public class in:

```text
number_calculator/number_calculator.py
```

Use the remaining modules for coherent supporting pieces such as:

* parsing
* validation
* serialization
* algorithms
* internal helper functions
* supporting classes
* distinct stages of a workflow

The goal is to shatter the implementation into understandable pieces without unnecessarily shattering the public concept.

### Public workflow stages

Break workflows into discrete, meaningful public steps. Those steps should be usable and inspectable independently in Python, not accessible only by running the entire workflow. Low-level implementation helpers may remain private.

An optional orchestration method can call the same public operations:

```python
def process(self) -> Result:
    source = self.read_source()
    prepared = self.prepare(source)
    return self.analyze(prepared)
```

A calling function or notebook can orchestrate those stages instead; `.process()` is optional. Useful public steps are the requirement.

### Extract helpers when the explanation needs a home

A short, clear calculation can stay where it is used:

```python
next_shape = ((height + 1) // 2, (width + 1) // 2)
```

Do not extract that expression merely to give it a name. But the threshold for a helper is not much higher: extract when a calculation becomes long, complicated, or deserving of a substantial explanation of what it does, why, or where it came from.

A short `#` comment can explain an inline calculation. A lengthy explanation belongs with a named helper and its docstring, rather than interrupting the orchestration. Reuse is not required; readability and organization are sufficient reasons to extract.

When the helper itself is substantial, move it into the relevant module's subpackage, for example `number_calculator/utilities.py`, or a more focused algorithm module. Keep those utilities local to the concept they support.

### Pass the dependencies that are actually used

If another function or class only needs a couple of properties, pass those properties rather than the entire owning object:

```python
# The calculation only needs the calibration value.
radius_px = radius_in_pixels(radius_um, image.microns_per_pixel)
```

Do not pass `image` or `self` just so the receiver can pull out `microns_per_pixel`. Pass the whole object when the receiver genuinely works with that object as an image, reader, analysis session, or other domain concept and needs its capabilities. Do not dismantle a useful domain object into dozens of arguments to satisfy this preference mechanically.

### Constructors and object origins

Keep constructors narrow enough that it is clear what state is being established.

Every accepted constructor argument should have a meaningful effect. Do not silently ignore arguments.

When an object may intentionally originate from different kinds of sources, use named constructors:

```python
Thing.from_file(...)
Thing.from_array(...)
Thing.from_report(...)
```

Do not make one constructor inspect an arbitrary object and guess which construction path the caller intended.

Different semantic origins are not aliases. They are different operations and should be represented explicitly. Compatible containers or path-like types within the same origin share the same constructor.

### Class methods

Use `@classmethod` for operations that conceptually belong to the class as a whole rather than to one existing instance.

This includes named constructors and operations that create, combine, or otherwise return instances of the class:

```python
Thing.from_file(...)
Thing.from_array(...)
Thing.concat([...])
```

When a class method constructs an instance, prefer returning `cls(...)` rather than hard-coding the class name so that inherited implementations can return the appropriate child type.

Use an instance method when the operation acts on one existing object.

Use a module-level function when the operation does not meaningfully belong to the class.

### Composition and inheritance

Prefer composition when one object simply needs to use another.

Use inheritance when:

* there is a genuine parent-child relationship
* the child is meaningfully a specialized form of the parent
* shared parent behavior prevents substantial duplication
* several implementations need to satisfy one deliberate interface

Avoid deep or decorative inheritance hierarchies.

Use `ABC` and `@abstractmethod` for intentional abstract base classes when appropriate.

A base class may define shared orchestration while requiring child classes to implement specific operations. A child should not be considered complete until it implements the required methods.

Use `NotImplementedError` for deliberate implementation hooks when abstract class machinery would not otherwise fit the design.

### Transformations return new objects

Transformations should return new objects and leave their inputs unchanged by default. This is a strong preference.

```python
cleaned = image.drop_labels(labels)  # image is unchanged
```

When mutation is genuinely useful to us or library users, an `in_place=False` option is welcome. The caller must explicitly choose `in_place=True` to mutate the object. Do not add the option mechanically to every operation, and do not invent a second family of synonymous transformation methods.

This concerns transformations of data, not operations whose purpose is to manage an object's state or resources, such as closing a reader.

### Make expensive work look like work

Use methods for expensive reads, materialization, or computation rather than hiding them behind ordinary properties:

```python
pixels = channel.asarray()  # Makes the potentially large read visible.
```

Do not also provide a `.array` property that silently performs the same full read. A cache does not make an expensive first access a cheap operation. Cheap access to already available metadata is appropriate for properties.

### Give public objects a useful text display

Objects representing meaningful public library concepts should have a useful, readable inspection surface through `print()` and/or `repr()`, rather than only an object address.

A good text summary is normally suitable for both terminals and notebooks. I do not require a special distinction between those displays or a particular tree layout. Rich notebook rendering is worthwhile for explicitly notebook-oriented interfaces, such as table viewers.

Generate the display from the object's structured information. Use available metadata rather than triggering a large computation or materializing a lazy source just to print it.

## Internal records and structured data

Use ordinary variables, dictionaries, tuples, lists, and similar structures for temporary or limited-scope data.

For reusable record-like values, use a dataclass or `NamedTuple` when it makes the code meaningfully clearer.

A frozen, slotted dataclass is appropriate when a reusable internal record should be immutable:

```python
@dataclass(frozen=True, slots=True)
class Record:
    name: str
    value: int
```

Use `NamedTuple` when tuple behavior or tuple interoperability is useful.

### JSON Schema authority

For complex external inputs, outputs, saved records, reports, API documents, or other exchanged structured data, prefer an authoritative JSON Schema.

Package schemas under:

```text
src/<package>/schemas/
```

and include them in the built distribution through `pyproject.toml`.

The JSON Schema defines the external structure.

A dataclass or domain class may be used internally to make that data easier to work with, but it is not a second independent contract. When both exist:

* the JSON Schema is authoritative for the serialized or exchanged representation
* the Python class is an implementation convenience
* do not independently invent conflicting field requirements or defaults in both places
* conversion to and from the Python representation should preserve the schema-defined structure

Do not use Pydantic for authoring our complex domain data models. This is a strong preference about where the model lives, not a claim that Pydantic is unsuitable for other developers.

Sophisticated data deserves a discoverable, documented definition readable outside the Python implementation. I do not want its fields, constraints, and defaults scattered across Python model classes throughout the codebase.

If a framework effectively requires Pydantic at an HTTP or framework boundary, confine it to that boundary. It must not become an independent authority competing with the packaged JSON Schema.

Do not introduce JSON Schema for simple function arguments or small internal structures that are clearer as ordinary typed Python values.

### When settings deserve their own object

A few ordinary parameters usually belong directly on the class or operation as named defaults, properties, or explicit setters. Three parameters are not, by themselves, a reason to invent a settings object.

Consider a schema-backed settings object when the number of parameters and the likelihood of tuning them make configuration cumbersome, or when the configuration already has an important, well-defined contract of fields, ranges, defaults, and relationships. Even a small configuration can qualify when it is central to the whole application.

Once settings are substantial enough to deserve their own object, they probably also deserve a packaged JSON Schema and documentation. The object provides ingestion and convenient access to that contract; it is not a competing data model. Apply the schema-defined defaults deliberately during ingestion rather than maintaining an unrelated set in Python.

This is a judgment call, not a parameter-count quota or a requirement that every dataclass have a schema.

### Table-shaped results should be tables

Return table-shaped analysis results as tables, usually pandas DataFrames. Do not wrap an ordinary table in a new result class merely to attach metadata already readily available from the inputs, such as their units.

Keep the information needed to interpret a table available. If a table is saved or passed beyond its original context, preserve that information alongside the output as appropriate. A genuinely composite result can still deserve an object; a table does not need one just because the surrounding library uses classes.

## Library and CLI design

### Library-first behavior

The Python library should be useful independently of the CLI.

The CLI should act as a front door to the library, not as a second implementation of the same behavior.

CLI commands should generally:

1. accept explicit inputs
2. construct or open the appropriate library objects
3. call the underlying library API
4. display or write the result

Substantial parsing, conversion, analysis, validation, and other domain behavior should live in reusable library classes and functions.

The same general principle applies to HTTP routes and notebook convenience functions. They should call reusable underlying code rather than contain their own separate implementation of the operation.

When several real backends or providers exist, a common base class or interface may organize them. Do not build a generalized backend architecture merely because a second implementation might someday exist.

### CLI conventions

Use Click. The CLI calls the library rather than maintaining separate behavior or defaults.

A single-purpose tool runs directly, without an unnecessary `run` subcommand. Give it an eager `--version` flag that prints the version and exits without requiring the operation's inputs. Use subcommands only when there are multiple real operations; then `version` can be a subcommand. Version alone does not justify turning a single-purpose tool into a command group. Reserve `-v` for verbosity, never version.

```text
segment input.ome.tif -o labels.ome.tif
segment --version

image_tool inspect input.ome.tif
image_tool convert input.tif -o output.ome.tif
image_tool version
```

Use one canonical long option name, optionally paired with a conventional short name. Do not provide several long-form synonyms. Specify output paths with named options such as `--output-path` / `-o`, and require the option when the output is required rather than making callers distinguish positional input and output paths.

Allow overwriting by default. Provide `--no-overwrite` for outputs and check for existing destinations before expensive work begins when it is selected.

Show option defaults in `--help`, and keep them consistent with the underlying library. Do not infer unrelated operations or scientific meaning from the shape or extension of a supplied path.

For long-running or sophisticated operations, keep routine stdout output minimal by default. Provide `--verbose` / `-v` for tqdm progress bars when sensible. At `-vv`, replace progress bars with granular, log-like progress details suitable for reading in logs.

## Files, streams, temporary data, and resources

Use context managers for files, temporary directories, subprocesses, readers, writers, and other resources with a controlled lifetime.

When a class owns an open resource, consider making the class itself a context manager:

```python
with Reader(path) as reader:
    ...
```

Code that opens or creates a resource should have a clear plan for closing it and cleaning up associated temporary data.

### Lazy children share the owning resource's scope

When a lazy child depends on a parent's open resource and does not independently own that resource, constrain it to the parent's lifetime. Use it inside the parent's `with` block. Do not quietly keep the reader alive, reopen it, or rely on the underlying resource happening to remain usable after closure.

```python
with Image.from_file(path) as image:
    tiles = image.tiles(layout)
    first_tile = tiles[0].copy()  # Independent data to keep after this scope.

# first_tile is usable; reading more source-backed tiles must fail clearly.
```

Extract independently owned data before leaving the scope when it needs to survive. A lazy view or disk-backed view is not automatically an independent copy.

### Own retained data, except where borrowing is the design

For objects retaining mutable in-memory data, strongly prefer owning a copy by default so later changes through the caller's reference do not change the object's state unexpectedly. This is an ownership decision, not a requirement to recast every input or copy every argument merely passed through a calculation.

Large lazy, disk-backed tools such as Tilework are an important exception: copying an enormous source would defeat their memory-efficient design. They may borrow a stable source, with the caller responsible for keeping it unchanged for the duration of use. Document ownership and lifetime rather than pretending borrowing provides independence.

### Large and temporary data

Avoid large reads, expensive initialization, network requests, or other substantial work merely because a module was imported.

When scientific files, images, or records may be large, prefer streaming, iteration, tiling, chunking, or bounded working data when those approaches fit the operation.

Do not load an entire dataset into memory when the operation can naturally proceed incrementally; use pythonic solutions such as the `yield` generator function when it helps accomplish this goal.

Temporary files and directories are appropriate when they make an operation safer or more practical.

For workflows that may create substantial temporary or cached data:

* provide a sensible default temporary location
* allow the caller to specify a temporary, working, or cache directory when useful
* clean up automatically whenever possible
* preserve user-specified output and cache locations according to the documented behavior

When a partial output would be misleading or harmful, write to a temporary destination and move or replace the final destination only after the operation succeeds.

### Caching is a deliberate performance choice

Do not cache results by default merely because a method might be called twice. An object representing `Number(3)` does not need a remembered answer for every call to `add_number(2)`.

Caching is useful when it avoids meaningful repeated work. Add it for the operation's actual performance needs, not as generic class infrastructure. A particular API may justify a cache as part of its design, but document that choice and make substantial retention, bounds, and release behavior clear.

## Errors and failure behavior

### Fail at the first failure

Stop at the first failure and propagate an informative error identifying the offending input, state, or invariant. This is a strong preference, including batch processing of independent samples and recognized invalid-input failures, not just unexpected exceptions.

Do not silently skip a failed sample, continue collecting partial successes, substitute another method, or reinterpret malformed values. Independence of the remaining work is not itself a reason to keep going.

Translate external-library or subprocess errors when that makes them more informative, preserving the cause through exception chaining where appropriate. A top-level CLI may render a readable error and exit unsuccessfully; cleanup code may catch and re-raise. Neither is permission to continue the computation.

A diagnostic task explicitly intended to collect multiple failures is a different requested behavior, not the default for batch tools. Do not add generic catch-and-continue infrastructure to ordinary domain logic.

### Trust an established validation barrier

Once the relevant contract has been validated, downstream code can normally trust it instead of repeating the same checks in each private helper.

Keep validation reusable through public or private methods when new inputs, new construction paths, or class extensions need to establish that contract. Public workflow stages should also establish any preconditions not already guaranteed by their inputs.

This is a practical default, not a rule that validation can only happen once. New data or changed state can require new validation; rechecking unchanged, already-established facts needs a reason.

## Type hints, docstrings, comments, and cleanup

### Typed signatures and readable calls

Use type annotations throughout public APIs and substantial internal code. I like the deliberate documentation provided by an annotated signature; missing annotations in my older code are not a preference to imitate.

Signatures should show expected inputs, return types, and defaults without making readers reconstruct them from prose. Describe the compatible interface actually accepted rather than annotating a single concrete container just to make the signature look stricter.

I am not a keyword-only absolutist. One obvious primary input can be positional. Use named arguments where positional meaning would be unclear; making every argument keyword-only can be useful when none is obvious.

```python
def measure(image: Image, *, radius: float, workers: int = 1) -> pd.DataFrame:
    ...
```

The purpose is readable, deliberate calls, not mechanically adding `*` to every signature.

### Docstrings should explain the contract without crowding out the code

Use docstrings for public concepts and non-obvious helpers. Explain what goes in, what comes out, and what is modified or retained, including units, ownership, or resource lifetime where those details matter.

Let type annotations do their job. Do not repeat every obvious type and default in a long parameter list unless that structure helps explain a complicated interface. A docstring has done its job when the contract is understandable; verbosity that makes nearby code harder to read is not an improvement.

I do not have a strong preference for NumPy-style versus Google-style sections. Follow the repository's convention rather than imposing a new format.

### Comprehensions and method chains are welcome

Use comprehensions, including filters and conditional expressions, when they remain readable. Do not expand a clear Python expression into a loop merely to appear explicit.

```python
names = [record.name for record in records if record.enabled]
```

I also like pandas method chains:

```python
# Mean area per sample, ordered by that mean.
summary = (
    cells.loc[cells["included"]]
    .groupby("sample_id", as_index=False)["area"]
    .mean()
    .sort_values("area")
)
```

A short `#` comment is enough for medium-complexity expressions. When a comprehension becomes difficult to follow, extract a helper with a useful docstring. When a chain gets unwieldy, break it into meaningful intermediate steps with comments. Readability is the limit, not a ban on concise Python or a contest to fit an algorithm on one line.

### Comments and cleanup

Use short comments for intent, assumptions, and non-obvious reasoning, not narration of each line. Move lengthy algorithm explanations into the helper that owns the calculation.

Remove dead code and unused imports as changes make them obsolete. Commented-out experiments can be temporary working material, but completed library code should use version control as the record of removed implementations. Do not perform unrelated mass formatting during a focused change.

## Environment choice

Prefer mamba for environment management and names ending in `_env`. Use the most modern compatible Python with minimally necessary conda-forge packages, then install most dependencies with pip.

## Testing and completion

I lean toward public-behavior and integration tests, especially small representative workflows that exercise the parts together. This is a preference about emphasis, not a prohibition on unit tests or direct tests of private helpers. My historical lack of tests is not a request for agents to write fewer of them.

Use the repository's established testing tools. Cover the behavior affected by a change, including relevant failure, schema, CLI, and resource-lifetime behavior. Keep tests aligned with the current intended design rather than preserving intentionally replaced APIs.

A simple reference implementation confined to tests can be useful for checking an optimized algorithm on tiny inputs. Hand-calculated examples, invariants, or comparison with an established implementation are also fine. I do not prescribe one approach; a test oracle for current behavior is not an obsolete second production API.

Run the relevant tests and configured checks before completing a task, and report what was actually checked.

## Scientific and sophisticated computational code

Prefer well-established foundational scientific Python libraries over more specialized dependencies whenever they can express the required computation clearly and correctly.

As a general preference:

```text
numpy / pandas
    >
scipy / scikit-learn / tifffile
    >
smaller but well-established Python libraries
    >
more specialized or eclectic Python libraries or R libraries or other scientific software
```

Do not add a specialized dependency merely because it provides a convenience wrapper around an operation that can be implemented cleanly with a major foundational library.

### Citations for standard scientific-library operations

Do not add scientific-method citations merely because code uses ordinary NumPy, pandas, SciPy, or scikit-learn functionality. A citation may instead explain **why a particular algorithm, statistic, threshold, distance measure, model, or analytical choice was selected**. Distinguish that rationale from the source of the implementation.

### Published bioinformatics and computational methods

When implementing or invoking a published bioinformatics or scientific method, cite the publication describing the method.

Before implementing it, explicitly consider four possible approaches:

1. use the authors' implementation as a library dependency
2. call an installed copy of the authors' software as an external program
3. incorporate compatible portions of the authors' source directly into the package
4. independently reimplement the published method in the package

Choose among these approaches deliberately rather than automatically reaching for another dependency.

### Large and sophisticated methods

For a long, sophisticated, or package-scale method, prefer using the established implementation rather than attempting a local reimplementation.

For Python software, prefer:

```text
library dependency
    >
external command invocation
```

when the software provides a reasonable Python API.

For software primarily implemented in R or another external environment, generally prefer invoking an installed version of the software rather than attempting to embed its implementation into the Python package.

In either case, cite the scientific source describing the method being run.

Avoid reimplementing a large mature package unless there is a compelling reason to take ownership of that amount of scientific and computational behavior.

### Moderately sophisticated published methods

For a moderately sophisticated published method, favor an independent implementation in our own package when doing so keeps the implementation understandable and avoids an unnecessary dependency.

When reimplementing a published method:

* identify and enumerate the computational steps carefully
* understand the authors' implementation as well as the published description
* reproduce the intended behavior faithfully
* preserve mathematically or scientifically meaningful edge-case behavior
* avoid making apparently harmless substitutions that change the method
* validate the reimplementation against the reference implementation or published examples whenever practical
* cite the publication or source describing the method being reimplemented

The goal is behavioral equivalence of the scientific method, not textual similarity to the authors' source code.

A direct inclusion or adaptation of the authors' source may be considered when its license permits redistribution and is compatible with the license of our package. However, licensing and provenance become substantially simpler when the method can be cleanly and independently reimplemented, so reimplementation is generally preferable to copying source for moderately sized methods.

### Small computational components

Use the established foundational-library implementation for ordinary statistical tests, distance calculations, clustering, regression, and similar computational primitives rather than recreating them locally. Cite a scientific rationale when needed, not ordinary library use for its own sake.

### General principle

Prefer owning the scientific logic that is small enough to understand, verify, and maintain.

Prefer relying on the established implementation when the method is sufficiently large or sophisticated that reimplementation would mean taking responsibility for an entire mature scientific software system.

Keep dependencies deliberate, keep citations scientifically meaningful, and make the provenance of published methods clear.

## Scientific provenance and reproducibility

Scientific outputs should make the computational process that produced them inspectable and reproducible whenever practical.

The goal is not to attach provenance machinery to every trivial operation. Record the information that would materially help another user, or a future version of ourselves, understand what software, method, parameters, and inputs produced an important result.

### Software and method provenance

When an external scientific program, library, model, or published method materially affects an output:

* record or expose the name of the software or method
* record the software version when it is meaningful and available
* preserve important parameters and non-obvious settings
* cite the scientific source when the operation implements or invokes a published method
* fail clearly when a required implementation is unavailable rather than silently substituting a different method

Do not claim greater reproducibility than the underlying software provides. If an implementation depends on nondeterministic behavior, hardware-specific behavior, or external state, document that when it matters.

### Docker and containerized scientific software

Docker is a useful way to make large or difficult scientific software dependencies reproducible without incorporating them directly into the Python environment.

Favor a container when:

* the scientific software has a complicated native or system-level dependency stack
* an official or well-maintained image exists
* reproducing the authors' intended runtime environment is preferable to rebuilding it ourselves
* the tool is naturally invoked as an external program rather than imported as a Python library

When using Docker or another container runtime for a scientific dependency:

* identify the image explicitly
* prefer a specific versioned image tag over a floating tag such as `latest`
* when exact reproducibility matters, record or pin the immutable image digest in addition to the human-readable tag
* expose the scientific software version separately when the container tag does not make it obvious
* keep the command and important arguments visible and inspectable
* mount input, output, and temporary working locations explicitly rather than hiding important state inside the container
* do not bake user data, credentials, or generated scientific outputs into reusable images
* cite the scientific software or method being executed; the container itself is an execution mechanism, not a substitute for scientific attribution

When we maintain the Dockerfile ourselves, keep it in the repository and make the scientific software versions and installation steps explicit enough that the image can be rebuilt.

A container should make execution easier to reproduce, not obscure what was actually run.

### Inputs and outputs

When identity of an important input or output matters, use a checksum or another stable identifier so that the exact file can be distinguished from files that merely share the same name.

Do not infer scientific meaning solely from a filename or extension when the file contents, metadata, explicit user input, or an authoritative reader can establish the format more reliably.

For complex scientific outputs, favor inspect and validate operations that can report important metadata and invariants in a machine-readable form.

When a workflow transforms one standardized scientific format into another, preserve relevant source metadata and record newly generated provenance without inventing unsupported metadata.

Keep the information needed to interpret and reproduce an important result accessible without making users reverse-engineer the implementation. For tables, this need not mean introducing a wrapper class.

### Parameters and defaults

Important defaults are part of the computational method.

When a default meaningfully affects a scientific result:

* make it visible in the Python API and CLI
* expose it in CLI help when applicable
* preserve it in structured provenance when results may later need to be reproduced

Avoid hidden heuristic defaults whose values depend on input shape, filename, environment, or other implicit context unless that behavior is itself an intentional and documented part of the method.

### Randomness and determinism

When stochastic behavior can materially affect a scientific result, expose a random seed or random-state parameter when the underlying implementation permits it.

Record the seed when reproducibility of a particular result matters.

Prefer deterministic ordering, serialization, and output naming when scientific meaning does not require otherwise.

### Temporary and intermediate data

Temporary working data may be used freely when it improves safety, memory usage, or interoperability with external scientific software.

When intermediates are purely implementation details, clean them up automatically when practical.

When intermediates are scientifically useful for inspection, debugging, or reproducibility, provide an explicit option to preserve them or direct them to a user-specified working directory.

Do not make undocumented intermediate files part of the public contract accidentally.

When detailed diagnostics become large, prefer retaining essential information and summary statistics by default, with explicit retention of full per-object or per-match detail when useful. Favor summaries that can be accumulated or combined when practical, rather than retaining a huge diagnostic catalogue just to summarize it later. This is a size-sensitive preference, not a reason to discard data the algorithm needs or an explicitly requested output.

## Expectations for coding agents

Read the affected code for its purpose, not as proof that every existing pattern is my style. Use my current explicit preferences over historical or agent-generated habits.

Make the smallest coherent change when the design works; reorganize the affected implementation when it obstructs the task. Keep the full affected surface consistent after an agreed change, including implementation, imports, exports, schemas, tests, examples, documentation, and CLI.

Discuss unapproved breaking public API changes first. Do not remove user capabilities under the name of simplification. Do not introduce aliases, speculative architecture, or unrelated refactors to make a patch look more complete.

Respect the phase of work: make an experiment easy to inspect in a notebook, and make an established library operation reusable through meaningful public steps.
