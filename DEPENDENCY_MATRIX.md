# Slimcoin Dependency Matrix

## Current Dependencies Analysis

### Overview

Based on analysis of `src/makefile.unix` and `.travis.yml`, here are Slimcoin's dependencies:

---

## Dependency Details

### 1. OpenSSL (CRYPTOGRAPHY)

**Status:** 🔴 CRITICAL

| Property | Value |
|----------|-------|
| **Current Version** | 1.0.x or 1.0.2 (estimated) |
| **Release Date** | 2012-2015 |
| **End of Life** | January 2016 (10+ years ago) |
| **Usage** | SSL/TLS, key generation, signing |
| **Vulnerabilities** | CVE-2014-0160 (Heartbleed) + 100+ others |
| **Severity** | CRITICAL - Security |

**Upgrade Path:**
```
1.0.2 (EOL 2019)
    ↓
1.1.1 (LTS until Sept 2026)
    ↓
3.0+ (Latest, LTS until 2028)
```

**Recommended:** Upgrade to OpenSSL 1.1.1 in Phase 1, plan 3.0 for later

**API Changes:** Minimal if staying on 1.x series

---

### 2. Boost (SYSTEM LIBRARIES)

**Status:** ⚠️ HIGH PRIORITY

| Property | Value |
|----------|-------|
| **Current Version** | 1.55 (estimated, 12+ years old) |
| **Release Date** | 2013 |
| **End of Life** | Not formally maintained |
| **Usage** | Threading, filesystem, program_options, unit_test |
| **Vulnerabilities** | Potential issues in old version |
| **Severity** | MEDIUM - Performance & Security |

**Upgrade Path:**
```
1.55 (2013)
    ↓
1.70 (2018) - Good compatibility
    ↓
1.80+ (2022+) - Latest
```

**Recommended:** Upgrade to 1.70 in Phase 1

**API Changes:** Minor, mostly compatible

**Libraries Used:**
- `boost_system` - System calls
- `boost_filesystem` - File operations
- `boost_program_options` - CLI arguments
- `boost_thread` - Threading
- `boost_unit_test_framework` - Testing

---

### 3. BerkeleyDB (DATABASE)

**Status:** ⚠️ MEDIUM - Deprecated but functional

| Property | Value |
|----------|-------|
| **Current Version** | 4.8 or 5.x (estimated) |
| **Last Update** | 2013 (BerkeleyDB 5.3) |
| **End of Life** | Oracle no longer maintains |
| **Usage** | Wallet storage, blockchain data |
| **Alternatives** | LevelDB, RocksDB |
| **Severity** | MEDIUM - Functionality |

**Upgrade Path:**
```
4.8 or 5.x (Current)
    ↓
Phase 1: Keep compatible
    ↓
Phase 6: Evaluate LevelDB
    ↓
Future: Migrate if needed
```

**Recommendation:**
- Phase 1: Test current version
- Phase 6: Plan LevelDB migration
- Not urgent: Wallet still works

---

### 4. Qt Framework (GUI)

**Status:** ⚠️ MEDIUM - OLD but optional

| Property | Value |
|----------|-------|
| **Current Version** | 5.5 (2015) |
| **Release Date** | 2015 |
| **Current LTS** | 5.15 (support until 2023) |
| **Latest** | 6.x (2020+) |
| **Usage** | GUI for wallet |
| **Severity** | MEDIUM - GUI only |

**Upgrade Path:**
```
5.5 (2015)
    ↓
5.15 LTS (2020) - Recommended for now
    ↓
6.x (2020+) - Long-term
```

**Recommendation:**
- Phase 1: Optional, keep as-is if CLI focus
- Phase 1: Update to 5.15 if GUI is priority
- Future: Plan Qt 6 migration

**Impact:** GUI only, doesn't affect daemon or consensus

---

### 5. miniupnpc (UPnP SUPPORT)

**Status:** ✅ OPTIONAL

