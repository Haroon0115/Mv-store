# Mv-store

A small multi-version concurrency control (MVCC) store written in Rust, with no external dependencies.

mvstore keeps several versions of each row at once so that readers see a consistent snapshot while writers keep working. It is built for study and experimentation, not for production use.

Overview

The crate models the core ideas behind a snapshot-isolated storage layer.

Snapshot reads, where each transaction sees the data as it was when it began.
Deletes recorded as tombstones, so a removed row stays hidden from readers that should no longer see it.
Garbage collection that reclaims old versions once no active transaction can still need them.
A sequence watermark that reports which changes are safe for a consumer to read, even while earlier transactions are still open.
A command-line scenario driver that replays a file of operations and prints what each read and watermark query returns.

The design follows ideas discussed in public pull requests to the Turso database project, especially around low-watermark garbage collection and publication ordering. This project is independent and is not affiliated with or endorsed by Turso.

Requirements
Rust 1.75 or newer, using edition 2021.
No network access is needed to build, because there are no third-party crates.
Building
cargo build --release --offline

The scenario driver is written to target/release/mvstore.

Running a scenario

The driver takes one scenario file and writes one line to standard output for each read, watermark or publish operation in it.

./target/release/mvstore path/to/scenario.txt

A scenario is a plain text list of operations. The supported operations are begin, insert, delete, commit, abort, publish, read, checkpoint, gc, gc_incremental and watermark. See the examples folder for sample files.

Running the tests
cargo test --offline
Project layout
src/store.rs holds the version chains and the read path.
src/gc.rs holds garbage collection and the rules for which versions can be reclaimed.
src/seq.rs holds change sequencing and the publication watermark.
examples holds sample scenario files.
Design notes

Three behaviors matter most when reading the code.

A delete that has not yet reached durable storage must keep its tombstone in memory. If it were reclaimed too early, a later read would find nothing visible in memory and fall through to the older durable row, bringing back data that was deleted.
The watermark never moves past a change that a still-open transaction could yet commit. A transaction can take an early change number, stay open, and commit after others with higher numbers. A reader that simply trusts the highest number would skip that change for good.
Garbage collection only reclaims a version when no active snapshot can read it, so freeing memory never changes what any reader sees.
Status and limitations
This is a learning project. It has no persistence layer of its own, no networking and no locking beyond what the scenarios exercise.
It has not been benchmarked and makes no performance claims.
Interfaces may change without notice.
How it was built

This project was designed and written with AI assistance from Claude. I reviewed the code, built it, ran the tests and own the final result.I.
