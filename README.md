<!--- Copyright Allo authors. All Rights Reserved. -->
<!--- SPDX-License-Identifier: Apache-2.0  -->

Restructured Allo Project
=========================

This repository is a restructured version of the Allo project.

Overview
--------

In this version:

*   The original example files under `examples/` are treated as **entry files** for direct execution.
    
*   The `allo` package, previously intended to be installed as a standalone module, is now directly imported by the example scripts.
    
*   The project is more self-contained and streamlined for local usage.
    

Notable Changes
---------------

*   Integrated the patched LLVM-19 project required to build the local LLVM-19 runtime.
    
*   Included the `past` Python package (`past-0.7.2`) from this release.
    
*   Updated `requirements.py` accordingly.
    
*   Removed `setup.py`.

Requirements
------------

A Linux x86-64 system with Conda installed.

Setup Instructions
------------------

Follow the steps below to set up and run the project.

### Create a Conda Environment

```bash
conda create -n allo -y
conda activate allo
conda install -c conda-forge clang clangxx cmake git ninja python=3.12 pybind11 -y
```

### Build the Local LLVM-19 Runtime

```bash
cd externals/llvm-project
mkdir -p build && cd build
cmake -G Ninja ../llvm \
-DLLVM_ENABLE_PROJECTS="clang;mlir;openmp" \
-DLLVM_BUILD_EXAMPLES=ON \
-DLLVM_TARGETS_TO_BUILD="host" \
-DCMAKE_BUILD_TYPE=Release \
-DLLVM_ENABLE_ASSERTIONS=ON \
-DLLVM_INSTALL_UTILS=ON \
-DMLIR_ENABLE_BINDINGS_PYTHON=ON \
-DPython3_EXECUTABLE=$(which python3)
ninja
# Set environment variable for LLVM build directory
export LLVM_BUILD_DIR=$(pwd)
cd ../../..
```

### Build MLIR Python Bindings

```bash
cd mlir
mkdir -p build && cd build
cmake -G Ninja .. \
-DMLIR_DIR=$LLVM_BUILD_DIR/lib/cmake/mlir \
-DPython3_EXECUTABLE=$(which python3)
# Temporarily increase the maximum number of open files allowed (for current session)
# Prevent errors such as `/usr/bin/ld: cannot find /usr/lib/x86_64-linux-gnu/libz.so: Too many open files`
ulimit -n 4096
ninja
cd ../..
```

### Install Additional Python Dependencies

```bash
pip3 install -r requirements.txt
```

### Run an Example

To run an example, use the `PYTHONPATH` environment variable to include the current directory in the module search path and the `LLVM_BUILD_DIR` environment variable:


```bash
PYTHONPATH=$(pwd) LLVM_BUILD_DIR=externals/llvm-project/build python3 examples/polybench/gemm.py
```

This will run the GEMM kernel using the CPU backend.

------

<img src="tutorials/allo-icon.png" width=128/> Accelerator Design Language
==============================================================================

