
# Cetus Regression Test Suite

This regression test suite helps verify the correctness and behavior of the Cetus source-to-source compiler by automating the process of preprocessing, semantic checking, Cetus transformation, and comparing the output against a set of ground truth files.

---

## Table of Contents

* [Prerequisites](#prerequisites)
* [Setting Up the Environment](#setting-up-the-environment)
* [Understanding the Code](#understanding-the-code)
* [Compilation](#compilation)
* [Running the Tests](#running-the-tests)
  * [Test Modes](#test-modes)
  * [Expected Parallelization Behavior](#expected-parallelization-behavior)
  * [Example Usage](#example-usage)
* [Output and Logging](#output-and-logging)
* [Test Directory Structure](#test-directory-structure)

---

## Prerequisites

Before you begin, ensure you have the following installed on your system:

- **C Compiler:** A C compiler (like GCC or Clang) is required to compile the test suite itself.
- **Clang:** The `clang` preprocessor is used to preprocess the input C files. The path to your `clang` executable is defined in `helper_tests.h` as `CLANG_PATH`. By default, it's set to `/opt/homebrew/opt/llvm/bin/clang`. **You'll likely need to adjust this path** to match your `clang` installation (e.g., `/usr/bin/clang` on Linux, or another Homebrew path if you're on macOS).
- **Cetus Compiler:** The Cetus compiler executable. The path to the Cetus executable is defined in `helper_tests.h` as `CETUS_PATH`. By default, it's `./cetus`, assuming Cetus is in the same directory as the compiled test runner. **Adjust this path** if your Cetus executable is located elsewhere.
- **`diff` utility:** The `diff` command-line utility is used to compare files. This is typically pre-installed on Unix-like systems.
- **`mkdir` utility:** Used for creating directories. Also standard on Unix-like systems.

---

## Setting Up the Environment

1. **Place the source files:** Ensure `cetus_regression_test.c` and `helper_tests.h` are in the same directory.

2. **Adjust Configuration:**
   - Open `helper_tests.h`.
   - Modify **`CLANG_PATH`** to point to your `clang` executable.
   - Modify **`CETUS_PATH`** to point to your Cetus executable.

3. **Create input directories:** The test suite expects input C files to be organized within an `input_files` directory, categorized by their `category`.

   Example:
   ```
   .
   ├── cetus_regression_test
   ├── cetus_regression_test.c
   ├── helper_tests.h
   ├── cetus
   └── input_files/
       ├── parallelization/
       │   ├── loop_test_1.c
       │   └── nested_loop.c
       └── basic_syntax/
           └── hello_world.c
   ```

4. **Ground Truth Directory (for comparison mode):** If you plan to use `COMPARE_MODE`, you will need a `ground_truth` directory with corresponding files. This directory will be automatically created when you run in `GENERATE_MODE`.

   Example:
   ```
   .
   └── ground_truth/
       ├── parallelization/
       │   ├── loop_test_1_gt.c
       │   └── nested_loop_gt.c
       └── basic_syntax/
           └── hello_world_gt.c
   ```

---

## Understanding the Code

### Files:

- **`cetus_regression_test.c`**  
  Contains the main logic:
  - `main`: parses arguments, initializes logs, runs tests.
  - `run_test_case`: handles:
    1. Directory creation
    2. Preprocessing via `clang`
    3. Semantic check via `check_syntax.sh`
    4. Running Cetus on `.i` file
    5. Ground truth handling (`generate` or `compare`)
    6. Logging results
  - Helper functions: `get_current_time`, `file_size`, `execute_command`, `log_test_outcome`

- **`helper_tests.h`**  
  Contains:
  - Constants like `CLANG_PATH`, `CETUS_PATH`, `GROUND_TRUTH_SUFFIX`
  - Enum definitions and log file pointers
  - Function prototypes

---

## Compilation

Use this command:

```bash
gcc -o cetus_regression_test cetus_regression_test.c -Wall -Wextra
```

- `-o cetus_regression_test`: Output binary name
- `-Wall -Wextra`: Enable warnings

---

## Running the Tests

```bash
./cetus_regression_test <mode> <category> <input_file_base_name.c> <expected_behavior>
```

---

### Test Modes

- **compare**: Compares Cetus output against ground truth.
- **generate**: Creates ground truth from Cetus output.

---

### Expected Parallelization Behavior

- **parallelize**: Expected to contain OpenMP pragmas.
- **no_parallelize**: Should remain sequential.
- **expect_failure**: Cetus is expected to fail or crash.

---

### Example Usage

Generate ground truth:

```bash
./cetus_regression_test generate parallelization my_loop_test.c parallelize
```

Compare against ground truth:

```bash
./cetus_regression_test compare parallelization my_loop_test.c parallelize
```

Non-parallel test:

```bash
./cetus_regression_test compare no_parallel_tests sequential_code.c no_parallelize
```

Expected failure:

```bash
./cetus_regression_test compare bad_input syntax_error.c expect_failure
```

---

## Output and Logging

The test suite outputs to the console and creates logs in `logs/`.

- `logs/all_tests.log`: Everything
- `logs/passed_tests.log`: Successful tests
- `logs/failed_tests.log`: Tests that failed
- `logs/crashes.log`: Crashes from Clang/Cetus
- `logs/missed_opportunities.log`: Expected parallelism missing
- `logs/incorrect_parallelization.log`: Unexpected or incorrect parallelization

---

## Test Directory Structure

```
.
├── cetus_regression_test           # Executable
├── cetus_regression_test.c         # Main source
├── helper_tests.h                  # Header file
├── cetus                           # Cetus executable
├── check_syntax.sh                 # Semantic check script (stub)
├── input_files/
│   ├── category_A/
│   │   ├── test_case_1.c
│   │   └── test_case_2.c
│   └── category_B/
│       └── another_test.c
├── ground_truth/
│   ├── category_A/
│   │   ├── test_case_1_gt.c
│   │   └── test_case_2_gt.c
│   └── category_B/
│       └── another_test_gt.c
├── logs/
│   ├── all_tests.log
│   ├── passed_tests.log
│   ├── failed_tests.log
│   ├── crashes.log
│   ├── missed_opportunities.log
│   └── incorrect_parallelization.log
├── cetus_intermediate_i_files/
│   ├── category_A/
│   └── category_B/
└── cetus_transformed_output/
    ├── category_A/
    └── category_B/
```
