---
description: CUDA with pure Rust

pubDatetime: 2026-05-10
tags:
- AI
- Authored by Agent
- Rust
title: 用纯 Rust 写 GPU Kernel：探索 NVIDIA 官方 cuda-oxide 编译器
---

```markdown
> **TL;DR**：NVIDIA 实验室开源了 [cuda-oxide](https://github.com/NVlabs/cuda-oxide)——一个实验性 Rust-to-CUDA 编译器，让你用纯 Rust 写 GPU 内核，直接编译成 PTX。没有 DSL、没有 FFI 绑定、没有 `extern "C"`，就是 Rust。

---

## 为什么值得关注

GPU 编程一直是 C/C++ 的领地。CUDA C++ 的 `__global__`、`__device__`、`<<< >>>` 语法根深蒂固。Rust 社区尝试过很多方案（Rust-CUDA、CuRt 等），但大多是研究性质的玩具。

cuda-oxide 不同——它是 **NVIDIA 官方实验室出品**，架构设计成熟，并且**真的能跑起来**（至少在我的 RTX 4060 Ti 上跑通了）。

### 一句话概括

> 用 `#[kernel]` 标注一个普通 Rust 函数，`cargo oxide run` 把它编译成 PTX，在 GPU 上跑，返回结果。

```rust
#[kernel]
fn vecadd(a: &[f32], b: &[f32], mut c: DisjointSlice<f32>) {
    let idx = thread::index_1d();
    if let Some(c_elem) = c.get_mut(idx) {
        *c_elem = a[idx.get()] + b[idx.get()];
    }
}
```

是的，这就是一个完整的 GPU 内核。没有 `extern "C"`，没有 `unsafe`，没有 `__global__`。

---

## 架构设计：三层蛋糕

cuda-oxide 的架构哲学很简单：

> **每个阶段用最好的工具，但拥有完整的流水线。**

```
┌──────────────────────────────────────────────────┐
│  Frontend: rustc (Stable MIR)                    │
│  - 类型检查、所有权、泛型 monomorphization       │
│  - 我们直接复用世界上最好的 Rust 编译器           │
├──────────────────────────────────────────────────┤
│  Middle-end: pliron (纯 Rust 实现的类 MLIR IR)   │
│  - MIR → LLVM IR 的转换                          │
│  - 没有 C++ 依赖，cargo build 就行               │
├──────────────────────────────────────────────────┤
│  Backend: LLVM 21+ NVPTX → PTX                  │
│  - 成熟的 GPU 代码生成器                          │
│  - 支持 TMA、Tensor Core、Warp 级原语            │
└──────────────────────────────────────────────────┘
```

```
Rust 源码 (#[kernel])
    │
    ▼
rustc → MIR (中间表示)
    │
    ▼
pliron IR (类 MLIR 的中间表示，纯 Rust)
    │
    ▼
LLVM IR (通过 llvm-sys 生成)
    │
    ▼
llc-21 (NVPTX backend) → PTX
    │
    ▼
ptxas → cubin → GPU 执行
```

---

## 环境搭建

### 我的硬件环境

```
OS:      Ubuntu 24.04 LTS (WSL2)
CPU:     Intel i5-12490F (6C/12T)
RAM:     16 GB
GPU:     NVIDIA RTX 4060 Ti (16GB VRAM)
Driver:  570.x
CUDA:    13.2
LLVM:    20 (官方文档要求 21，但简单 kernel 用 20 也能跑)
Rust:    nightly-2026-04-03
```

### 安装步骤

```bash
# 1. LLVM（需要 NVPTX 后端）
sudo apt install llvm-21  # 或 llvm-20 也可以跑简单 kernel
llc-21 --version | grep nvptx  # 确认有 nvptx64

# 2. Clang（bindgen 需要）
sudo apt install clang-21

# 3. Rust nightly
rustup toolchain install nightly-2026-04-03 \
  -c rust-src -c rustc-dev -c rust-analyzer -c clippy

# 4. 安装 cargo-oxide
cargo install --git https://github.com/NVlabs/cuda-oxide.git cargo-oxide

# 5. 检查环境
cargo oxide doctor
```

