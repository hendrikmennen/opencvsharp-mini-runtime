# AGENTS.md

## Purpose
This repository builds and publishes slim native OpenCvSharp runtime packages for multiple OS/arch targets.

The repo does not contain the full OpenCvSharp/OpenCV source trees. Instead, CI pulls upstream sources and applies local workflow + patch customizations.

Important: produced NuGet packages are consumed by a desktop app runtime, not only CI.
Prioritize runtime portability and end-user machine behavior over CI-only success.

## Repository Layout
- `.github/workflows/opencv.yml`: builds OpenCV artifacts per OS/arch.
- `.github/workflows/opencvsharp.yml`: downloads OpenCV artifacts, checks out upstream OpenCvSharp, applies `eng/opencvsharp.patch`, builds `OpenCvSharpExtern`, uploads artifacts.
- `.github/workflows/make-nuget.yml`: downloads OpenCvSharp artifacts, creates runtime NuGet packages, and pushes them.
- `eng/opencvsharp.patch`: patch applied to upstream `shimat/opencvsharp` tag defined in workflow.

## Critical Coupling Rules
1. Keep matrix naming aligned across workflows.
- `opencv.yml` artifact names: `opencv-${os}-${arch}`
- `opencvsharp.yml` artifact names: `opencvsharp-${os}-${arch}`
- `make-nuget.yml` matrix `upstream` values must match these artifact suffixes.

2. Keep upstream versions compatible.
- OpenCV version is defined in `opencv.yml` (`OPENCV_VERSION`).
- OpenCvSharp version is defined in `opencvsharp.yml` (`OPENCVSHARP_VERSION`).
- `eng/opencvsharp.patch` must apply cleanly to the OpenCvSharp version in workflow.

3. Any change to native runtime dependencies must be propagated to packaging.
- If new `.dll`/`.so`/`.dylib` dependencies are needed at runtime, ensure they are copied in `opencvsharp.yml` artifact step and included in `make-nuget.yml`.

## Safe Change Workflow For Agents
1. Inspect affected workflow(s) and confirm artifact naming/version assumptions.
2. If changing `eng/opencvsharp.patch`, validate it against upstream before finalizing.
3. Update workflow comments when behavior changes (codec/backends/dependency strategy).
4. Verify no orphan references remain (artifact names, paths, matrix keys).

## Required Validation (Before Final Answer)
Run these checks locally when relevant:

### Patch Applicability Check
Use when `eng/opencvsharp.patch` changes.

```bash
tmp=$(mktemp -d)
git clone --depth 1 --branch 4.11.0.20250507 https://github.com/shimat/opencvsharp "$tmp/opencvsharp"
cd "$tmp/opencvsharp"
git apply --check /path/to/repo/eng/opencvsharp.patch
```

If `OPENCVSHARP_VERSION` changes, update branch/tag in this check accordingly.

### Workflow Sanity Check
At minimum, verify YAML parses and key references are consistent.

```bash
rg -n "opencv-\$\{\{ matrix\.os \}\}-\$\{\{ matrix\.arch \}\}|opencvsharp-\$\{\{ matrix\.os \}\}-\$\{\{ matrix\.arch \}\}" .github/workflows
```

If `actionlint` is available, run it:

```bash
actionlint
```

## Editing Guidelines
- Prefer minimal diffs in workflow files; these are release-critical.
- Do not introduce destructive git operations.
- Do not assume local source trees for OpenCV/OpenCvSharp exist in this repo.
- Keep comments accurate and brief; remove stale comments when behavior changes.

## Common Pitfalls
- Patch applies in local clone but not in CI due to OpenCvSharp version drift.
- Artifact directory/path mismatch between `opencv.yml` and `opencvsharp.yml`.
- Adding runtime dependencies but forgetting NuGet inclusion in `make-nuget.yml`.
- Updating one OS branch logic but not the corresponding matrix variants.

## Quick Context Commands
```bash
rg --files
rg -n "OPENCV_VERSION|OPENCVSHARP_VERSION|matrix|artifact|ffmpeg|videoio" .github/workflows eng/opencvsharp.patch
```
