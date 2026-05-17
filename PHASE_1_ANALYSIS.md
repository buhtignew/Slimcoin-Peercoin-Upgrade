# Phase 1: Build System & Dependencies - Detailed Analysis

## Current State Assessment

### Build System Overview

**Current Setup:**
- Makefile-based build system (`src/makefile.unix`)
- Travis CI for continuous integration (`.travis.yml`)
- Manual dependency management
- C++11 support (with legacy C++0x fallback)

### Analysis of makefile.unix

**Key Observations:**

```makefile
# C++ Standard (lines 20-24)
ifneq (${USE_OLDC}, -)
    DEFS += -std=c++0x
else
    DEFS += -std=c++11
endif
```

**Status**: Build supports both C++0x and C++11, defaulting to C++11. This is good but dated.

**Dependencies Detected:**

| Library | Purpose | Current Status |
|---------|---------|----------------|
| Boost | Threading, filesystem, program options | Very old (1.55+ era) |
| BerkeleyDB (db_cxx) | Wallet storage | EOL, deprecated |
| OpenSSL | Cryptography | CRITICAL: EOL |
| miniupnpc | UPnP support | Optional, can be updated |
| zlib | Compression | Standard, easy update |
| pthread | Threading | System library |

### Analysis of .travis.yml

**Configuration Details:**
- OS: Ubuntu 14.04 LTS "Trusty" (EOL: April 2019)
- Compiler: GCC
- Qt: 5.5 (from PPA)
- Build: Both CLI (`slimcoind`) and GUI

**Status**: Travis CI free tier ended September 2020. Configuration is outdated.

---

## Dependency Audit

### Critical Vulnerabilities

#### 🔴 OpenSSL 1.0.1 (CRITICAL)

**Current Status:**
- Likely using OpenSSL 1.0.1 or 1.0.2
- End of Life: January 2016
- **10 YEARS OUTDATED**
- Known vulnerabilities: CVE-2014-0160 (Heartbleed) and 100+ others

**Recommendation:**
- Upgrade to OpenSSL 1.1.1 (minimum)
- Later: OpenSSL 3.0+ support
- **Priority: HIGH** - Security critical

#### ⚠️ BerkeleyDB (MEDIUM-HIGH)

**Current Status:**
- Used for wallet storage
- No longer maintained
- Performance issues on modern systems

**Options:**
1. Keep for compatibility, upgrade version
2. Migrate to LevelDB (like Bitcoin Core did)
3. Keep as legacy, offer modern alternative

**Recommendation for Slimcoin:**
- Phase 1: Keep compatible version
- Phase 6: Evaluate LevelDB migration

#### ⚠️ Boost Library

**Current Status:**
- Version 1.55 or older
- Used for: threading, filesystem, program_options, unit_test_framework

**Recommendation:**
- Upgrade to Boost 1.70+ (2018)
- Later: Evaluate C++ std alternatives

### Standard Libraries

#### ✅ zlib
- Widely available, easy to update
- Recommend: Latest stable (1.2.x series)

#### ✅ Qt Framework
- Current: Qt 5.5 (very old)
- Recommend: Qt 5.15 LTS or Qt 6.x
- Not critical for Phase 1 if focusing on CLI

---

## Build System Recommendations

### Option A: Minimal Changes (Low Risk)

**Approach:** Minimal modifications to makefile.unix

**Changes:**
1. Update compiler flags to C++14
2. Update dependency versions
3. Replace Travis CI with GitHub Actions

**Pros:**
- Low risk
- Preserves existing build process
- Faster implementation

**Cons:**
- Makefile becomes more complex
- Harder to maintain long-term
- Cross-platform builds more difficult

### Option B: Modern Build System (Recommended)

**Approach:** Introduce CMake build system alongside makefile

**Changes:**
1. Create CMakeLists.txt
2. Keep makefile.unix for backwards compatibility
3. Use GitHub Actions to test both

**Pros:**
- Better cross-platform support
- Easier dependency management
- Modern ecosystem
- Attracts newer developers

**Cons:**
- More work upfront
- Requires learning curve
- Testing both systems

**Recommendation: Option B** - Better long-term sustainability

---

## C++ Standard Decision

### Current: C++11
### Recommended for Phase 1: C++14
### Future: C++17

**Rationale:**
- C++14 adds:
  - `std::make_unique`
  - Generic lambdas
  - Relaxed constexpr
  - Better auto type deduction
- Widely supported by GCC 5+, Clang 3.4+
- Not a breaking change
- Enables better code from Peercoin

**Action:**
```makefile
# Change from:
DEFS += -std=c++11

# To:
DEFS += -std=c++14
```

---

## CI/CD Migration: Travis → GitHub Actions

### Current Problem
- Travis CI free tier ended
- Configuration outdated
- Need replacement

### Solution: GitHub Actions

