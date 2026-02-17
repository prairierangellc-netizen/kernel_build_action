# Android Kernel Build Action - Project Context

## Agent Quick Start

- For significant features or refactors, sketch a plan first; keep it updated as you work.
- Default to `rg` for searching and keep edits ASCII unless the file already uses non-ASCII.
- Run the component-specific checks below before handing work off; do not skip failing steps.
- When unsure which path to take, favor minimal risk changes that can run the workflow successfully.

## Project Overview

This is the **Android Kernel Build Action** - a comprehensive GitHub Action that automates the building of Android kernel source code. The project provides a flexible, configurable workflow for compiling Android kernels with support for various toolchains, architectures, and kernel modifications.

### Key Technologies
- **GitHub Actions** (YAML-based CI/CD automation)
- **TypeScript** (main build logic - migrated from bash composite action)
- **Node.js 20** (runtime environment)
- **@actions/toolkit** (core, exec, cache, artifact, github, tool-cache)
- **Python 3** (kernel patch scripts)
- **Android NDK/AOSP toolchains** (GCC and Clang)
- **Various kernel modification frameworks** (KernelSU, NetHunter, LXC, Re-Kernel, BBG)
- **Multi-architecture support** (AMD64, ARM64)
- **Coccinelle** (semantic patching for kernel modifications)

## Project Structure

```
/home/user/kernel_build_action/
├── action.yml              # Main GitHub Action definition (node20 runtime)
├── README.md               # Comprehensive usage documentation
├── mkdtboimg.py            # Python tool for DTB/DTBO image manipulation
├── package.json            # Node.js dependencies and scripts
├── tsconfig.json           # TypeScript compiler configuration
├── yarn.lock               # Yarn dependency lock file
├── .eslintrc.json          # ESLint configuration
├── .prettierrc             # Prettier formatting configuration
├── .yamllint               # YAML linting configuration
├── LICENSE                 # Apache License 2.0
├── SECURITY.md             # Security policy documentation
├── AGENTS.md               # Project context and developer guide
├── IFLOW.md                # iFlow CLI configuration
├── dist/                   # Compiled JavaScript output (ncc bundled)
│   └── index.js            # Single bundled file for distribution
├── src/                    # TypeScript source code
│   ├── index.ts            # Main entry point with main/post phases
│   ├── cache.ts            # @actions/cache integration for ccache
│   ├── clean.ts            # Cleanup logic (post phase)
│   ├── error.ts            # Error log analysis (30+ patterns)
│   ├── toolchain.ts        # Toolchain download and management
│   ├── kernel.ts           # Kernel source cloning and version detection
│   ├── config.ts           # Kernel config manipulation (LTO, KVM, etc.)
│   ├── patches.ts          # Kernel patches (KernelSU, NetHunter, LXC, BBG, ReKernel)
│   ├── builder.ts          # Kernel compilation with make
│   ├── packager.ts         # Output packaging (boot.img/AnyKernel3)
│   ├── artifact.ts         # @actions/artifact integration
│   ├── release.ts          # GitHub Release creation
│   └── utils.ts            # Utility functions
├── .gemini/                # AI assistant configuration
│   ├── config.yaml
│   └── styleguide.md
├── .github/                # GitHub configuration
│   ├── dependabot.yml
│   ├── pull_request_template.md
│   ├── ISSUE_TEMPLATE/
│   │   ├── bug-report.yml
│   │   ├── common.yml
│   │   └── config.yml
│   └── workflows/
│       ├── main.yml
│       ├── build.yml
│       ├── lint.yml
│       ├── check.yml
│       ├── lkm.yml
│       └── close-pr.yml
├── kernelsu/               # KernelSU integration scripts
│   ├── apply_cocci.py
│   ├── classic.cocci
│   ├── minimal.cocci
│   └── README.md
├── lxc/                    # LXC/Docker support
│   ├── cgroup.cocci
│   ├── config.py
│   ├── patch_cocci.py
│   ├── xt_qtaguid.cocci
│   └── README.md
├── nethunter/              # Kali NetHunter integration
│   ├── config.py
│   ├── patch.py
│   └── README.md
└── rekernel/               # Re-Kernel support patches
    ├── patches/
    │   ├── binder.cocci
    │   ├── PATCH_ANALYSIS.md
    │   ├── proc_ops.cocci
    │   └── signal.cocci
    ├── cocci.zip
    ├── Kconfig
    ├── Makefile
    ├── patch.py
    ├── README.md
    ├── rekernel.c
    ├── rekernel.h
    └── src.zip
```

