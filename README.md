# RLReplayParser

A Python library (from-scratch) to parse Rocket League `.replay` files.

Status: initial / planning — no parser code implemented yet.

Goal
----
Build a clean, well-tested, and maintainable parser for Rocket League replay files (.replay) that is suitable as:
- a standalone library (core parsing logic) that can be open-sourced, and
- a backend component used by a private web app to accept user uploads and store parsed results.

Motivation (why build from scratch)
----------------------------------
Most existing libraries for parsing Rocket League replays are outdated, incomplete, or brittle across game updates. Re-implementing (or reworking) the parser from first principles gives us:
- clear, modern code with tests,
- a design that separates parsing core from application concerns (storage, web UI, auth),
- a chance to document the format and parsing approach so future contributors can understand and maintain it.

Audience and scope
------------------
This README is written for a fellow CS student / developer. You should have basic familiarity with Python and binary data concepts (bytes, endianness, structs). I will explain intermediate concepts in an accessible way.

This project focuses on:
- extracting metadata (map, duration, players, teams, scores),
- extracting and structuring gameplay events (goals, demolitions, major state changes),
- providing a clean Python API and CLI for batch processing,
- safe processing practices suitable for uploaded files.

This project does NOT (initially) aim to:
- reimplement the full game engine,
- perfectly reconstruct every in-game frame,
- provide a production webapp — the webapp will be a separate repository (private until ready).

Planned features
----------------
Planned features for the parser core (first releases):
- Read `.replay` file container and extract header + payload.
- Decompress payload if required and split into logical blocks.
- Extract basic metadata: map name, duration, timestamp, player list, teams, final score.
- Parse a useful subset of events: goals, assists, demolitions, match start/end and notable timeline events.
- Provide data models (dataclasses or pydantic models) for Metadata, Player, Event, etc.
- Expose a streaming API (generator) for events to support low-memory processing.
- Export helpers (JSON, CSV) for parsed outputs.
- CLI to parse single files, directories, and produce structured outputs.
- Test suite with anonymized sample replays.

High-level parsing approach (explained simply)
----------------------------------------------
A replay file is a binary file that stores the recorded session state and events. Parsing it typically involves these steps:

1. Read the file header
   - The header contains basic info and pointers to the rest of the file. Think of it as a table of contents.

2. Decompress / decode the main payload
   - Some replay formats compress or encode payload blocks. We must detect and decode them before deeper parsing.

3. Split payload into chunks/frames
   - The payload often contains a stream of recorded frames, network packets or serialized blocks. We split it into manageable pieces.

4. Deserialize each chunk to recover typed data
   - Binary blobs are converted to Python values. This often uses struct.unpack-like operations and custom deserializers.

5. Reconstruct semantic objects (metadata, players, events)
   - Raw values are assembled into higher-level objects (e.g., an Event object with type and timestamp).

6. Post-processing and validation
   - Validate data, compute derived fields (match duration), and optionally sanitize player-identifying info.

Important concepts (brief)
- Endianness: the byte order used to encode integers. We must read numbers with the correct endianness.
- Struct-like parsing: fixed-layout binary fields can be parsed with Python's struct module or a custom reader.
- Variable-length encoding: some fields use length prefixes; parse carefully to avoid overruns.
- Streaming vs in-memory: for large replays, a streaming parser (generator) avoids loading everything at once.
- Strict vs lenient modes: strict mode fails on unexpected data; lenient mode attempts to recover.

Design principles
-----------------
- Separation of concerns: the parsing core must not handle storage, HTTP, or auth. Expose a small, well-documented API and let the webapp use it.
- Testability: every parsing step should be unit-testable with small binary fixtures.
- Resource safety: validate file sizes and run parsing inside controlled environments (process or container) when handling untrusted uploads.
- Easy-to-read models: use dataclasses or pydantic (optional) to represent parsed objects.
- Extensibility: design to add new event types or format changes with minimal breakage.

Repository structure (planned)
------------------------------
A suggested structure for the repo (we will create these once we start coding):

- README.md
- LICENSE
- pyproject.toml / setup.cfg
- src/
  - rl_replay_parser/
    - __init__.py
    - parser.py           # main Parser class and orchestration
    - formats.py          # low-level format helpers, readers
    - models.py           # dataclasses / pydantic models (Metadata, Player, Event)
    - utils.py            # small helpers (byte reader, checksums)
    - cli.py              # CLI entry points
    - exceptions.py       # parser-specific exceptions
- tests/
  - data/                # anonymized replay samples for tests (never real personal data)
  - test_parser.py
  - test_formats.py
- examples/               # small examples and developer notes
- .gitignore
- CONTRIBUTING.md
- SECURITY.md
- .github/workflows/      # CI (tests, linters)

Development setup (how we'll start)
-----------------------------------
When we add code, a minimal dev setup will be:

1. Create a virtual environment:
   python -m venv .venv
   source .venv/bin/activate

2. Install development dependencies (placeholder list):
   pip install -e ".[dev]"

Typical dev tools to include:
- pytest (testing)
- ruff / flake8 (linting)
- black (formatting)
- mypy (optional type checks)
- pre-commit (hooks)

Testing strategy
----------------
- Unit tests for format-level readers using small binary fixtures.
- Integration tests that parse full anonymized replays and assert expected metadata and several events.
- CI runs tests on each pull request and enforces style checks.

Privacy, security and legal considerations
-----------------------------------------
- Replays can include player names, IDs, and other personal data. Treat them as personal data.
- Never commit real replays with identifiable info to a public repo. Use anonymized or synthetic samples for tests.
- If the parsed data will be collected from external users (webapp):
  - obtain clear consent and publish a privacy policy,
  - implement data retention and deletion mechanisms,
  - comply with relevant regulations (e.g., GDPR) if you have EU users.
- Do not store or publish user secrets, tokens or DB credentials in the repo; use environment variables or secret managers instead.
- Process untrusted files in an isolated environment (container, separate process) and validate file sizes and types prior to parsing.

Contributing
------------
When the repo is public, contributions will be welcome. Some guidelines we will include:
- Open issues for design or feature discussions before implementing large changes.
- Small, focused pull requests with tests.
- Follow coding style (black + ruff) and add type hints where helpful.
- Add tests for every bug fix or feature.

Roadmap (initial milestones)
----------------------------
1. Project skeleton, CI, formatting and linter config.
2. Implement low-level binary reader utilities and basic header decoding.
3. Extract metadata (map, players, duration, timestamp).
4. Implement events iterator with a few core event types.
5. Add export helpers + simple CLI.
6. Improve test coverage and harden parsing for newer replay variants.

How you (the reader) can help
-----------------------------
- If you want to contribute later: start by writing tests for a parsing piece or by researching parts of the `.replay` format.
- Report bugs or format differences with small reproducible cases (anonymized).
- Help create example anonymized test replays if you have safe samples.

Notes for next steps
--------------------
We will start by adding the repository skeleton and basic dev tooling. The first code task will be a small, well-tested utility to read binary fields and validate them against expected values from a tiny anonymized fixture. From there we will implement header parsing and build up.

Acknowledgements and references
-------------------------------
- Existing community projects and any public documentation on the `.replay` format (used as references during research; we will carefully cite any source code or docs we consult).
- General resources on binary parsing and Python tooling.

Contact
-------
Repository: https://github.com/ismaelmormor/RLReplayParser

If you prefer, open an issue in the repo to discuss design choices or tasks to start with. I will keep explanations concise and clear so you, as a second-year CS student, can follow the decisions and reasoning behind the parser design.