**Advantages:**
- Free for public repositories
- Native GitHub integration
- YAML-based configuration (like Travis)
- More powerful than Travis
- Better secrets management

**Proposed Workflow:**
```yaml
name: Build and Test
on: [push, pull_request]
jobs:
  build-linux:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Install dependencies
        run: sudo apt-get install -y libboost-dev libssl-dev libdb++-dev
      - name: Build
        run: cd src && make -f makefile.unix -j 4 slimcoind
      - name: Test
        run: cd src && make -f makefile.unix test_slimcoin && ./test_slimcoin -r short
```

---

## Ubuntu Base Image Migration

### Current: Ubuntu 14.04 LTS (Trusty)
- EOL: April 2019
- 7 years old

### Recommended: Ubuntu 20.04 LTS (Focal)
- Support until 2025
- Modern toolchain
- Better performance

### Benefits:
- GCC 9.x (much better C++ support)
- Boost 1.71+ available
- OpenSSL 1.1.1
- Qt 5.12

---

## Platform Support Matrix

| Platform | Current | Recommended | Status |
|----------|---------|-------------|--------|
| Linux (Ubuntu 14.04) | ✅ | ❌ EOL | Upgrade to 20.04 |
| Linux (Ubuntu 20.04) | ? | ✅ | Add support |
| macOS | ❌ | ✅ | Add GitHub Actions |
| Windows | ⚠️ Manual | ⚠️ Manual | Keep as-is |

---

## Task Breakdown

### Task 1: GitHub Actions Setup (2 hours)

**What:** Create `.github/workflows/build.yml`

**Steps:**
1. Create Linux build job
2. Create macOS build job (optional)
3. Test both CLI and GUI builds
4. Add test execution

**Success Criteria:**
- ✅ Linux builds succeed
- ✅ Tests pass
- ✅ GUI compiles (if applicable)

### Task 2: C++ Standard Upgrade (1-2 hours)

**What:** Update makefile.unix to use C++14

**Steps:**
1. Change `-std=c++11` to `-std=c++14`
2. Build and test
3. Fix any C++11-only issues
4. Document changes

**Success Criteria:**
- ✅ Builds with C++14
- ✅ All tests pass
- ✅ No warnings about C++ version

### Task 3: OpenSSL Assessment (2-3 hours)

**What:** Analyze OpenSSL upgrade impact

**Steps:**
1. Identify OpenSSL version in use
2. Test upgrade to 1.1.1
3. Document API changes needed
4. Create migration plan

**Success Criteria:**
- ✅ OpenSSL version identified
- ✅ Upgrade path documented
- ✅ Migration plan created

### Task 4: Dependency Inventory (2-3 hours)

**What:** Create comprehensive dependency matrix

**Steps:**
1. Document all dependencies
2. Find update paths
3. Check for security issues
4. Test each update

**Success Criteria:**
- ✅ All dependencies documented
- ✅ Upgrade paths identified
- ✅ Security issues noted

### Task 5: Testing & Documentation (3-4 hours)

**What:** Verify all changes work together

**Steps:**
1. Test on Linux
2. Test on macOS (if feasible)
3. Document all findings
4. Create implementation guide

**Success Criteria:**
- ✅ Clean builds on all platforms
- ✅ All tests pass
- ✅ Documentation complete

---

## Timeline

**Phase 1 Estimated Duration: 1-2 weeks**

**Week 1:**
- Day 1-2: GitHub Actions setup
- Day 3-4: C++ standard upgrade
- Day 5: Testing and documentation

**Week 2:**
- Day 1-2: OpenSSL assessment
- Day 3-4: Dependency inventory
- Day 5: Final documentation and sign-off

---

## Risk Assessment

| Change | Risk | Impact |
|--------|------|--------|
| GitHub Actions | LOW | Build system, no runtime |
| C++14 upgrade | LOW | Compiler feature, backwards compatible |
| Dependency updates | MEDIUM | Compatibility must be tested |
| Ubuntu 20.04 | MEDIUM | CI/CD only |
| OpenSSL plan | LOW | Planning phase only |

---

## Success Criteria for Phase 1

✅ All critical criteria must be met:

- [ ] GitHub Actions workflow creates successful builds
- [ ] All tests pass with C++14
- [ ] Build system documented
- [ ] Dependency matrix complete
- [ ] OpenSSL upgrade path identified
- [ ] No regressions in functionality
- [ ] New documentation added to repository

---

## Next Steps

1. **Review** this analysis
2. **Decide** on CMake vs. makefile-only
3. **Start** Task 1: GitHub Actions
4. **Report** results
5. **Proceed** to next tasks

---

## Questions for Copilot

1. Should we introduce CMake or stick with makefile?
2. Should we drop Qt support or maintain it?
3. Should we target C++14 or C++17?
4. Timeline: 1 week or 2 weeks?

---

**Document Version:** 1.0  
**Date:** May 17, 2026  
**Status:** Ready for implementation planning
