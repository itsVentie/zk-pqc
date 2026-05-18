**ZK-PQC Core** is a cryptographic service that ensures digital sovereignty by combining post-quantum cryptography (PQC) on lattices with zero-knowledge proofs.

The architecture is designed with a focus on hardcore privacy, cryptographic integrity, and maximum performance, utilizing a multi-tier technology stack: **Go** for high-load network interaction, **Rust** for secure memory management and cryptographic primitives, and **C/Assembly** (AVX-512) for hardware acceleration of lattice mathematics.

The Project has only started. The basic code will appear soon.

---

## 1. Architecture and Components

The project is organized as a single repository divided into isolated layers.

### Multi-layer stack

* **Layer 1: Hardware and Mathematics (`sys`)**
Low-level implementation of number-theoretic transformation (NTT) and discrete Gaussian samplers in C using vectorization (AVX-512).
* **Layer 2: Cryptographic Core (`core`)**
Safe wrappers in Rust. Key management, proof generation (Prover), and validation logic (Verifier). Exported to C-ABI via FFI.
* **Layer 3: ZKP Circuits (`circuits`)**
Mathematical circuits in Circom for verifying logic (e.g., age or identity verification) without revealing the data itself.
* **Layer 4: Transport and Business Logic (`service`)**
A network daemon in Go that manages gRPC/HTTP requests, access to the PostgreSQL database, and communication with the core via CGO.

---

## 2. Cryptographic Basis

Post-quantum security is based on the mathematics of cryptographic lattices, specifically the Ring Learning with Errors (RLWE) problem. All operations take place in the polynomial factor ring $R_q = \mathbb{Z}_q[X]/(X^n + 1)$.

Public key generation and encryption rely on the matrix-vector equation:


$$ \mathbf{t} = \mathbf{A}\mathbf{s} + \mathbf{e} \pmod q $$


Where $\mathbf{A}$ is the public polynomial matrix, $\mathbf{s}$ is the secret vector, and $\mathbf{e}$ is the noise vector.

Fast polynomial multiplication (NTT) is performed in $\mathcal{O}(n \log n)$ time with direct translation into 512-bit processor registers.

---

## 3. Environment Requirements

To build and run the node, the following must be installed on the host machine:

* **Go** 1.22+
* **Rust** (nightly toolchain) + Cargo
* **GCC / Clang** (with support for the `-mavx512f` and `-mavx512dq` flags)
* **Node.js** 20+ and the `npm` package manager
* **Circom** 2.1.6+
* **Docker** & **Docker Compose**
* A processor with support for the AVX-512 instruction set (or an emulator for testing)

---

## 4. Building the project

The build process proceeds “bottom-up”—from compiling the C code and Rust library to running the Go server.

### Initializing the workspace

```bash
cd zk-pqc-project
go work init ./service

```

### Building the cryptographic core (C + Rust)

```bash
cd core
cargo build --release

```

### Compiling ZK circuits

```bash
cd circuits
npm install
circom age_verification.circom --r1cs --wasm --sym -c

```

### Running the microservice (Go)

```bash
cd service
CGO_ENABLED=1 go run cmd/server/main.go

```

---

## 5. Using Docker

A multi-stage build is used for complete isolation and fast deployment.

```bash
docker-compose up --build -d

```

---

## 6. Directory Structure

```text
zk-pqc-project/
├── api/
│   └── v1/
│       ├── prover.proto
│       ├── types.proto
│       └── verifier.proto
├── circuits/
│   ├── common/
│   ├── inputs/
│   ├── scripts/
│   └── tests/
├── core/
│   ├── benches/
│   ├── src/
│   │   ├── ffi/
│   │   └── math/
│   └── tests/
├── scripts/
├── service/
│   ├── cmd/
│   └── internal/
│       ├── config/
│       ├── crypto/
│       ├── domain/
│       ├── repository/
│       └── transport/
└── sys/
    ├── include/
    ├── src/
    └── tests/

```

---

## 7. API and Integration

The service provides gRPC and HTTP/REST interfaces. By default, routes are served on port `8080` (HTTP) and `50051` (gRPC).

```bash
curl -X POST http://localhost:8080/api/v1/verify \
  -H “Content-Type: application/json” \
  -d ‘{“proof”: “base64_encoded_zk_proof”, “public_inputs”: [‘hash_1’, “timestamp”]}’

```

---

## 8. Security and Digital Sovereignty

This code implements the principles of end-to-end encryption (E2EE) and zero-trust.
The service layer (`service`) operates exclusively with public keys, hashes, and cryptographic proofs. The PostgreSQL database is designed without tables for storing user data in plaintext. Private vectors $\mathbf{s}$ and Gaussian noise exist only in the isolated memory of the client-side process or within secure enclaves during proof generation.

Translated with DeepL.com (free version)