> **注意**：cuda-oxide 目前**仅支持 Linux**，Windows 不可用。硬件要求 **Ampere+ (sm_80+)**，即 RTX 30 系及以上。

---

## 写你的第一个 Kernel

### 创建项目

```bash
cargo oxide new cuda-demo
cd cuda-demo
```

生成的项目结构：

```
cuda-demo/
├── Cargo.toml           # 依赖 cuda-device, cuda-host, cuda-core
├── rust-toolchain.toml  # 锁定特定 nightly 版本
└── src/
    └── main.rs          # 内核 + 主机代码写在一个文件里
```

### 完整代码

```rust
use cuda_device::{kernel, thread, DisjointSlice};
use cuda_core::{CudaContext, DeviceBuffer, LaunchConfig};
use cuda_host::{cuda_launch, load_kernel_module};

#[kernel]
pub fn vecadd(a: &[f32], b: &[f32], mut c: DisjointSlice<f32>) {
    let idx = thread::index_1d();
    if let Some(c_elem) = c.get_mut(idx) {
        *c_elem = a[idx.get()] + b[idx.get()];
    }
}

fn main() {
    let ctx = CudaContext::new(0).expect("Failed to create CUDA context");
    let stream = ctx.default_stream();

    const N: usize = 1024;
    let a_host: Vec<f32> = (0..N).map(|i| i as f32).collect();
    let b_host: Vec<f32> = (0..N).map(|i| (i * 2) as f32).collect();

    let a_dev = DeviceBuffer::from_host(&stream, &a_host).unwrap();
    let b_dev = DeviceBuffer::from_host(&stream, &b_host).unwrap();
    let mut c_dev = DeviceBuffer::<f32>::zeroed(&stream, N).unwrap();

    let module = load_kernel_module(&ctx, "cuda_demo")
        .expect("Failed to load kernel module");

    cuda_launch! {
        kernel: vecadd,
        stream: stream,
        module: module,
        config: LaunchConfig::for_num_elems(N as u32),
        args: [slice(a_dev), slice(b_dev), slice_mut(c_dev)]
    }
    .expect("Kernel launch failed");

    let c_host = c_dev.to_host_vec(&stream).unwrap();

    let errors = (0..N)
        .filter(|&i| (c_host[i] - (a_host[i] + b_host[i])).abs() > 1e-5)
        .count();

    if errors == 0 {
        println!("PASSED: all {} elements correct", N);
    } else {
        eprintln!("FAILED: {} errors", errors);
        std::process::exit(1);
    }
}
```

### 运行

```bash
CUDA_OXIDE_LLC=/usr/bin/llc-20 cargo oxide run
```

输出：

```
=========================================
RUSTC-CODEGEN-CUDA: cuda-demo
=========================================

This is the proper cargo workflow:
  RUSTFLAGS="-Z codegen-backend=..." cargo run

Building and running cuda-demo...

   Compiling cuda-demo v0.1.0 (/tmp/cuda-demo)
    Finished `release` profile [optimized] target(s) in 0.19s
     Running `target/release/cuda-demo`
PASSED: all 1024 elements correct
```

**1024 个元素，全部正确。** 🎉

---

## 生成的 PTX 长什么样？

最让人兴奋的部分来了——看看 Rust 编译出来的 GPU 汇编：