| Property | Value |
|----------|-------|
| **Current Version** | Likely 1.x |
| **Usage** | Automatic port forwarding |
| **Severity** | LOW - Optional |
| **Recommendation** | Update to 2.x if supported |

**Build Flag:** `USE_UPNP=1` (optional)

---

### 6. zlib (COMPRESSION)

**Status:** ✅ STANDARD

| Property | Value |
|----------|-------|
| **Current Version** | System default (usually 1.2.x) |
| **Usage** | Data compression |
| **Severity** | LOW - Well maintained |
| **Recommendation** | Use system version |

---

### 7. Standard C Library (pthread, dl, etc.)

**Status:** ✅ SYSTEM

| Property | Value |
|----------|-------|
| **pthread** | POSIX threading (system provided) |
| **dl** | Dynamic linking (system provided) |
| **rt** | Real-time library (system provided) |
| **Severity** | NONE - System libraries |

---

## Platform Dependency Versions

### Ubuntu 14.04 LTS (Current in Travis)

```
OpenSSL:    1.0.1f (EOL 2016)
Boost:      1.54-1.55
BerkeleyDB: 5.x
Qt:         5.5 (via PPA)
GCC:        4.8.2
C++ Std:    C++11
```

**Issues:**
- 🔴 OpenSSL is 10+ years EOL
- ⚠️ GCC 4.8 is very old
- ⚠️ Boost is very old

### Ubuntu 20.04 LTS (Recommended)

```
OpenSSL:    1.1.1 (LTS until 2026)
Boost:      1.71+
BerkeleyDB: 5.3+
Qt:         5.12-5.15
GCC:        9.x
C++ Std:    C++14/C++17 capable
```

**Benefits:**
- ✅ Modern, supported, secure
- ✅ Better toolchain
- ✅ Better performance
- ✅ Supported until 2025

---

## Upgrade Priority Matrix

| Dependency | Current | Recommended | Phase | Priority | Risk |
|------------|---------|-------------|-------|----------|------|
| OpenSSL | 1.0.x | 1.1.1 | 1 | CRITICAL | MEDIUM |
| Boost | 1.55 | 1.70 | 1 | HIGH | LOW |
| BerkeleyDB | 5.x | 5.3+ | 1 | MEDIUM | LOW |
| Qt | 5.5 | 5.15 | 2 | MEDIUM | LOW |
| miniupnpc | 1.x | 2.x | 2 | LOW | LOW |
| zlib | System | System | - | NONE | NONE |

---

## Build Instructions by Dependency

### Installing on Ubuntu 20.04

```bash
# All essential dependencies
sudo apt-get install -y \
  build-essential \
  libboost-all-dev \
  libdb++-dev \
  libssl-dev \
  libminiupnpc-dev \
  zlib1g-dev \
  qt5-qmake \
  qt5-default

# Then build
cd src
make -f makefile.unix -j $(nproc) slimcoind
```

### Installing on macOS

```bash
brew install boost openssl berkeley-db@4 miniupnpc zlib qt5

# Build
cd src
make -f makefile.unix -j $(nproc) slimcoind
```

---

## Vulnerability Status

### Known Issues

| Component | Vulnerability | Severity | Fixed In | Action |
|-----------|----------------|----------|----------|--------|
| OpenSSL | Heartbleed (CVE-2014-0160) | CRITICAL | 1.0.1g+ / 1.0.2+ | URGENT |
| OpenSSL | Multiple DoS | HIGH | 1.1.1+ | URGENT |
| Boost | Various (old) | MEDIUM | 1.70+ | Phase 1 |
| Qt | Various (old) | MEDIUM | 5.15+ | Phase 2 |

---

## Compatibility Check List

- [ ] OpenSSL 1.1.1 with current code
- [ ] Boost 1.70 with current code
- [ ] BerkeleyDB 5.3 with wallet data
- [ ] Qt 5.15 with GUI
- [ ] GCC 9.x compilation
- [ ] C++14 standards compliance

---

**Document Version:** 1.0  
**Last Updated:** May 17, 2026  
**Status:** Ready for Phase 1 implementation