## Architecture

### TypeScript Action Structure

The action uses a two-phase execution model:

#### Main Phase
- Environment validation (GitHub Actions Linux runner)
- Dependency installation (apt/pacman)
- Toolchain setup (AOSP/Custom GCC/Clang)
- Kernel source cloning
- Patch application (KernelSU, NetHunter, LXC, BBG, ReKernel)
- Kernel compilation with make
- Output packaging (boot.img or AnyKernel3)
- Artifact upload or Release creation

#### Post Phase (always runs)
- Error log analysis (only if build failed)
- Cleanup (removes toolchains, kernel source, temp files)
- Environment variable cleanup

### Module Organization

| Module | Responsibility |
|--------|---------------|
| `cache.ts` | ccache setup with @actions/cache |
| `clean.ts` | Directory and file cleanup |
| `error.ts` | Build log analysis with 30+ error patterns |
| `toolchain.ts` | Toolchain download and path management |
| `kernel.ts` | Git operations and kernel version detection |
| `config.ts` | Kernel config modifications |
| `patches.ts` | Integration with Python patch scripts |
| `builder.ts` | make execution with proper environment |
| `packager.ts` | boot.img/AnyKernel3 packaging |
| `artifact.ts` | @actions/artifact integration |
| `release.ts` | GitHub Release creation |
| `utils.ts` | Helper functions |

## Building and Running

### Development Workflow

```bash
# Install dependencies
yarn install

# Build (compile TypeScript to dist/index.js)
yarn build

# Lint
yarn lint

# Format
yarn format
```

### Build Process

1. TypeScript Compilation: tsc compiles src/**/*.ts
2. Bundling: @vercel/ncc bundles into dist/index.js
3. Distribution: The bundled file is committed for GitHub Actions use

### Usage

Users reference the action in workflows:

```yaml
- name: Build Kernel
  uses: dabao1955/kernel_build_action@main
  with:
    kernel-url: https://github.com/username/kernel_repo
    kernel-branch: main
    config: defconfig
    arch: arm64
    aosp-clang: true
    android-version: 12
```

## Dependencies

### Runtime
- @actions/core: Input/output, logging, state
- @actions/exec: Shell command execution
- @actions/cache: ccache caching
- @actions/artifact: Build artifact upload
- @actions/github: GitHub API
- @actions/tool-cache: Tool downloading
- @octokit/rest: GitHub REST API

### Development
- typescript: TypeScript compiler
- @vercel/ncc: Bundler
- eslint: Linting
- prettier: Formatting
- @types/node: Type definitions

## Key Features

### Kernel Modifications
- KernelSU: Root access framework (with LKM support)
- NetHunter: Penetration testing tools
- LXC/Docker: Container support
- Re-Kernel: Performance optimizations
- BBG: BaseBandGuard security
- KVM: Hardware virtualization

### Toolchain Support
- AOSP GCC/Clang
- Custom toolchains via URL
- System toolchain fallback

### Build Features
- ccache via @actions/cache
- LTO control
- Parallel builds
- Cross-compilation

### Output Options
- boot.img
- AnyKernel3 ZIP
- GitHub Release

## Git Commit Conventions

```
component: <type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

Example:
```
action: feat(cache): Add @actions/cache for ccache

Replace external ccache-action with native @actions/cache.

Signed-off-by: user <user@example.com>
```

## Code Quality

Before committing:
```bash
yarn lint
yarn format:check
yamllint action.yml
yarn build
```
