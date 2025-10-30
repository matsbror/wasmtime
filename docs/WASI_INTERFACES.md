# WASI Interfaces in Wasmtime

This document provides an overview of WASI (WebAssembly System Interface) implementations in Wasmtime, categorizing them by their architecture and compatibility with WASI 0.2/0.3 standards.

## Architecture Overview

Wasmtime implements WASI interfaces using two different approaches:

1. **Modern Component Model** - Uses WIT (WebAssembly Interface Types) and the Component Model
2. **Legacy WITX** - Uses WITX files with the `wiggle` binding generator (WASI Preview 1)

## Modern Component Model Interfaces (WASI 0.2/0.3 Compatible)

These interfaces use the Component Model with WIT definitions and are forward-compatible with WASI 0.2 and 0.3 standards.

### Core WASI (`wasmtime-wasi` crate)

**Location**: `crates/wasi/`

The main WASI implementation with multi-version support:

- **WASI Preview 2 (p2)**: `wasi:*@0.2.6` - Stable release
- **WASI Preview 3 (p3)**: `wasi:*@0.3.0-rc-2025-08-15` - Release candidate

**Includes**:
- `wasi:cli` - Command-line interface (stdio, environment, exit)
- `wasi:clocks` - Wall clock and monotonic clock
- `wasi:filesystem` - File and directory operations
- `wasi:random` - Random number generation
- `wasi:sockets` - TCP/UDP networking

**Features**:
```toml
p0 = ["p1"]
p1 = ["dep:wiggle", "p2"]  # Preview 1 compatibility
p2 = ["wasmtime/component-model", "wasmtime/async"]  # Preview 2
p3 = ["wasmtime/component-model-async", "wasmtime/component-model-async-bytes"]  # Preview 3
```

---

### Stable Interfaces

#### 1. **wasi-io** - `wasi:io@0.2.6`

**Location**: `crates/wasi-io/`

Foundation interface for asynchronous I/O operations.

**Provides**:
- `pollable` - Async readiness notifications
- `input-stream` - Asynchronous read operations
- `output-stream` - Asynchronous write operations
- `error` - Error handling types

**Key Features**:
- `#![no_std]` compatible
- Used as foundation by filesystem, sockets, CLI, and HTTP interfaces
- Full async/await support

**Dependencies**: None (foundation layer)

---

#### 2. **wasi-http** - `wasi:http@0.2.6`

**Location**: `crates/wasi-http/`

HTTP client and server implementation with both Preview 2 and Preview 3 support.

**Provides**:
- `wasi:http/types` - HTTP types (methods, headers, requests, responses)
- `wasi:http/handler` - Server-side handler interface
- `wasi:http/proxy` - Client-side proxy interface

**Key Features**:
- Both **p2** and **p3** support (via feature flag)
- Built on hyper and tokio
- TLS support via tokio-rustls
- Full async streaming

**Dependencies**: wasi:io, wasi:clocks

---

### Draft Interfaces (0.2.0-draft)

#### 3. **wasi-keyvalue** - `wasi:keyvalue@0.2.0-draft`

**Location**: `crates/wasi-keyvalue/`

Key-value storage abstraction for Components.

**Provides**:
- `wasi:keyvalue/store` - Basic get/set/delete operations
- `wasi:keyvalue/batch` - Batch operations
- `wasi:keyvalue/atomic` - Atomic operations
- `wasi:keyvalue/watch` - Watch for changes

**Supported Backends**:
- In-memory store (empty identifier)

**Key Features**:
- Component Model resources for buckets
- Async operations
- Trappable errors

**Official Spec**: https://github.com/WebAssembly/wasi-keyvalue

---

#### 4. **wasi-config** - `wasi:config@0.2.0-draft`

**Location**: `crates/wasi-config/`

Configuration variable access for Components.

**Provides**:
- `wasi:config/store` - Configuration retrieval

**Key Features**:
- Simple key-value configuration access
- Component Model resources
- Async operations

**Official Spec**: https://github.com/WebAssembly/wasi-config

---

#### 5. **wasi-tls** - `wasi:tls@0.2.0-draft`

**Location**: `crates/wasi-tls/`

TLS/SSL cryptographic transport layer.

**Provides**:
- `wasi:tls/types` - TLS connection types and configuration

**Key Features**:
- Built on tokio-rustls
- Integrates with wasi:io streams
- Support for client and server connections

**Dependencies**: wasi:io

---

### Custom Extensions (0.2.0-draft)

These are Wasmtime-specific extensions not part of the official WASI specification.

#### 6. **wasi-accelerator** - `wasi:accelerator@0.2.0-draft`

**Location**: `crates/wasi-accelerator/`

Hardware acceleration APIs for computational workloads.

**Provides**:
- Matrix operations (via nalgebra)
- Host-side buffer management
- Matrix multiplication offloading

**Use Case**: Offload compute-intensive operations to host

**Key Features**:
- Resource-based buffer management
- Zero-copy where possible
- F32 matrix operations

---

