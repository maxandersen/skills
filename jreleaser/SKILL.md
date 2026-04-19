---
name: jreleaser
description: Configure JReleaser to release projects to GitHub — supports Go, Rust, Java, Zig, C#, Deno, Swift, and 20+ languages. Creates jreleaser.yml and GitHub Actions workflows from proven templates.
metadata:
  tags: jreleaser, release, github, ci-cd, packaging
---

## When to use

Use when setting up JReleaser for a project, creating `jreleaser.yml`, or adding release workflows.
This skill covers **GitHub releases only** — the fast path that needs nothing beyond a GitHub repo and token.
For package managers (Homebrew, Snap, Scoop), Maven Central, signing, or announcements, consult the docs (see bottom).

## The simplest release (no config needed)

If you just want to tag a GitHub release with some files — no cross-compilation, no packaging:

```bash
# Auto-config mode: no jreleaser.yml needed at all
jreleaser release --auto-config \
  --project-name=myapp \
  --project-version=1.0.0 \
  --file=dist/myapp.zip
```

Or with a minimal `jreleaser.yml` (just creates a tagged release with auto-generated changelog):

```yaml
project:
  name: myapp
  version: 1.0.0
```

Then run `jreleaser release`. That's it.

See: `https://jreleaser.org/guide/latest/examples/miscellaneous/simple-release.html` and `auto-config-release.html`

## Cross-platform releases

For building and releasing binaries across platforms, every JReleaser config follows one of two patterns:

### Pattern 1: Native binary (Go, Rust, Zig, C#, Deno, Swift, C++, Nim, etc.)

All native binary repos share the exact same structure — only the build commands and matrix labels differ.

```yaml
# 1. Build matrix: maps language-specific targets to JReleaser platforms
matrix:
  rows:
    - { VAR1: val, VAR2: val, platform: osx-aarch_64   }
    - { VAR1: val, VAR2: val, platform: osx-x86_64     }
    - { VAR1: val, VAR2: val, platform: linux-aarch_64  }
    - { VAR1: val, VAR2: val, platform: linux-x86_64    }
    - { VAR1: val, VAR2: val, platform: windows-x86_64  }

# 2. Hooks: run the language's build command per matrix row
hooks:
  script:
    before:
      - run: |
          # language-specific build command using {{ matrix.VAR1 }}, {{ matrix.VAR2 }}
        applyDefaultMatrix: true
        verbose: true
        filter:
          includes: ['assemble']

# 3. Project metadata
project:
  name: myapp
  description: My CLI tool
  links:
    homepage: https://github.com/OWNER/REPO
  authors:
    - Your Name
  license: MIT
  inceptionYear: 2024
  stereotype: CLI

# 4. Release config (same for everyone)
release:
  github:
    overwrite: true
    changelog:
      formatted: ALWAYS
      preset: conventional-commits
      contributors:
        format: '- {{contributorName}}{{#contributorUsernameAsLink}} ({{.}}){{/contributorUsernameAsLink}}'

# 5. Archive assembly (same structure for everyone)
assemble:
  archive:
    myapp:
      active: ALWAYS
      formats: [ ZIP ]
      applyDefaultMatrix: true
      archiveName: '{{distributionName}}-{{projectVersion}}-MATRIX_VARS'
      fileSets:
        - input: 'target/BUILD_OUTPUT_DIR'
          output: 'bin'
          includes: [ 'myapp{.exe,}' ]
        - input: '.'
          includes: [ 'LICENSE' ]

# 6. Distribution (same for everyone)
distributions:
  myapp:
    executable:
      windowsExtension: exe
```

**To get the exact build commands and matrix for your language**, each has:
- A **helloworld repo** with working config: `github.com/jreleaser/helloworld-<name>`
- An **example doc page** explaining the setup (fetch as markdown for efficiency)

