# Instruction-Level Attribution Extension to gpuFI-4

[![Paper](https://img.shields.io/badge/Paper-ScienceDirect-4b83c3)](https://www.sciencedirect.com/science/article/pii/S0167739X26000063)
[![DOI](https://img.shields.io/badge/DOI-10.1016%2Fj.future.2026.108372-blue)](https://doi.org/10.1016/j.future.2026.108372)

This repository is an open-source artifact accompanying our paper:

**BiD-Accel: Accelerated bidimensional input-aware SDC vulnerability assessment for GPU static instructions**  
Zhenyu Qian, Lianguo Wang, Pengfei Zhang, Jianing Rao.  
*Future Generation Computer Systems*, Volume 180, 2026, Article 108372.  
DOI: 10.1016/j.future.2026.108372  
Paper: https://www.sciencedirect.com/science/article/pii/S0167739X26000063  
Code: https://github.com/zhenyu818/gpufi-instinject

It contains an extended build of **[gpuFI-4](https://github.com/caldi-uoa/gpuFI-4.git)** that adds **instruction-level attribution**: every injected fault is mapped to a specific instruction. This enables fine-grained analysis of the sensitivity of **static instructions** and their contribution to outcome categories (**Masked**, **SDC**, **DUE**). ⚡

> **Upstream note:** gpuFI-4 is built on the GPGPU-Sim/Accel-Sim ecosystem (e.g., GPGPU-Sim 4.0). Please respect the licenses and citation requirements of upstream projects.

> **Default target:** RTX 2060–class GPU (SM75). To target other GPUs, edit `gpgpusim.config` in the repo root and set `GPU_ARCH` in `inst_fault_inject_exp.sh` accordingly. 🛠️

---

## Publication / Citation 📄

If you use this repository in your research, please cite our paper:

```bibtex
@article{QIAN2026108372,
  title = {BiD-Accel: Accelerated bidimensional input-aware SDC vulnerability assessment for GPU static instructions},
  journal = {Future Generation Computer Systems},
  volume = {180},
  pages = {108372},
  year = {2026},
  issn = {0167-739X},
  doi = {10.1016/j.future.2026.108372},
  url = {https://www.sciencedirect.com/science/article/pii/S0167739X26000063},
  author = {Zhenyu Qian and Lianguo Wang and Pengfei Zhang and Jianing Rao},
  keywords = {GPU, Soft error, Silent data corruptions, Static instruction vulnerability, Fault injection, Input feature}
}
````

---

## Features ✨

* **Instruction-level fault attribution**

  * Binds each WRITER effect to the responsible instruction (kernel, source line, instruction text).
  * Aggregates outcomes per instruction and per source (WRITER).
  * Optional register-name attribution via `INJ_PARAMS`.

* **Automated campaigns**

  * Build, (optional) golden-result generation, campaign setup, execution, progress tracking, and CSV export.

* **Containerized workflow**

  * Runs inside the published Accel-Sim Docker image for reproducibility. 📦

---

## Quick Start 🚀

1. **Pull the container image**

   ```bash
   docker pull accelsim/ubuntu-18.04_cuda-11:latest
   ```

2. **Launch the container and mount this repo (example)**

   ```bash
   docker run --rm -it \
     -v "$PWD":/workspace \
     -w /workspace \
     accelsim/ubuntu-18.04_cuda-11:latest bash
   ```

3. **Run an experiment**

   ```bash
   bash inst_fault_inject_exp.sh
   ```

---

## Repository Layout 🗂️

* `inst_fault_inject_exp.sh`
  Orchestrates building, optional golden-result generation, campaign setup, progress tracking, and CSV export via `analysis_fault.py`.

* `campaign_exec.sh`, `campaign_profile.sh`
  Injection runner and profiling helper; configure injection parameters and collect per-run logs/effects.

* `analysis_fault.py`
  Parses `inst_exec.log` and writes per-instruction CSV summaries to `test_result/`.

* `test_apps/`

  * Each subfolder name is an application (e.g., `Pathfinder`, `Stencil1D`).
  * `result_gen/`: Programs that generate **Golden Results**. When `DO_RESULT_GEN=1`, outputs go to `test_apps/<app>/result/`.
  * `inject_app/`: Programs used during fault injection to compare against Golden Results.
  * `size_list.txt`: One command-line parameter set per line. Both `inject_app` and `result_gen` may contain multiple programs to support multiple input sets; **program names in both folders must match**.

---

## Configuration (via `inst_fault_inject_exp.sh`) ⚙️

| Variable                | Description                                                             | Example      |
| ----------------------- | ----------------------------------------------------------------------- | ------------ |
| `TEST_APP_NAME`         | Application name (matches a folder under `test_apps/`).                 | `Pathfinder` |
| `COMPONENT_SET`         | Components to inject into (colon-separated list; see map below).        | `0:1`        |
| `INJECT_BIT_FLIP_COUNT` | Number of bits flipped per injection.                                   | `2`          |
| `RUN_PER_EPOCH`         | Number of injections to execute in this round.                          | `1000`       |
| `GPU_ARCH`              | Target GPU architecture for compilation.                                | `sm_75`      |
| `DO_BUILD`              | Build before running (enable on first run).                             | `1`          |
| `DO_RESULT_GEN`         | Generate Golden Results (enable on first run or when adding a new app). | `1`          |

**Component map (for `COMPONENT_SET`):**

| Code | Component    |
| :--: | ------------ |
|   0  | `RF`         |
|   1  | `local_mem`  |
|   2  | `shared_mem` |
|   3  | `L1D_cache`  |
|   4  | `L1C_cache`  |
|   5  | `L1T_cache`  |
|   6  | `L2_cache`   |

> Example: `COMPONENT_SET=0:1` flips both **RF** and **local_mem**.

---

## Running an Experiment 🧪

1. **Start the container** (see *Quick Start*).

2. **Configure `inst_fault_inject_exp.sh`**
   Set `TEST_APP_NAME`, `COMPONENT_SET`, `INJECT_BIT_FLIP_COUNT`, `RUN_PER_EPOCH`, `GPU_ARCH`, `DO_BUILD`, `DO_RESULT_GEN`.

3. **Verify inputs**
   Ensure your app’s `size_list.txt` contains one parameter set per line.

4. **Launch**

   ```bash
   ./inst_fault_inject_exp.sh
   ```

5. **Review results**
   Live progress is printed; summaries are exported to `test_result/` as:

   ```text
   test_result_<app>_<test>_<components>_<bitflip>.csv
   ```

---

## Outputs & Logs 📊

* `inst_exec.log` — Aggregated run log with effects, parameters, and results.
* `test_result/` — Per-instruction CSV summaries for the current run.
* `logs*/` — Per-batch temporary output (cleaned depending on script options).

---

## What’s New vs. gpuFI-4 🆕

* **Instruction-level attribution for fault effects**

  * Binds each WRITER effect to the responsible instruction (kernel, source line, instruction text), enabling fine-grained sensitivity analysis.
  * Aggregates outcomes per instruction and per source (WRITER), with optional register-name attribution from `INJ_PARAMS`.

---

## Notes & Tips 📝

* With `DO_RESULT_GEN=1`, programs in `result_gen/` automatically produce Golden Results under `test_apps/<app>/result/`.
* The injector can target explicit **PTX register names** (see `campaign_exec.sh`) for fine-grained, register-level injections.
* To target GPUs other than SM75, update both `gpgpusim.config` and `GPU_ARCH`.

---

## Upstream Projects & Attribution 🔗

This repository is **based on** and **extends** the following upstream projects:

* **gpuFI-4** (fault injection framework): [https://github.com/caldi-uoa/gpuFI-4.git](https://github.com/caldi-uoa/gpuFI-4.git)
* **GPGPU-Sim / Accel-Sim ecosystem** (GPU simulation infrastructure, including GPGPU-Sim 4.0):
  [https://github.com/gpgpu-sim/gpgpu-sim_distribution](https://github.com/gpgpu-sim/gpgpu-sim_distribution)
  [https://github.com/accel-sim/accel-sim-framework](https://github.com/accel-sim/accel-sim-framework)

Please follow upstream **licenses**, keep their copyright headers intact,
and cite upstream papers/tools when required by their documentation.

---

## Acknowledgments 🙌

Built on top of **gpuFI-4** and the broader **GPGPU-Sim / Accel-Sim** communities and tooling. Thanks to the authors and maintainers for their open-source contributions.

---

## License ⚖️

This repository contains code derived from upstream projects. Please check:

* the `LICENSE` file in this repository (if present), and
* the licenses and citation requirements of upstream dependencies linked above.

If you redistribute or publish results, ensure compliance with all applicable upstream licenses.