#### 7. **wasi-dataframe** - `wasi:dataframe@0.2.0-draft`

**Location**: `crates/wasi-dataframe/`

DataFrame operations for data analysis workloads.

**Provides**:
- DataFrame creation from rows/CSV
- JSON serialization
- Polars integration for data manipulation

**Use Case**: Data analysis and transformation in WebAssembly

**Key Features**:
- Built on Polars
- Lazy evaluation support
- CSV parsing

---

### Hybrid Interfaces

#### 8. **wasi-nn** - Dual Interface

**Location**: `crates/wasi-nn/`

Neural network inference with both legacy and modern interfaces.

**Provides**:
- Legacy: WITX-based interface (Preview 1)
- Modern: WIT-based Component Model interface (Preview 2)

**Supported Backends**:
- ONNX Runtime
- OpenVINO
- PyTorch (libtorch)
- Windows ML

**Key Features**:
- Graph (model) loading and execution
- Tensor operations
- Multiple ML backend support
- Backend selection at runtime

**Official Spec**: https://github.com/WebAssembly/wasi-nn

---

## Legacy WASI Interfaces (Preview 1)

These interfaces use the older WITX format and are compatible only with WASI Preview 1.

### 1. **wasi-common**

**Location**: `crates/wasi-common/`

WASI Preview 1 compatibility layer.

**Architecture**:
- Uses WITX files in `witx/` directory
- Uses `wiggle` macro for bindings generation
- No Component Model support

**Purpose**: Maintains compatibility with existing WASI Preview 1 modules (non-Component)

**Status**: Legacy - use `wasmtime-wasi` p1 feature for Preview 1 support with modern architecture

---

### 2. **wasi-threads**

**Location**: `crates/wasi-threads/`

Thread proposal support (experimental).

**Architecture**:
- Depends on wasi-common
- No Component Model migration yet
- Preview 1 style

**Status**: Experimental thread support, not yet migrated to Component Model

---

## Identifying Modern vs Legacy Interfaces

### Modern Component Model Interfaces Have:

1. ✅ **WIT files** in `wit/` directories (not WITX)
2. ✅ **`wasmtime::component::bindgen!`** macro in source
3. ✅ **`component-model` feature** in Cargo.toml
4. ✅ **Version tags** like `@0.2.6` or `@0.3.0-rc-*` in WIT package declarations
5. ✅ **Resource types** using Component Model patterns
6. ✅ **Async/await** support with tokio integration
7. ✅ **Trappable errors** with proper error types

### Legacy Interfaces Have:

- ❌ **WITX files** in `witx/` directories
- ❌ **`wiggle::from_witx!`** macro for bindings
- ❌ **No Component Model** features required
- ❌ **Synchronous-only** APIs

---

## Migration Path

### For WASI Preview 1 Components/Modules:
Use `wasmtime-wasi` with the `p1` feature enabled (includes wiggle-based compatibility layer).

### For WASI Preview 2/Component Model:
Use `wasmtime-wasi` with the `p2` feature (default) for stable 0.2.6 interfaces.

### For WASI Preview 3 (Future):
Use `wasmtime-wasi` with the `p3` feature for async streaming and latest 0.3.0 interfaces.

---

## Summary Statistics

**Total WASI Interfaces**: 10

**Modern Component Model**: 8 interfaces
- Core: wasmtime-wasi (p2/p3)
- Stable: wasi-io, wasi-http
- Draft: wasi-keyvalue, wasi-config, wasi-tls
- Extensions: wasi-accelerator, wasi-dataframe
- Hybrid: wasi-nn (supports both)

**Legacy Only**: 2 interfaces
- wasi-common
- wasi-threads

**Percentage Modern**: 80%

---

## Adding WASI Interfaces to Linker

### Modern Interface Example (Component Model):

```rust
use wasmtime::{Engine, Store, component::Linker};
use wasmtime_wasi::{WasiCtx, WasiCtxView, WasiView};
use wasmtime_wasi_keyvalue::{WasiKeyValue, WasiKeyValueCtx};

let mut linker = Linker::new(&engine);

// Add core WASI p2
wasmtime_wasi::p2::add_to_linker_async(&mut linker)?;

// Add wasi-keyvalue
wasmtime_wasi_keyvalue::add_to_linker(&mut linker, |ctx: &mut Ctx| {
    WasiKeyValue::new(&ctx.wasi_keyvalue_ctx, &mut ctx.table)
})?;
```

### Preview 1 Example (Legacy):

```rust
use wasmtime::Linker;
use wasi_common::WasiCtx;

let mut linker = Linker::new(&engine);
wasmtime_wasi::add_to_linker_sync(&mut linker)?;
```

---

## References

- [WASI Specification](https://github.com/WebAssembly/WASI)
- [Component Model](https://github.com/WebAssembly/component-model)
- [WIT Format](https://github.com/WebAssembly/component-model/blob/main/design/mvp/WIT.md)
- [Wasmtime Book](https://docs.wasmtime.dev/)
