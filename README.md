# ReFun: Identifying and Extracting Functions from EVM Bytecode

ReFun is a research artifact for recovering functions from Ethereum Virtual Machine (EVM) runtime bytecode. It disassembles bytecode, identifies function boundaries, extracts the corresponding basic blocks, and writes a final per-function representation that can facilitate downstream tasks.



## Contents

- [1. Artifact at a Glance](#1-artifact-at-a-glance)
- [2. Capabilities](#2-capabilities)
- [3. Artifact Structure](#3-artifact-structure)
- [4. Requirements and Installation](#4-requirements-and-installation)
- [5. Quick Start](#5-quick-start)
- [6. Full-Dataset Evaluation](#6-full-dataset-evaluation)
- [7. Usage Reference](#7-usage-reference)
- [8. Output and Statistics](#8-output-and-statistics)
- [9. Evaluation and Reproducibility](#9-evaluation-and-reproducibility)
- [10. Reuse and Extension](#10-reuse-and-extension)
- [11. Troubleshooting and Limitations](#11-troubleshooting-and-limitations)
- [12. Data and Citation](#12-data-and-citation)
- [13. License](#13-license)

## 1. Artifact at a Glance

| Item | Description |
|---|---|
| Artifact type | Python research software, trained model, and EVM bytecode dataset |
| Primary platform | 64-bit Linux; Ubuntu desktop is strongly recommended |
| Recommended setup | Anaconda/Miniconda with the supplied `environment.yml` |
| Recommended IDE | Spyder, launched from the `refun` Conda environment |
| User interfaces | Tkinter GUI, single-run CLI, and batch CLI |
| Hardware | CPU only; no GPU or specialized hardware is required |
| Network access | Not required after dependencies have been installed |
| Quick dataset | 20 files in [`QuickStart/`](QuickStart/) |
| Wild dataset | 4,700 randomly collected Ethereum compiled contract files in [`inputs(wild contracts)/`](inputs%28wild%20contracts%29/) |
| Persistent batch output | Final `*_functions.txt` files and `result_statistics.txt` only |
| Long-running evaluation | Full batch processing is expected to take tens of hours |

The artifact can be exercised at three levels:

1. **Sanity test:** analyze the supplied `input_bytecode` file.
2. **Quick evaluation:** analyze the 20 files in `QuickStart/`.
3. **Complete evaluation:** analyze all 4,700 files in `inputs(wild contracts)/`.

The first two levels are sufficient to check installation, functionality, output structure, and adaptation to multiple inputs without committing to the
long full-dataset run.

## 2. Capabilities

ReFun provides the following functionality:

- validates and disassembles EVM runtime bytecode;
- constructs the basic-block and control-flow representation used by the recovery analysis;
- uses public function selectors as stable IDs and entry clues, then reconstructs each function's boundary, path, and basic blocks;
- detects candidate private-function call/return boundaries;
- uses deterministic analysis and the bundled random-forest model to classify private-function candidates;
- traverses from a public function's selector-associated block or a private function's recovered entry block;
- bounds path exploration to prevent deterministic loops from running indefinitely;
- extracts and groups the basic blocks belonging to each recovered function;
- emits an explicit path, entry block, end block, and completion status for every recovered function; and
- provides a user-friendly GUI that enables intuitive one-click analysis, supports large-scale batch processing and result summarization, and frees users from tedious scripting and implementation details.

## 3. Artifact Structure

### 3.1 User-facing files

| Path | Purpose |
|---|---|
| [`refun_gui.py`](refun_gui.py) | GUI for single and batch analysis |
| [`run_ReFun.py`](run_ReFun.py) | Single-input command-line runner |
| [`run_ReFun_auto.py`](run_ReFun_auto.py) | Folder-based batch runner |
| [`result_statistics.py`](result_statistics.py) | Aggregate statistics from final batch results |
| [`input_bytecode`](input_bytecode) | Default single-run input |
| [`output_functions_final`](output_functions_final) | Default single-run result |
| [`QuickStart/`](QuickStart/) | 20-file rapid-evaluation dataset |
| [`inputs(wild contracts)/`](inputs%28wild%20contracts%29/) | 4,700-file wild dataset |
| [`outputs(extracted functions)/`](outputs%28extracted%20functions%29/) | Default batch-result directory |

### 3.2 Core components

| Component | Responsibility |
|---|---|
| `disassembler.py`, `bytecodeblock.py`, `bytecodes.py`, `opcodes.py` | Bytecode disassembly and EVM instruction/block representation |
| `CG.py`, `EVM_modeling.py`, `EVM_for_path_exploration.py` | Control-flow and symbolic EVM support |
| `likely_CR_site_pairings_analyzer.py` | Produces candidate call/return pairs |
| `stage__hub.py`, `stage__a.py` ... `stage__k.py` | Jumps contexts |
| `function_signature_hash_extractor.py`                       | Extracts selector used to seed public-function recovery      |
| `path_through_function.py`                                   | Performs bounded function-path traversal                     |
| `function_block_extractor.py`                                | Writes final per-function basic blocks                       |
| `random_forest_model.pkl`                                    | Bundled private-function classifier                          |

## 4. Requirements and Installation

### 4.1 Recommended environment

The recommended configuration is:

- a 64-bit Ubuntu desktop;
- Anaconda;
- Python 3.11;
- Spyder opened from the ReFun Conda environment; and
- a graphical desktop session when using the GUI.

ReFun is CPU-based. It does not require a GPU, Ethereum node, database, Docker, or network access during analysis.

### 4.2 Install with Conda

From the project root:

```bash
cd /path/to/ReFun
conda env create -f environment.yml
conda activate refun
```

For an existing environment:

```bash
conda env update -n refun -f environment.yml --prune
conda activate refun
```

Verify the installation:

```bash
python -c "import joblib, pandas, sklearn, tkinter; print('Dependencies OK'); print('scikit-learn', sklearn.__version__)"
```

Expected:

```text
Dependencies OK
scikit-learn 1.3.2
```

The model is distributed with the artifact and was created with scikit-learn 1.3.2. Using the pinned version avoids model-compatibility warnings and unintended prediction differences.

### 4.3 Recommended Spyder workflow

```bash
conda activate refun
spyder
```

In Spyder:

1. open `refun_gui.py`;
2. set the working directory to the ReFun project root;
3. click **Run file**; and
4. interact with the ReFun window.

Ubuntu + Anaconda + Spyder is the recommended setup because the interpreter, scientific dependencies, editor, console, and graphical integration are
available in one environment.

### 4.4 Alternative pip installation

```bash
cd /path/to/ReFun
python3.11 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

If Tkinter is unavailable with system Python:

```bash
sudo apt update
sudo apt install python3-tk
```

Conda remains preferable for GUI evaluation because its Python and Tk packages are managed together.

## 5. Quick Start

Reviewers can try the single-input sanity test first and then use the 20-file QuickStart set. The full dataset is not necessary for checking basic
functionality.

### 5.1 Single-input sanity test

Activate the environment and run:

```bash
cd /path/to/ReFun
conda activate refun
python run_ReFun.py
```

The result is written to:

```text
output_functions_final
```

Inspect its header:

```bash
head -n 6 output_functions_final
```

For the packaged `input_bytecode`, the expected function counts are:

```text
PUBLIC_FUNCTIONS 3
PRIVATE_FUNCTIONS 1
```

`ANALYSIS_TIME_SECONDS` is machine-dependent. Function counts and recovered blocks should remain stable when using the supplied environment.

### 5.2 Single analysis through the GUI

Launch:

```bash
python refun_gui.py
```

The GUI loads `input_bytecode` automatically. On the **Single analysis** tab, click **Run single analysis**. The result appears in the output panel and is
also saved as `output_functions_final`. The GUI additionally accepts pasted hexadecimal bytecode, optional whitespace, an optional `0x` prefix, or a selected bytecode file.

### 5.3 Batch evaluation with 20 inputs

Command-line workflow:

```bash
mkdir -p "outputs(extracted functions)/quickstart"
python run_ReFun_auto.py \
  --input-folder QuickStart \
  --output-folder "outputs(extracted functions)/quickstart"
```

The console reports per-file progress and finishes with `[COMPLETE]`, `[STATISTICS]`, and `[SUMMARY]` records.

GUI workflow:

1. open the **Batch analysis** tab;
2. select `QuickStart` as the input folder;
3. select `outputs(extracted functions)/quickstart` as the output folder;
4. click **Run batch analysis**; and
5. wait for the completion dialog.

The GUI displays the aggregate statistics and previews the three most recent results. Complete results remain in
`outputs(extracted functions)/quickstart`.

Expected persistent files:

```text
outputs(extracted functions)/quickstart/<input-stem>_functions.txt
outputs(extracted functions)/quickstart/result_statistics.txt
```

No persistent `*_result.txt` intermediate files are produced.

## 6. Full-Dataset Evaluation

The `inputs(wild contracts)/` directory contains 4,700 compiled-contract bytecode files randomly collected from the Ethereum blockchain.

Run with the default folders:

```bash
python run_ReFun_auto.py
```

Equivalent explicit command:

```bash
python run_ReFun_auto.py \
  --input-folder "inputs(wild contracts)" \
  --output-folder "outputs(extracted functions)" \
  --input-bytecode-file input_bytecode
```

For an unattended Ubuntu run, `tmux` and a log file are useful:

```bash
tmux new -s refun
conda activate refun
cd /path/to/ReFun
python -u run_ReFun_auto.py 2>&1 | tee full_run.log
```

After completion:

```bash
cat "outputs(extracted functions)/result_statistics.txt"
find "outputs(extracted functions)" -maxdepth 1 -name '*_functions.txt' | wc -l
```

The GUI may also run the full dataset and displays progress continuously. Clicking **Stop analysis** terminates the active analysis process tree; already completed result files are retained. If a run is stopped before the final statistics step, regenerate the summary with:

```bash
python result_statistics.py --output-folder "outputs(extracted functions)"
```

Batch inputs are preserved by default. The optional `--delete-inputs` flag is destructive and should not be used during artifact evaluation.

## 7. Usage Reference

Run commands from the project root.

### 7.1 Single-run CLI

Place one uninterrupted, even-length hexadecimal bytecode string in `input_bytecode`, without a `0x` prefix, and run:

```bash
python run_ReFun.py
```

The next successful single run overwrites `output_functions_final`.

### 7.2 Batch CLI

```bash
python run_ReFun_auto.py \
  --input-folder /path/to/bytecode \
  --output-folder /path/to/results
```

| Option | Default | Description |
|---|---|---|
| `--input-folder` | `./inputs(wild contracts)` | Folder containing bytecode files |
| `--output-folder` | `./outputs(extracted functions)` | Final result destination |
| `--input-bytecode-file` | `./input_bytecode` | Shared scratch input used by the pipeline |
| `--delete-inputs` | disabled | Delete each source after successful processing |

The batch reader scans each file line by line and accepts the first line that contains an even number of hexadecimal characters. A leading `0x` is allowed.
Files without a valid bytecode line are reported with `[SKIP]`. Use unique file stems: `contract.txt` and `contract.hex` both map to `contract_functions.txt`.

### 7.3 GUI controls

| Control | Behavior |
|---|---|
| **Load bytecode file** | Loads a file into the single-input editor |
| **Run single analysis** | Calls `run_ReFun.py` and displays its final result |
| **Input folder / Output folder** | Selects the batch source and destination |
| **Run batch analysis** | Calls `run_ReFun_auto.py` and updates the progress bar |
| **Stop analysis** | Terminates the active runner and its child processes |

The batch output panel retains previews of the three most recent results. A large preview may be truncated, but the saved file is complete.

### 7.4 Important working files

`input_bytecode` and `disassembly_result.txt` are shared scratch files and are overwritten during analysis. Do not run two analyses concurrently from the same project checkout. For parallel experiments, create one independent project copy and output folder per worker.

## 8. Output and Statistics

### 8.1 Final function file

Each final file begins with:

```text
REFUN FINAL FUNCTION BASIC BLOCKS
SOURCE <source-file>
PUBLIC_FUNCTIONS <count>
PRIVATE_FUNCTIONS <count>
ANALYSIS_TIME_SECONDS <seconds>
======================================================================
```

Each function record follows this structure:

```text
FUNCTION public 0x12345678
ENTRY block_12
END block_34
STATUS complete
PATH block_12 -> block_18 -> block_34
BEGIN_BLOCKS

block_12
...

END_FUNCTION
```

| Field | Meaning |
|---|---|
| `FUNCTION` | Visibility and function ID |
| `ENTRY` | Entry block used for traversal |
| `END` | Expected or observed final block |
| `STATUS` | `complete` or `partial` |
| `PATH` | Ordered basic-block path |
| `BEGIN_BLOCKS` ... `END_FUNCTION` | Recovered block contents |
| `ERROR` | Extraction error, when one was caught |

Public functions use selectors such as `0x715018a6` as stable result IDs. However, locating a selector is only the starting clue: ReFun must still find the selector-associated entry block, traverse control flow, reconstruct the function boundary, and extract its basic blocks. Private functions use per-file labels such as `private_1`. `STATUS partial` means ReFun retained the blocks it could recover but did not reach the expected end condition or encountered an extraction exception. It does not imply that the entire input-file analysis failed.

### 8.2 Statistics

Batch processing automatically creates `result_statistics.txt`. It can be regenerated at any time:

```bash
python result_statistics.py --output-folder /path/to/results
```

The statistics include:

- result-file count;
- total extracted functions;
- correctly extracted functions;
- overall function-extraction precision; and
- average per-file analysis time.

## 9. Evaluation and Reproducibility

### 9.1 Suggested evaluator sequence

1. create and activate the supplied Conda environment;
2. verify the dependency versions;
3. run the packaged single-input sanity test;
4. confirm the expected 3-public/1-private header;
5. process the 20-file QuickStart set;
6. inspect several final function files and `result_statistics.txt`;
7. add or modify one bytecode input and confirm that a new result is produced;
8. test the GUI progress and stop controls if a desktop is available; and
9. optionally start the complete 4,700-file evaluation.

### 9.2 Observable claims

| Capability | Evaluation action | Evidence |
|---|---|---|
| EVM bytecode is transformed into per-function output | Run `run_ReFun.py` | `output_functions_final` |
| Public function boundaries are reconstructed from selector-associated entry clues | Inspect `FUNCTION public` records | `ENTRY`, `END`, `PATH`, status, and extracted blocks |
| Private functions contain recovered entry/end paths | Inspect `FUNCTION private` records | `ENTRY`, `END`, `PATH`, and block contents |
| Path exploration terminates on cyclic control flow | Run the QuickStart batch | Batch completion and function statuses |
| Multiple contracts are processed automatically | Run `run_ReFun_auto.py` | Per-input final files and progress records |
| Aggregate extraction quality and timing are generated | Inspect `result_statistics.txt` | Total extracted/correctly extracted functions, precision, and average time |
| New inputs can be analyzed without source changes | Add a valid bytecode file | A new `<stem>_functions.txt` result |



### 9.3 Expected variation

Function IDs, output structure, recovered blocks, and counting rules should be stable in the supplied environment. Wall-clock time varies with hardware and system load, so exact timing is not a reproducibility requirement. Interrupted runs contain only results completed before termination.

## 10. Reuse and Extension

### 10.1 Analyze a new dataset

No source change is required:

```bash
python run_ReFun_auto.py \
  --input-folder /path/to/new_bytecode \
  --output-folder /path/to/new_results
```

Place one contract's EVM bytecode in each file and use unique filenames.

### 10.2 Use ReFun from Python

```python
from pathlib import Path
from run_ReFun_auto import batch_process

results = batch_process(
    input_folder=Path("/path/to/new_bytecode"),
    result_folder=Path("/path/to/new_results"),
    input_bytecode_file=Path("input_bytecode"),
)
```

Statistics can also be consumed programmatically:

```python
from result_statistics import collect_statistics

statistics = collect_statistics("/path/to/new_results")
print(statistics["extracted_function_count"])
print(statistics["correctly_extracted_function_count"])
print(statistics["function_extraction_precision_percent"])
print(statistics["average_time_seconds"])
```

The final output is plain UTF-8 text with explicit record markers, making it suitable for downstream parsers, graph construction, or empirical studies.

### 10.3 Extend private-function recovery

Private-function hints are separated into `stage__a.py` through `stage__k.py` and coordinated by `stage__hub.py`.

To change the feature set:

1. implement or revise a stage;
2. update `HintProvider` in `stage__hub.py`;
3. update feature construction in `single_called_function_recover.py`;
4. retrain `random_forest_model.pkl` if the model schema changes; and
5. add regression inputs to `QuickStart/`.

The runtime feature names, ordering, and trained model must stay consistent.

### 10.4 Adapt traversal or validation

- Modify `path_through_function.py` to experiment with another path policy.
- Modify `function_block_extractor.py` to change the final representation.
- Modify `PRIVATE_MARKERS` or `_parse_final_result()` in
  `result_statistics.py` when a stronger ground-truth oracle becomes available.

Downstream parsers should identify named fields and records between `FUNCTION` and `END_FUNCTION`, rather than relying on fixed line numbers.

## 11. Troubleshooting and Limitations

### 11.1 Common problems

| Problem | Cause and solution |
|---|---|
| `ModuleNotFoundError` | Activate `refun` and run `conda env update -n refun -f environment.yml --prune`. Launch Spyder from the activated environment. |
| `InconsistentVersionWarning` | Install `scikit-learn=1.3.2` and `joblib=1.3.2`, matching the bundled model. |
| GUI does not open | The machine may be headless or lack Tk. Use the CLI, or install `python3-tk` when using system Python. |
| `no display name and no $DISPLAY` | A graphical session is unavailable. Use `run_ReFun.py` or `run_ReFun_auto.py`. |
| `no valid even-length hexadecimal bytecode line found` | Select a bytecode-only folder and ensure each input has a valid even-length hexadecimal line. |
| GUI tries to analyze `.py` files | The project root was selected by mistake. Select `QuickStart/`, `inputs(wild contracts)/`, or another bytecode directory. |
| Analysis appears slow | Contract complexity varies. Start with the single input or QuickStart set; stop the GUI run or press `Ctrl-C` if necessary. |
| Statistics contain no timing data | Regenerate final results with the current runner, then rerun `result_statistics.py`. |
| Old results affect file counts | Use a new empty output directory for each experiment. ReFun does not delete unrelated final files. |
| Permission denied | Move the project and result directory to a writable location; root privileges are not required. |

### 11.2 Known limitations

- ReFun accepts hexadecimal EVM bytecode; it does not compile Solidity source.
- Dynamic jumps, obfuscation, unusual compiler idioms, malformed bytecode, or unsupported EVM behavior may produce partial paths.
- The classifier depends on the current feature schema and scikit-learn 1.3.2.
- A single checkout is not safe for concurrent runs because scratch files are shared.
- Full-dataset runtime varies substantially across contracts and machines.
- The GUI requires a graphical desktop; command-line analysis works headlessly.



## 12. License

This artifact is released under the MIT License. See the `LICENSE` file for details.
