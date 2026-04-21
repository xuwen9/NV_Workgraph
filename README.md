# NV_Workgraph

Research and experimentation on **D3D12 Work Graphs**, a GPU-driven execution model introduced by Microsoft in 2024 and supported by both NVIDIA and AMD drivers. This repository contains a minimal end-to-end sample used to study how the command processor schedules node records into wavefronts, how backing memory mediates producer/consumer flow between nodes, and how the dispatch path differs from traditional `Dispatch` calls.

## What this repo contains

A single-file D3D12 Work Graphs sample (`HelloWorkGraph.cpp`) based on AMD's publicly available reference (`license.txt` retained per license terms). The sample sets up the minimum pipeline required to exercise a Work Graph:

- DXC compilation of an HLSL node shader (`lib_6_8`) annotated with `[Shader("node")]`, `[NodeLaunch("broadcasting")]`, `[NodeDispatchGrid]`, `[NumThreads]`
- State object assembly combining a DXIL library subobject, Work Graph subobject, and global root signature
- Backing memory allocation via `ID3D12WorkGraphProperties::GetWorkGraphMemoryRequirements`
- Dispatch through `ID3D12GraphicsCommandList10::SetProgram` + `DispatchGraph`
- UAV readback and fence synchronization

The shader writes `"Hello World"` to a UAV and the host reads it back — intentionally minimal so the full pipeline fits in one source file.

## Research focus

The sample is a starting point for deeper investigation into:

- **Launch modes** — `broadcasting` vs `coalescing` vs `thread` and their impact on wave packing at the SPI
- **Backing memory semantics** — how record queues, producer/consumer FIFOs, and scheduler metadata are laid out in GPU scratch
- **Cache coherency** — visibility of node-to-node record writes across the GPU cache hierarchy
- **Command processor scheduling** — how new dispatch packets are decoded and translated into wave launches
- **Fence/interrupt ordering** — ensuring completion is signaled only after all dynamically-spawned node work retires

## Build requirements

- Windows 10/11 with a GPU and driver supporting `D3D12_WORK_GRAPHS_TIER_1_0`
- Agility SDK 1.613+ (vendored via `packages.config`)
- Visual Studio 2022 with C++ desktop workload
- DirectX Shader Compiler (`dxcompiler.dll`, loaded at runtime)

Open `HelloWorkGraph.sln`, build x64, and run.

## References

- [D3D12 Work Graphs announcement](https://devblogs.microsoft.com/directx/d3d12-work-graphs/)
- [GPUOpen Work Graphs introduction](https://gpuopen.com/learn/gpu-work-graphs/gpu-work-graphs-intro/)
- [Getting Started with Work Graphs](https://gpuopen.com/learn/gpu-work-graphs/gpu-work-graphs-part1/)

## License

MIT — see `license.txt`. Original sample copyright © 2023-2024 Advanced Micro Devices, Inc.
