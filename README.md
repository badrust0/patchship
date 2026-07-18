# Patchship

TCP file transfer for sending and receiving files between servers; clients.

## Features

- Send a file to a remote host over TCP
- Receive files on a listening port (accept/reject prompt)
- Optional secure shred after a successful send (overwrite with random data, then delete)
- Filename sanitation to reduce path-traversal risk on receive

## Requirements

- [Rust](https://www.rust-lang.org/tools/install) (edition 2024)

## Build

```bash
cargo build --release
```

The binary is written to `target/release/patchship-controls` (crate name: `patchship-controls`).

## Usage

### Receive

Start a receiver first (listens on port `8080`):

```bash
cargo run --release -- --receive --file unused --target unused
```

Or with the built binary:

```bash
./target/release/patchship-controls -r -f unused -t unused
```

When a transfer arrives, you are prompted to accept or reject it. Accepted files are saved under the sanitized basename in the current directory.

### Send

On the sending machine:

```bash
cargo run --release -- --file /path/to/file.bin --target 192.168.1.10:8080
```

Flags:

| Flag | Short | Description |
|------|-------|-------------|
| `--file` | `-f` | Path of the file to send |
| `--target` | `-t` | Receiver address (`host:port`) |
| `--shred` | `-s` | After send, overwrite the local file with random data and delete it |
| `--receive` | `-r` | Run in receive mode instead of send |

### Shred after send

```bash
./target/release/patchship-controls -f secret.txt -t 192.168.1.10:8080 --shred
```

## Protocol (brief)

1. Connect over TCP
2. Send filename length (`u16` big-endian), then filename bytes
3. Send file size (`u64` big-endian)
4. Stream file contents in 4 KiB chunks

## Project layout

| File | Role |
|------|------|
| `main.rs` | CLI entrypoint (send / receive / shred) |
| `sender.rs` | Interactive sender helper |
| `receiver.rs` | Interactive receiver helper |

## License

See repository for license details. If none is listed, all rights reserved by the author.