[**Documentation**](https://cornell-zhang.github.io/allo) | [**Installation**](https://cornell-zhang.github.io/allo/setup/index.html) | [**Tutorials**](https://github.com/cornell-zhang/allo-tutorials)

![GitHub](https://img.shields.io/github/license/cornell-zhang/allo)
![Allo Test](https://github.com/cornell-zhang/allo/actions/workflows/config.yml/badge.svg)

Allo is a Python-embedded Accelerator Design Language (ADL) and compiler that facilitates the construction of large-scale, high-performance hardware accelerators in a modular and composable manner. Allo has several key features:
* **Progressive hardware customizations**: Allo decouples hardware customizations from algorithm specifications and treats each hardware customization as a primitive that performs a rewrite on the program. Allo not only decouples the loop-based transformations, but also extends the decoupling to memory, communication, and data types. All the transformations are built on top of [MLIR](https://mlir.llvm.org/) that is easier to target different backends.
* **Reusable parameterized kernel templates**: Allo supports declaring type variables during kernel creation and instantiating the kernel when building the hardware executable, which is an important feature for building reusable hardware kernel libraries. Allo introduces a concise grammar for creating kernel templates, eliminating the need for users to possess complicated metaprogramming expertise.
* **Composable schedules**: Allo empowers users to construct kernels incrementally from the bottom up, adding customizations one at a time while validating the correctness of each submodule. Ultimately, multiple schedules are progressively integrated into a complete design using the `.compose()` primitive. This approach, unachievable by prior top-down methods, significantly enhances productivity and debuggability.


## Getting Started

Please check out the [Allo documentation](https://cornell-zhang.github.io/allo) for installation instructions and tutorials.
If you encounter any problems, please feel free to open an [issue](https://github.com/cornell-zhang/allo/issues).


## Publications
Please refer to our [PLDI'24 paper](https://dl.acm.org/doi/10.1145/3656401) for more details. If you use Allo in your research, please use the following bibtex entry to cite us:
```bibtex
@article{chen2024allo,
    author = {Hongzheng Chen and Niansong Zhang and Shaojie Xiang and Zhichen Zeng and Mengjia Dai and Zhiru Zhang},
    title = {Allo: A Programming Model for Composable Accelerator Design},
    journal = {Proc. ACM Program. Lang.},
    year = {2024},
    month = {jun},
    url = {https://doi.org/10.1145/3656401},
    doi = {10.1145/3656401},
    articleno = {171},
    volume = {8},
    number = {PLDI},
    publisher = {Association for Computing Machinery},
    address = {New York, NY, USA},
    issue_date = {June 2024},
}
```

Please also consider citing the following papers if you utilize specific components of Allo:
* [AIE backend](https://cornell-zhang.github.io/allo/backends/aie.html): Jinming Zhuang, Shaojie Xiang, Hongzheng Chen, Niansong Zhang, Zhuoping Yang, Tony Mao, Zhiru Zhang, Peipei Zhou, "**ARIES: An Agile MLIR-Based Compilation Flow for Reconfigurable Devices with AI Engines**", *International Symposium on Field-Programmable Gate Arrays (FPGA)*, 2025. (Best paper nominee)
* [Equivalence checker](https://github.com/cornell-zhang/allo/blob/main/allo/verify.py): Louis-Noël Pouchet, Emily Tucker, Niansong Zhang, Hongzheng Chen, Debjit Pal, Gabriel Rodríguez, Zhiru Zhang, "**Formal Verification of Source-to-Source Transformations for HLS**", *International Symposium on Field-Programmable Gate Arrays (FPGA)*, 2024. (Best paper award)
* [LLM accelerator](https://github.com/cornell-zhang/allo/tree/main/examples): Hongzheng Chen, Jiahao Zhang, Yixiao Du, Shaojie Xiang, Zichao Yue, Niansong Zhang, Yaohui Cai, Zhiru Zhang, "**Understanding the Potential of FPGA-Based Spatial Acceleration for Large Language Model Inference**", *ACM Transactions on Reconfigurable Technology and Systems (TRETS)*, 2024. (FCCM’24 Journal Track)


## Related Projects
* Accelerator Programming Languages: [Exo](https://github.com/exo-lang/exo), [Halide](https://github.com/halide/Halide), [TVM](https://github.com/apache/tvm)
* Accelerator Design Languages: [Dahlia](https://github.com/cucapra/dahlia), [HeteroCL](https://github.com/cornell-zhang/heterocl), [PyLog](https://github.com/hst10/pylog), [Spatial](https://github.com/stanford-ppl/spatial)
* HLS Frameworks: [Stream-HLS](https://github.com/UCLA-VAST/Stream-HLS), [ScaleHLS](https://github.com/hanchenye/scalehls)
* Compiler Frameworks: [MLIR](https://mlir.llvm.org/)
