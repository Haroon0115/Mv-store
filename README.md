# Mv-store
A zero-dependency Rust implementation of a multi-version concurrency control store. It supports snapshot reads, deletes through tombstones, garbage collection of old versions, and a publication watermark that tells readers which changes are safe to consume. It comes with a command-line scenario driver and a test harness that pins its public API.