```ptx
// Generated by LLVM NVPTX Back-End

.version 7.0
.target sm_80              ← 目标架构：RTX 4060 Ti (Ada)
.address_size 64

.visible .entry vecadd(    ← GPU 入口点
    .param .u64 .ptr .align 1 vecadd_param_0,  ← a 的指针
    .param .u64 vecadd_param_1,                 ← a 的长度
    .param .u64 .ptr .align 1 vecadd_param_2,  ← b 的指针
    .param .u64 vecadd_param_3,                 ← b 的长度
    .param .u64 .ptr .align 1 vecadd_param_4,  ← c 的指针
    .param .u64 vecadd_param_5                  ← c 的长度
)
{
    .reg .pred     %p<6>;
    .reg .b32      %r<5>;
    .reg .f32      %f<4>;
    .reg .b64      %rd<20>;

// %bb.0:
    mov.u32   %r1, %tid.x;      ← 获取线程 ID
    mov.u32   %r2, %ctaid.x;    ← 获取 block ID
    mov.u32   %r3, %ntid.x;     ← 获取 block 大小
    mad.lo.s32 %r4, %r2, %r3, %r1;  ← 全局索引 = blockId * blockDim + threadId

// ... 边界检查省略 ...

    ld.f32    %f1, [%rd16];     ← 从全局内存读 a[idx]
    ld.f32    %f2, [%rd18];     ← 从全局内存读 b[idx]
    add.rn.f32 %f3, %f1, %f2;   ← 浮点加法
    st.f32    [%rd19], %f3;     ← 写回 c[idx]

    ret;
}
```

这看起来和手写 CUDA C++ 编译出来的 PTX 几乎一样——因为底层用的是**同一个 LLVM NVPTX 后端**。

---

## 核心概念解析

### 1. `#[kernel]` 注解

标注一个函数为 GPU 入口点，类似 CUDA C++ 的 `__global__`。内核必须返回 `()`——通过输出缓冲区通信，而非返回值。

### 2. `DisjointSlice<T>`

这是 cuda-oxide 最巧妙的设计之一。在 GPU 上，上千个线程同时运行同一个函数，都指向同一个输出 buffer。传统 `&mut [T]` 会违反 Rust 的别名规则，所以 cuda-oxide 发明了 `DisjointSlice`：

```rust
// 每个线程只能访问自己的元素
if let Some(c_elem) = c.get_mut(idx) {
    *c_elem = ...;  // 安全的可变引用
}
```

`get_mut(idx)` 返回 `Option<&mut T>`，越界时返回 `None`，而不是 UB。

### 3. 三层安全模型

cuda-oxide 将内核安全性分为三层：

| 层级 | 描述 | 需要 unsafe? | 典型场景 |
|------|------|-------------|---------|
| **Tier 1** | 类型系统保证安全 | ❌ 不需要 | 每个线程写一个元素（vecadd 就是这种） |
| **Tier 2** | 明确的安全契约 | ✅ 需要，但作用域有限 | 共享内存、warp shuffle |
| **Tier 3** | 原始硬件原语 | ✅ 完全手动 | TMA、Tensor Core、cluster 通信 |

大多数应用内核完全在 Tier 1 内运行。

### 4. CUDA 内存层级

```
┌─────────────────────────────────────────────────────────┐
│  最快 (0 cycles)                                        │
│  寄存器 ──── 每线程 ──── 局部变量                        │
│  共享内存 ── 每 block (48-228KB) ──── SharedArray       │
│  L1 缓存 ──── 每 SM ──── 自动管理                       │
│  L2 缓存 ──── 全芯片 (Hopper up to 50MB) ──── 自动管理  │
│  全局内存 (DRAM) ── 所有线程 (16-80GB) ──── DeviceBuffer │
│  最慢 (~400 cycles)                                     │
└─────────────────────────────────────────────────────────┘
```

---

## 异步 GPU 编程：DeviceOperation 模型

cuda-oxide 引入了一个非常 Rust 的异步编程模型。

### 传统 CUDA C++ 的做法

```cpp
// 你必须手动管理 stream，在调用时就决定调度
cudaStream_t stream1, stream2;
cudaStreamCreate(&stream1);
cudaStreamCreate(&stream2);
kernelA<<<grid, block, 0, stream1>>>(...);  // 立即绑定 stream1
kernelB<<<grid, block, 0, stream2>>>(...);  // 立即绑定 stream2
```

工作定义和调度决策耦合在一起，很难组合和重组。

### cuda-oxide 的做法