| Name | Doc page | Build approach |
|---|---|---|
| `go` | `miscellaneous/go.html` | `go build` per GOOS/GOARCH |
| `rustx` | `miscellaneous/rust.html` | `cargo-zigbuild` per Rust target triple |
| `zig` | `miscellaneous/zig.html` | `zig build-exe` per arch-os |
| `csharp` | `miscellaneous/csharp.html` | `dotnet publish` per runtime ID |
| `fsharp` | `miscellaneous/fsharp.html` | `dotnet publish` per runtime ID |
| `deno` | `miscellaneous/deno.html` | `deno compile --target` per target |
| `bun` | `miscellaneous/bun.html` | `bun build --compile --target` per target |
| `swift` | `miscellaneous/swift.html` | `swift build` (macOS/Linux only) |
| `cpp` | `miscellaneous/cpp.html` | CMake per platform |
| `nim` | `miscellaneous/nim.html` | `nim compile` per platform |
| `crystal` | `miscellaneous/crystal.html` | `crystal build` per platform |
| `odin` | `miscellaneous/odin.html` | `odin build` per platform |
| `haskell` | `miscellaneous/haskell.html` | `stack build` per platform |
| `elixir` | `miscellaneous/elixir.html` | Burrito per platform |
| `ocaml` | `miscellaneous/ocaml.html` | `opam`/`dune` per platform |
| `pascal` | `miscellaneous/pascal.html` | Free Pascal per platform |
| `perl` | `miscellaneous/perl.html` | PAR::Packer per platform |
| `ballerina` | `miscellaneous/ballerina.html` | `bal build` with GraalVM |

Doc pages are at `https://jreleaser.org/guide/latest/examples/<path>`. Use `https://markdown.new/<url>` for smaller token reads.

To fetch a repo's raw config:
```bash
curl -s https://raw.githubusercontent.com/jreleaser/helloworld-<LANG>/main/jreleaser.yml
```

### Pattern 2: Java

Java has sub-variants depending on how you distribute:

**Simplest — single JAR** (no assembly, users need Java installed):
```yaml
project:
  name: myapp
  description: My Java app
  license: MIT
  stereotype: CLI
  languages:
    java:
      version: 17
      groupId: com.example
      artifactId: myapp
      mainClass: com.example.Main

release:
  github:
    overwrite: true
    changelog:
      formatted: ALWAYS
      preset: conventional-commits

distributions:
  myapp:
    type: SINGLE_JAR
    artifacts:
      - path: target/{{distributionName}}-{{projectVersion}}.jar
```

**Java binary** (JAR wrapped in a zip with scripts):
```yaml
# Same project/release as above, plus:
assemble:
  javaArchive:
    myapp:
      active: ALWAYS
      formats: [ ZIP ]
      fileSets:
        - input: '.'
          includes: [ 'LICENSE' ]
      mainJar:
        path: target/{{distributionName}}-{{projectVersion}}.jar
```

**For jlink, jpackage, or GraalVM native-image** — these are more complex. Each has a doc page and helloworld repo:

| Variant | Doc page | Repo |
|---|---|---|
| jlink | `examples/java/jlink.html` | `helloworld-java-jlink` |
| jpackage | `examples/java/jpackage.html` | `helloworld-java-jpackage` |
| GraalVM native-image | `examples/java/binary.html` | `helloworld-java-graalvm` |
| Java binary (scripts+JAR) | `examples/java/java-binary.html` | `helloworld-java-bin` |
| Single JAR | `examples/java/single-jar.html` | `helloworld-java-jar` |

Fetch doc as markdown: `https://markdown.new/https://jreleaser.org/guide/latest/<doc-page>`

## GitHub Actions workflow

Two workflows — use both. Fetch from any helloworld repo:

```bash
curl -s https://raw.githubusercontent.com/jreleaser/helloworld-go/main/.github/workflows/release.yml
curl -s https://raw.githubusercontent.com/jreleaser/helloworld-go/main/.github/workflows/early-access.yml
```

### Release workflow (manual trigger)

