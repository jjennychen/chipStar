# chipStar Device Library

This directory contains the source code for the chipStar Device Library. The header files declare math functions and intrinsics that can be used inside kernel code with C++ linkage and call an underlaying device-side function. The device-side function implementations are provided either by OpenCL, OCML, or our own custom implementation. 

## Device-side Function Headers
The headers for device-side functions can be found in `chipStar/include/hip/devicelib`. The header files are not provided by HIP-COMMON so instead these headers file were assmebled by hand from the official CUDA and HIP documentation:
[CUDA Math API](https://docs.nvidia.com/cuda/cuda-math-api/index.html)
[HIP Programming Guide](https://docs.amd.com/bundle/HIP-Programming-Guide-v5.0/page/Programming_with_HIP.html)

## Deviside-side Function Implementations
Most of the function implementations come from OpenCL. For other cases, we use the OCML library. The OCML library is provided by AMD and can be found in the [ROCm-Device-Libs](https://github.com/RadeonOpenCompute/ROCm-Device-Libs) repo. 
If a function is not provided by OpenCL or OCML, we provide our own implementation which lives in `chipStar/bitcode/devicelib.cl`.

# ROCm-Device-Library
This library provides function implmementations that can be compiled into LLVM IR bitcode. Unfortunately, it was implemented to for AMD architectures which results in the use of admgcn intrinsics which won't compile to SPIR-V. For this reason, we had to modify some of the implementations and correctness of these implementations is poorly tested. 

In order to link a functioning bitcode library, we must link in some control libraries which modify the behavior of the device-side functions. These libraries are OCLC and OCKL
`chipStar/bitcode/CMakeLists.txt`:
```
# ROCm-Device-Libs provides OCML and its dependencies (OCLC, OCKL, etc.)
# Since these targets don't seem to get exported as normal targets, we have to link this way.
set(OCML_LIBS
  ocml
  oclc_finite_only_off
  oclc_unsafe_math_off
  oclc_correctly_rounded_sqrt_off
  oclc_daz_opt_off
  oclc_isa_version_803
)
```
Note: Some amdgcn intrinsics still don't have generic equivalents so for example `oclc_daz_opt_off` is mandatory.

# Develper Notes
* Don't change the order of the headers.
* If there are missing headers, add them to the end of the list with a comment that they're undocumented. The missing documentationt should be reported to AMD or NVIDIA. 
* Avoid using macros, escpecially in the headers.
* `devicelib.cl` does use some macros mainly to avoid code duplication when declaring function overloads. If you do use macros, make sure to leave a comment with the full function name that the macro expands to.
* All device-side functions should be declared with `extern "C"` and follow the following naming convention: `__chip_<function_name_snake_case>_<type>` where type is one of `f32`, `f64`, `i32`, `i64`, `u32`, `u64`. For example, `__chip_sin_f32` is the device-side function that implements the `sin` function for `float` arguments. Note: this convention is not yet fully implemented.

## Obfuscated Pointer Parameters

A selection of devicelib definitions use `__chip_obfuscated_ptr_t` as
parameter type insted of regular pointer type. This is for working
around LLVM-SPIR-VTranslation feature that attempts to recover
original pointee types in LLVM bitcode input which uses opaque
pointers (only option with the latest LLVM).

An issue with the feature is that the type inference is influenced by
the surrounding code and this causes situations where same LLVM
function declarations and definitions may end up to be have parameters
with different pointee types across SPIR-V modules. Linking such
modules together will probably trigger undefined behavior. Al least on
Mesa's rusticl frontend the linking is known to fail.

The shortcoming of the type inference is worked around by passing the
pointer arguments as some non-pointer type (`__chip_obfuscated_ptr_t`)
and it's used for devicelib definitions which are linked at runtime.
