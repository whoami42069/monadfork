# How to Build Monad Blockchain

## Quick Start for Claude Code
This guide provides step-by-step instructions to build the Monad blockchain from source using Docker.

## Source Repository
https://github.com/whoami42069/monadfork

## Prerequisites Check
```bash
# Verify Docker is running
docker --version

# Remove any existing container (if retrying)
docker rm -f monad-build 2>/dev/null || true
```

## Docker Setup

### 1. Create Ubuntu 25.04 Container
```bash
docker run -d --name monad-build \
  --memory=16g \
  --cpus=8 \
  --tmpfs /tmp:rw,size=10g \
  ubuntu:25.04 sleep infinity
```

### 2. Install Dependencies
```bash
# Install all required packages in one command
docker exec monad-build bash -c '
apt update && apt install -y \
  gcc-15 g++-15 cmake ninja-build git \
  libboost-all-dev libgtest-dev libgmock-dev \
  libarchive-dev liburing-dev libzstd-dev \
  libbrotli-dev libcap-dev libcrypto++-dev \
  libgmp-dev libtbb-dev libssl-dev \
  pkg-config python3 python3-pip \
  libcli11-dev libhugetlbfs-dev
'
```

### 3. Clone Repository
```bash
docker exec monad-build git clone https://github.com/whoami42069/monadfork /monad
```

### 4. Fix Submodules (Critical Step)
```bash
# Fix SSH URLs in main repository
docker exec monad-build bash -c '
cd /monad && \
sed -i "s|git@github.com:|https://github.com/|g" .gitmodules && \
git submodule update --init --recursive
'

# Fix submodules in silkpre (required for libff)
docker exec monad-build bash -c '
cd /monad/third_party/silkpre && \
git submodule update --init --recursive
'
```

### 5. Build
```bash
# Configure and build (takes ~8 minutes)
docker exec monad-build bash -c '
cd /monad && \
export CC=gcc-15 && \
export CXX=g++-15 && \
./scripts/configure.sh && \
./scripts/build.sh
'
```

## Verify Build Success
```bash
# Check if binary exists
docker exec monad-build ls -la /monad/build/cmd/monad

# Expected output: File should be ~160MB
```

## Build Output
- **Binary Location**: `/monad/build/cmd/monad`
- **Build Time**: ~8 minutes
- **Files Compiled**: 641
- **Binary Size**: 160MB

## Testing (Optional)
```bash
# Run VM tests (383 tests, takes ~2 minutes)
docker exec monad-build /monad/build/test/unittests/unit_test

# Run execution tests
docker exec monad-build /monad/build/test/unittests/execution_test

# Run EVM tests (11 tests)
docker exec monad-build /monad/build/test/unittests/evm_test
```

## Common Issues & Solutions

### Issue 1: Missing Dependencies
If build fails with missing headers, ensure all dependencies installed:
```bash
docker exec monad-build apt install -y libcli11-dev libhugetlbfs-dev
```

### Issue 2: Submodule Errors
If git submodule fails, manually fix SSH URLs:
```bash
docker exec monad-build bash -c '
cd /monad && \
git config --global url."https://github.com/".insteadOf git@github.com:
'
```

### Issue 3: Out of Memory
If build fails with memory errors, increase Docker memory allocation or reduce parallel jobs:
```bash
docker exec monad-build bash -c '
cd /monad/build && \
ninja -j4  # Reduce from default
'
```

## System Requirements
- **CPU**: 8+ cores recommended (4 minimum)
- **RAM**: 16GB minimum (build uses ~12GB peak)
- **Storage**: 50GB free space
- **Docker**: With WSL2 backend on Windows
- **OS**: Any OS that supports Docker

## Important Notes
- Ubuntu 25.04 specifically required for GCC 15 support
- Build uses AVX2 optimizations (`-march=haswell`)
- All 383 VM tests should pass
- Binary size should be approximately 160MB
- Total build time: ~8-10 minutes on 8-core machine

## Success Indicators
✅ 641 files compiled successfully
✅ Binary created at `/monad/build/cmd/monad`
✅ Binary size ~160MB
✅ No compilation errors in output