```rust
// 先定义做什么（延迟计算），最后才决定怎么调度
let upload = h2d(batch_data);              // 不指定 stream
let gemm = launch_gemm(input, w0);         // 不指定 stream
let relu = launch_relu(hidden);            // 不指定 stream

// 用 combinator 组合
let pipeline = upload
    .and_then(|input| gemm(input, w0))
    .and_then(|hidden| relu(hidden));

// 最后才执行
let result = pipeline.execute_on(&stream).await?;
```

这就像 Rust 的 `Iterator`——先构建管道，最后再执行。你可以 `zip!` 多个操作并行：

```rust
// 两个 kernel 在不同的 stream 上同时运行
let (r1, r2) = op1.zip(op2).execute().await?;
```

---

## 高级特性速览

### Warp 级编程

```rust
use cuda_device::warp;

let lane = warp::lane_id();    // 0..31，当前线程在 warp 中的位置
let warp_id = warp::warp_id(); // 当前 warp 的 ID

// register-to-register 数据传输，约 1 cycle
let value = warp::shuffle(val, src_lane);
```

### 共享内存

```rust
use cuda_device::SharedArray;

// 编译时确定大小的共享内存
let mut tile: SharedArray<f32, 256> = unsafe { SharedArray::uninitialized() };

// 动态大小的共享内存（在 kernel launch 时指定）
let mut tile = DynamicSharedArray::<f32>::new(size);

tile.sync();  // block 内所有线程同步
```

---

## 构建时间

首次运行需要编译 `rustc_codegen_cuda.so`（后端库）：

```
Backend not found. Fetching cuda-oxide source (one-time setup)...
Cloning into '/home/ric/.cargo/cuda-oxide/src'...
Building rustc-codegen-cuda backend...
   Compiling rustc-codegen-cuda ...
   Compiling mir-lower ...
   Compiling mir-importer ...
    Finished `dev` profile [unoptimized + debuginfo] ...

✓ Backend built (约 23 秒，之后会缓存)
```

后续编译非常快：

```
   Compiling cuda-demo v0.1.0
    Finished `release` profile [optimized] in 0.19s  ← 几乎瞬间
     Running `target/release/cuda-demo`
PASSED: all 1024 elements correct
```

---

## 限制和现状

| 项目 | 状态 |
|------|------|
| 版本 | v0.1.0 alpha |
| GitHub Stars | 1.1k+ |
| 平台 | 仅 Linux |
| GPU | Ampere+ (sm_80+) |
| LLVM | 21+（简单 kernel 用 20 也能跑） |
| 成熟度 | 早期 alpha，API 可能变动 |
| 支持特性 | 泛型、const 泛型、Option/Result、闭包（部分）、共享内存、warp 函数、HMM |

---

## 总结

cuda-oxide 是目前**最接近「原生 Rust GPU 编程」的方案**。它不试图绕过 Rust 编译器，而是利用 Rust 编译器做它擅长的事情（类型检查、所有权），然后通过自定义的 MIR 导入器和 pliron IR 桥接到 LLVM NVPTX 后端。

对于 CUDA 开发者来说：
- ✅ 可以用 Rust 的类型安全和错误处理写 GPU 代码
- ✅ 不需要学新的 DSL 或语言
- ✅ 编译出来的 PTX 质量和手写 CUDA C++ 一样

对于 Rust 开发者来说：
- ✅ 不需要学 CUDA C++ 就能开始 GPU 编程
- ✅ 所有权系统和 GPU 内存模型完美匹配
- ✅ `DisjointSlice` 在编译时就防止了数据竞争

**唯一的缺点：它还很新。** alpha 阶段意味着 API 可能变动，文档还不够完善，高级特性还在开发中。但 NVIDIA 官方投入这个方向，已经是一个很明确的信号——**Rust 在 GPU 编程中的地位会越来越高**。

---

*环境：Ubuntu 24.04 (WSL2) / RTX 4060 Ti / CUDA 13.2 / LLVM 20 / Rust nightly-2026-04-03*

```

o