```yaml
name: Release
on:
  workflow_dispatch:
    inputs:
      version:
        description: 'Release version'
        required: true

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      # Add language setup step here (actions/setup-go, actions/setup-java, etc.)

      - name: Assemble
        uses: jreleaser/release-action@v2
        with:
          arguments: assemble
        env:
          JRELEASER_PROJECT_VERSION: ${{ github.event.inputs.version }}
          JRELEASER_GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

      - name: Release
        uses: jreleaser/release-action@v2
        with:
          arguments: release
        env:
          JRELEASER_PROJECT_VERSION: ${{ github.event.inputs.version }}
          JRELEASER_GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

      - name: JReleaser output
        if: always()
        uses: actions/upload-artifact@v4
        with:
          retention-days: 1
          name: jreleaser-release
          path: |
            out/jreleaser/trace.log
            out/jreleaser/output.properties
```

### Early access workflow (on push to main)

```yaml
name: EarlyAccess
on:
  push:
    branches: [ main ]

jobs:
  precheck:
    if: startsWith(github.event.head_commit.message, 'Releasing version') != true
    runs-on: ubuntu-latest
    outputs:
      VERSION: ${{ steps.vars.outputs.VERSION }}
    steps:
      - uses: actions/checkout@v4
      - id: vars
        run: echo "VERSION=$(cat VERSION)" >> $GITHUB_OUTPUT

  release:
    needs: [ precheck ]
    if: endsWith(${{ needs.precheck.outputs.VERSION }}, '-SNAPSHOT')
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      # Add language setup step here

      - name: Assemble
        uses: jreleaser/release-action@v2
        with:
          arguments: assemble
        env:
          JRELEASER_PROJECT_VERSION: ${{ needs.precheck.outputs.VERSION }}
          JRELEASER_GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

      - name: Release
        uses: jreleaser/release-action@v2
        with:
          arguments: release
        env:
          JRELEASER_PROJECT_VERSION: ${{ needs.precheck.outputs.VERSION }}
          JRELEASER_GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

      - name: JReleaser output
        if: always()
        uses: actions/upload-artifact@v4
        with:
          retention-days: 1
          name: jreleaser-release
          path: |
            out/jreleaser/trace.log
            out/jreleaser/output.properties
```

The early-access workflow needs a `VERSION` file in the repo root (e.g., `0.1.0-SNAPSHOT`).

## Key things to know

- **Mustache templates**: `{{distributionName}}`, `{{projectVersion}}`, `{{ matrix.VAR }}` — double braces, spaces optional inside
- **`stereotype: CLI`** tells JReleaser this is a command-line tool
- **`applyDefaultMatrix: true`** on hooks and assemble sections means "run this for each matrix row"
- **`platform` in matrix rows** must use JReleaser's naming: `osx-x86_64`, `osx-aarch_64`, `linux-x86_64`, `linux-aarch_64`, `windows-x86_64`, `windows-aarch_64`
- **Platform replacements** if downstream tools use different names:
  ```yaml
  platform:
    replacements:
      aarch_64: aarch64
  ```
- **Validate before releasing**: `jreleaser config` checks config, `jreleaser assemble` tests assembly
- **Environment variables** override config: `JRELEASER_PROJECT_VERSION`, `JRELEASER_GITHUB_TOKEN`
- **The GitHub Action** is `jreleaser/release-action@v2`
- **SLSA provenance**: append `-slsa` to any helloworld repo name for the provenance variant

## Going further (docs)

For advanced features requiring external service registration, fetch the relevant doc page as markdown for efficient reading:

```
https://markdown.new/https://jreleaser.org/guide/latest/reference/<page>.html
```

Key pages:
- `reference/packagers/index.html` — Homebrew, Snap, Scoop, Chocolatey, Docker, Flatpak, WinGet
- `reference/deploy/index.html` — Maven Central, Nexus, Artifactory
- `reference/signing.html` — GPG/cosign signing
- `reference/announce/index.html` — Twitter, Mastodon, Slack, Discord, Zulip, etc.
- `reference/catalog/index.html` — SBOM generation (CycloneDX, SPDX, Syft)
- `reference/hooks/index.html` — script/command hooks
- `reference/name-templates.html` — all available template variables
- `reference/matrix.html` — build matrix reference
- `continuous-integration/index.html` — GitLab CI, Jenkins, Azure DevOps, etc.
