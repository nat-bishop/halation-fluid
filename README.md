# Halation Fluid

**A native GPU fluid engine, with liquid simulation inside Houdini.**

Halation Fluid is a fluid simulation project built in Rust with wgpu and WGSL. It combines particle-and-grid liquid simulation, surface reconstruction, and a Houdini integration that brings the engine into an artist's existing geometry and animation workflow.

The engine grew out of [Halation](https://github.com/nat-bishop/halation), an agent-native procedural 3D application. It is now being prepared as a standalone engine and Houdini addon, with liquid simulation as the initial product focus. The broader engine also includes Eulerian smoke and fire, OpenVDB output, and particle and whitewater machinery.

> **Project showcase.** This repository describes the project and its engineering. The implementation is private; this is not a source release or an installable distribution.

## Liquid simulation

The liquid solver combines particles with a staggered velocity grid. FLIP and APIC transfer modes support different motion characteristics, from splashy flows to smoother swirling motion.

- **Physical controls:** gravity, viscosity, and surface tension expressed in physical units.
- **Animated interaction:** liquid sources and colliders driven by host geometry, including translation, rotation, and supported deformation.
- **Dense and narrow-band simulation:** alternative storage modes, with narrow-band particles concentrated near the liquid surface.
- **GPU numerical work:** pressure and density projection, adaptive substeps, and surface reconstruction.
- **Reusable simulation state:** persistent sessions, checkpoints, and cached frame outputs support playback, continuation, and edits.

The engine produces particles, reconstructed surfaces, and field observations that a host can use for visualization, surfacing, or downstream processing.

## Working in Houdini

The Houdini integration is a SOP asset with three geometry inputs: **initial liquid**, **liquid source**, and **collider**. Artists supply ordinary SOP geometry and animate it through their existing networks.

The asset evaluates those inputs on Houdini's timeline and drives a native solver process. It can return particles, surface and velocity fields, or a native surface mesh. Houdini's Particle Fluid Surface and File Cache nodes can then handle downstream surfacing and caching.

Houdini owns scene evaluation and presentation. The native engine owns simulation state, substeps, checkpoint recovery, and output publication. Keeping those responsibilities separate lets the integration reuse the liquid engine without requiring the Halation application to run.

## How it is built

| Layer | Responsibility |
|---|---|
| Host integration | Evaluate geometry and animation, prepare physical inputs, display results |
| Native liquid session | Own the simulation lifecycle, cache identity, checkpoints, and committed outputs |
| Rust + wgpu / WGSL | Run the GPU solver and surface reconstruction |
| Output layer | Expose particles, meshes, and fields for host workflows; OpenVDB for gas volumes |

Batch simulation and the persistent liquid process share one session implementation. This keeps core simulation behavior in one place while allowing different hosts to control when work happens and how results are presented.

## Engineering approach

Development pairs visual inspection with numerical checks: volume behavior, collision response, finite values, solver convergence, and repeatability under controlled configurations. Cache and checkpoint tests examine whether continuing or restoring a simulation preserves the expected result.

Host integration checks also matter: a correct solver is only useful if the host supplies geometry, motion, units, and frame timing correctly. Houdini testing therefore covers input conversion, moving geometry, output placement, playback, and downstream surfacing.

## Current status

**Active development — September 2026.** The fluid engine and Houdini integration originated in the Halation project; their standalone extraction and release preparation are in progress.

The integration has documented development testing on Windows 11 with Houdini 22.0.368 Apprentice and an NVIDIA GPU using Vulkan. That describes the original integration's tested environment, not a general compatibility guarantee.

A 0.1.0 release candidate has been built and validated privately: standalone build and CPU checks, GPU liquid acceptance, real-Houdini host acceptance, and a clean install from the versioned package. No public binaries or installable Houdini package are offered here yet.

**Stack:** Rust · wgpu · WGSL · Python · Houdini · OpenVDB

Built by [Nat Bishop](https://github.com/nat-bishop). See the [Halation showcase](https://github.com/nat-bishop/halation) for the larger procedural 3D project.

## Rights

© 2026 Nat Bishop. All rights reserved. This page is published for portfolio and evaluation purposes. The engine and addon source remain private.
