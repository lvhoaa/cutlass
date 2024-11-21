# NOTE ON HIPIFYING CUTLASS:


Run on Version: cuda(11.8), LLVM(15.0.0)


1. git clone https://github.com/NVIDIA/cutlass.git
2. cd cutlass

### remove unnecessary folder (i have to remove it bc these error in the hipification. -- if we actually need these folders, we will work through these errors) 
2. rm -rf test
3. rm -rf examples
4. rm -rf tools/profiler 

### run hipify-clang to recursively hipify all files inside this folder. 
Note: 
-I is to include the header files that hipify-clang need to hipify 
[/scratch/bcjw/hla/cutlass/] is my path to cutlass folder. If you run it, replace by yours 
-std=c++20: add this flag to make clang work for C++20. 

### hipify-clang on CUDA files (.cu)

5. find . -name '*.cu' -exec hipify-clang -inplace {} -I /scratch/bcjw/hla/cutlass/include -I /scratch/bcjw/hla/cutlass/tools/library/include -I /scratch/bcjw/hla/cutlass/tools/library/src  -I /scratch/bcjw/hla/cutlass/include/cutlass -I /scratch/bcjw/hla/cutlass/test/unit/common -I /scratch/bcjw/hla/cutlass/tools/util/include/ -I /scratch/bcjw/hla/cutlass/tools/profiler/include/  -I /scratch/bcjw/hla/googletest/googletest/include  -- -std=c++20 \;


This effort seems good because it only returns 2 warnings and 0 errors. 

```
In file included from /tmp/reduction_device.cu-1677a5.hip:39: In file included from /scratch/bcjw/hla/cutlass/tools/library/src/reduction/reduction_operation.h:40: /scratch/bcjw/hla/cutlass/include/cutlass/reduction/thread/reduction_operators.h:175:12: warning: 'integer_subbyte' is deprecated: Implicit conversion is deprecated; please use explicit construction instead [-Wdeprecated-declarations] 175 | return uint1b_t(!item); | ^ /scratch/bcjw/hla/cutlass/include/cutlass/integer_subbyte.h:79:3: note: 'integer_subbyte' has been explicitly marked deprecated here 79 | integer_subbyte(T value) | ^ In file included from /tmp/reduction_device.cu-1677a5.hip:39: In file included from /scratch/bcjw/hla/cutlass/tools/library/src/reduction/reduction_operation.h:40: /scratch/bcjw/hla/cutlass/include/cutlass/reduction/thread/reduction_operators.h:198:12: warning: 'integer_subbyte' is deprecated: Implicit conversion is deprecated; please use explicit construction instead [-Wdeprecated-declarations] 198 | return uint1b_t(item); | ^ /scratch/bcjw/hla/cutlass/include/cutlass/integer_subbyte.h:79:3: note: 'integer_subbyte' has been explicitly marked deprecated here 79 | integer_subbyte(T value) | ^ 2 warnings generated when compiling for host.
```


### hipify-clang on header files (.h & .hpp)

6. find . -name '*.h' -o -name '*.hpp' -exec hipify-clang -inplace {} -I /scratch/bcjw/hla/cutlass/include -I /scratch/bcjw/hla/cutlass/include/cute -I /scratch/bcjw/hla/cutlass/include/cutlass  -I /scratch/bcjw/hla/cutlass/include/cute/numeric -I /scratch/bcjw/hla/cutlass/tools/library/include -I /scratch/bcjw/hla/cutlass/tools/library/src -I /scratch/bcjw/hla/cutlass/tools/util/include/ -I /opt/rocm/include/  -- -std=c++20 \;


Now, this is the main problem when it produces so many errors. Common errors are unknown type name, redefinition, use of undeclared identifier

Problems like redefinition are probably due to includes, so I have ensured that header guards/ include guards are added. But they still do not work. 


```
/tmp/cuda_host_adapter.hpp-a4dcb6.hip:146:1: error: unknown type name 'cudaDriverEntryPointQueryResult'
  146 | CUTLASS_CUDA_DRIVER_WRAPPER_DECL(cuTensorMapEncodeTiled, 12000);
      | ^
/tmp/cuda_host_adapter.hpp-a4dcb6.hip:127:5: note: expanded from macro 'CUTLASS_CUDA_DRIVER_WRAPPER_DECL'
  127 |     cudaDriverEntryPointQueryResult cuda_status;                \
      |     ^
/tmp/cuda_host_adapter.hpp-a4dcb6.hip:146:1: error: use of undeclared identifier 'cudaDriverEntryPointSuccess'
/tmp/cuda_host_adapter.hpp-a4dcb6.hip:134:24: note: expanded from macro 'CUTLASS_CUDA_DRIVER_WRAPPER_DECL'
  134 |     if (cuda_status != cudaDriverEntryPointSuccess ||           \
      |                        ^
/tmp/cuda_host_adapter.hpp-a4dcb6.hip:146:1: error: unknown type name 'PFN_cuTensorMapEncodeTiled'
/tmp/cuda_host_adapter.hpp-a4dcb6.hip:138:29: note: expanded from macro 'CUTLASS_CUDA_DRIVER_WRAPPER_DECL'
  138 |     return reinterpret_cast<PFN_##func>(pfn)(args...);          \
      |                             ^
<scratch space>:150:1: note: expanded from here
  150 | PFN_cuTensorMapEncodeTiled
      | ^
/tmp/cuda_host_adapter.hpp-a4dcb6.hip:147:1: error: unknown type name 'cudaDriverEntryPointQueryResult'
  147 | CUTLASS_CUDA_DRIVER_WRAPPER_DECL(cuTensorMapEncodeIm2col, 12000);
      | ^
/tmp/cuda_host_adapter.hpp-a4dcb6.hip:127:5: note: expanded from macro 'CUTLASS_CUDA_DRIVER_WRAPPER_DECL'
  127 |     cudaDriverEntryPointQueryResult cuda_status;                \
      |     ^
/tmp/cuda_host_adapter.hpp-a4dcb6.hip:147:1: error: use of undeclared identifier 'cudaDriverEntryPointSuccess'
/tmp/cuda_host_adapter.hpp-a4dcb6.hip:134:24: note: expanded from macro 'CUTLASS_CUDA_DRIVER_WRAPPER_DECL'
  134 |     if (cuda_status != cudaDriverEntryPointSuccess ||           \
      |                        ^
/tmp/cuda_host_adapter.hpp-a4dcb6.hip:147:1: error: unknown type name 'PFN_cuTensorMapEncodeIm2col'
/tmp/cuda_host_adapter.hpp-a4dcb6.hip:138:29: note: expanded from macro 'CUTLASS_CUDA_DRIVER_WRAPPER_DECL'
  138 |     return reinterpret_cast<PFN_##func>(pfn)(args...);          \
      |                             ^
<scratch space>:153:1: note: expanded from here
  153 | PFN_cuTensorMapEncodeIm2col
      | ^
/tmp/cuda_host_adapter.hpp-a4dcb6.hip:332:5: error: unknown type name 'CUtensorMap'
  332 |     CUtensorMap* tensorMap,
      |     ^
/tmp/cuda_host_adapter.hpp-a4dcb6.hip:333:5: error: unknown type name 'CUtensorMapDataType'
  333 |     CUtensorMapDataType tensorDataType,
      |     ^
/tmp/cuda_host_adapter.hpp-a4dcb6.hip:343:5: error: unknown type name 'CUtensorMapInterleave'
  343 |     CUtensorMapInterleave interleave,
      |     ^
/tmp/cuda_host_adapter.hpp-a4dcb6.hip:344:5: error: unknown type name 'CUtensorMapSwizzle'
  344 |     CUtensorMapSwizzle swizzle,
      |     ^
/tmp/cuda_host_adapter.hpp-a4dcb6.hip:345:5: error: unknown type name 'CUtensorMapL2promotion'
  345 |     CUtensorMapL2promotion l2Promotion,
      |     ^
/tmp/cuda_host_adapter.hpp-a4dcb6.hip:346:5: error: unknown type name 'CUtensorMapFloatOOBfill'
  346 |     CUtensorMapFloatOOBfill oobFill) const = 0;
      |     ^
/tmp/cuda_host_adapter.hpp-a4dcb6.hip:350:5: error: unknown type name 'CUtensorMap'
  350 |     CUtensorMap* tensorMap,
      |     ^
/tmp/cuda_host_adapter.hpp-a4dcb6.hip:351:5: error: unknown type name 'CUtensorMapDataType'
  351 |     CUtensorMapDataType tensorDataType,
      |     ^
/tmp/cuda_host_adapter.hpp-a4dcb6.hip:358:5: error: unknown type name 'CUtensorMapInterleave'
  358 |     CUtensorMapInterleave interleave,
      |     ^
/tmp/cuda_host_adapter.hpp-a4dcb6.hip:359:5: error: unknown type name 'CUtensorMapSwizzle'
  359 |     CUtensorMapSwizzle swizzle,
      |     ^
/tmp/cuda_host_adapter.hpp-a4dcb6.hip:360:5: error: unknown type name 'CUtensorMapL2promotion'
  360 |     CUtensorMapL2promotion l2Promotion,
      |     ^
/tmp/cuda_host_adapter.hpp-a4dcb6.hip:361:5: error: unknown type name 'CUtensorMapFloatOOBfill'
  361 |     CUtensorMapFloatOOBfill oobFill) const = 0;
      |     ^
/tmp/cuda_host_adapter.hpp-a4dcb6.hip:365:5: error: unknown type name 'CUtensorMap'
  365 |     CUtensorMap* tensorMap,
      |     ^
/tmp/cuda_host_adapter.hpp-a4dcb6.hip:108:5: warning: 'cudaDriverEntryPointQueryResult' is unsupported in 'HIP'.
  108 |     cudaDriverEntryPointQueryResult cuda_status;                \
      |     ^
/tmp/cuda_host_adapter.hpp-a4dcb6.hip:113:9: warning: 'cudaEnableDefault' is unsupported in 'HIP'.
  113 |         cudaEnableDefault,                                      \
      |         ^
/tmp/cuda_host_adapter.hpp-a4dcb6.hip:115:24: warning: 'cudaDriverEntryPointSuccess' is unsupported in 'HIP'.
  115 |     if (cuda_status != cudaDriverEntryPointSuccess ||           \
      |                        ^
/tmp/cuda_host_adapter.hpp-a4dcb6.hip:127:5: warning: 'cudaDriverEntryPointQueryResult' is unsupported in 'HIP'.
  127 |     cudaDriverEntryPointQueryResult cuda_status;                \
      |     ^
/tmp/cuda_host_adapter.hpp-a4dcb6.hip:129:28: warning: 'cudaGetDriverEntryPoint' is experimental in 'HIP'; to hipify it, use the '--experimental' option.
  129 |     cudaError_t cuda_err = cudaGetDriverEntryPoint(             \
      |                            ^
/tmp/cuda_host_adapter.hpp-a4dcb6.hip:132:9: warning: 'cudaEnableDefault' is unsupported in 'HIP'.
  132 |         cudaEnableDefault,                                      \
      |         ^
/tmp/cuda_host_adapter.hpp-a4dcb6.hip:134:24: warning: 'cudaDriverEntryPointSuccess' is unsupported in 'HIP'.
  134 |     if (cuda_status != cudaDriverEntryPointSuccess ||           \
      |                        ^
/tmp/cuda_host_adapter.hpp-a4dcb6.hip:146:34: warning: 'cuTensorMapEncodeTiled' is unsupported in 'HIP'.
  146 | CUTLASS_CUDA_DRIVER_WRAPPER_DECL(cuTensorMapEncodeTiled, 12000);
      |                                  ^
/tmp/cuda_host_adapter.hpp-a4dcb6.hip:147:34: warning: 'cuTensorMapEncodeIm2col' is unsupported in 'HIP'.
  147 | CUTLASS_CUDA_DRIVER_WRAPPER_DECL(cuTensorMapEncodeIm2col, 12000);
      |                                  ^
/tmp/cuda_host_adapter.hpp-a4dcb6.hip:170:3: warning: 'CUlaunchAttribute' is unsupported in 'HIP'.
  170 |   CUlaunchAttribute launch_attributes[kMaximumAttributeCount];
      |   ^
/tmp/cuda_host_adapter.hpp-a4dcb6.hip:174:28: warning: 'CUlaunchAttribute' is unsupported in 'HIP'.
  174 |   CudaHostLaunchAttributes(CUlaunchAttribute *launch_attributes_ = nullptr,
      |                            ^
/tmp/cuda_host_adapter.hpp-a4dcb6.hip:184:3: warning: 'CUlaunchAttribute' is unsupported in 'HIP'.
  184 |   CUlaunchAttribute const* data() const {
      |   ^
/tmp/cuda_host_adapter.hpp-a4dcb6.hip:332:5: warning: 'CUtensorMap' is unsupported in 'HIP'.
  332 |     CUtensorMap* tensorMap,
      |     ^
/tmp/cuda_host_adapter.hpp-a4dcb6.hip:333:5: warning: 'CUtensorMapDataType' is unsupported in 'HIP'.
  333 |     CUtensorMapDataType tensorDataType,
      |     ^
/tmp/cuda_host_adapter.hpp-a4dcb6.hip:343:5: warning: 'CUtensorMapInterleave' is unsupported in 'HIP'.
  343 |     CUtensorMapInterleave interleave,
      |     ^
/tmp/cuda_host_adapter.hpp-a4dcb6.hip:344:5: warning: 'CUtensorMapSwizzle' is unsupported in 'HIP'.
  344 |     CUtensorMapSwizzle swizzle,
      |     ^
/tmp/cuda_host_adapter.hpp-a4dcb6.hip:345:5: warning: 'CUtensorMapL2promotion' is unsupported in 'HIP'.
  345 |     CUtensorMapL2promotion l2Promotion,
      |     ^
/tmp/cuda_host_adapter.hpp-a4dcb6.hip:346:5: warning: 'CUtensorMapFloatOOBfill' is unsupported in 'HIP'.
  346 |     CUtensorMapFloatOOBfill oobFill) const = 0;
      |     ^
/tmp/cuda_host_adapter.hpp-a4dcb6.hip:350:5: warning: 'CUtensorMap' is unsupported in 'HIP'.
  350 |     CUtensorMap* tensorMap,
      |     ^
/tmp/cuda_host_adapter.hpp-a4dcb6.hip:351:5: warning: 'CUtensorMapDataType' is unsupported in 'HIP'.
  351 |     CUtensorMapDataType tensorDataType,
      |     ^
/tmp/cuda_host_adapter.hpp-a4dcb6.hip:358:5: warning: 'CUtensorMapInterleave' is unsupported in 'HIP'.
  358 |     CUtensorMapInterleave interleave,
      |     ^
/tmp/cuda_host_adapter.hpp-a4dcb6.hip:359:5: warning: 'CUtensorMapSwizzle' is unsupported in 'HIP'.
  359 |     CUtensorMapSwizzle swizzle,
      |     ^
/tmp/cuda_host_adapter.hpp-a4dcb6.hip:360:5: warning: 'CUtensorMapL2promotion' is unsupported in 'HIP'.
  360 |     CUtensorMapL2promotion l2Promotion,
      |     ^
/tmp/cuda_host_adapter.hpp-a4dcb6.hip:361:5: warning: 'CUtensorMapFloatOOBfill' is unsupported in 'HIP'.
  361 |     CUtensorMapFloatOOBfill oobFill) const = 0;
      |     ^
/tmp/cuda_host_adapter.hpp-a4dcb6.hip:365:5: warning: 'CUtensorMap' is unsupported in 'HIP'.
  365 |     CUtensorMap* tensorMap,
      |     ^
27 warnings and 19 errors generated when compiling for host.
Error while processing /tmp/cuda_host_adapter.hpp-a4dcb6.hip.
/tmp/synclog.hpp-c77c3f.hip:297:13: error: redefinition of 'synclog_setup'
  297 | inline void synclog_setup() {
      |             ^
/scratch/bcjw/hla/cutlass/include/cutlass/arch/synclog.hpp:297:13: note: previous definition is here
  297 | inline void synclog_setup() {
      |             ^
/tmp/synclog.hpp-c77c3f.hip:339:6: error: redefinition of 'synclog_emit_syncthreads'
  339 | void synclog_emit_syncthreads(uint32_t line) {
      |      ^
/scratch/bcjw/hla/cutlass/include/cutlass/arch/synclog.hpp:339:6: note: previous definition is here
  339 | void synclog_emit_syncthreads(uint32_t line) {
      |      ^
/tmp/synclog.hpp-c77c3f.hip:352:6: error: redefinition of 'synclog_emit_syncwarp'
  352 | void synclog_emit_syncwarp(uint32_t line) {
      |      ^
/scratch/bcjw/hla/cutlass/include/cutlass/arch/synclog.hpp:352:6: note: previous definition is here
  352 | void synclog_emit_syncwarp(uint32_t line) {
      |      ^
/tmp/synclog.hpp-c77c3f.hip:365:6: error: redefinition of 'synclog_emit_named_barrier_arrive_and_wait'
  365 | void synclog_emit_named_barrier_arrive_and_wait(
      |      ^
/scratch/bcjw/hla/cutlass/include/cutlass/arch/synclog.hpp:365:6: note: previous definition is here
  365 | void synclog_emit_named_barrier_arrive_and_wait(
      |      ^
/tmp/synclog.hpp-c77c3f.hip:385:6: error: redefinition of 'synclog_emit_named_barrier_arrive'
  385 | void synclog_emit_named_barrier_arrive(
      |      ^
/scratch/bcjw/hla/cutlass/include/cutlass/arch/synclog.hpp:385:6: note: previous definition is here
  385 | void synclog_emit_named_barrier_arrive(
      |      ^
/tmp/synclog.hpp-c77c3f.hip:405:6: error: redefinition of 'synclog_emit_cluster_barrier_init'
  405 | void synclog_emit_cluster_barrier_init(
      |      ^
/scratch/bcjw/hla/cutlass/include/cutlass/arch/synclog.hpp:405:6: note: previous definition is here
  405 | void synclog_emit_cluster_barrier_init(
      |      ^
/tmp/synclog.hpp-c77c3f.hip:425:6: error: redefinition of 'synclog_emit_cluster_barrier_wait'
  425 | void synclog_emit_cluster_barrier_wait(
      |      ^
/scratch/bcjw/hla/cutlass/include/cutlass/arch/synclog.hpp:425:6: note: previous definition is here
  425 | void synclog_emit_cluster_barrier_wait(
      |      ^
/tmp/synclog.hpp-c77c3f.hip:448:6: error: redefinition of 'synclog_emit_cluster_barrier_test_wait'
  448 | void synclog_emit_cluster_barrier_test_wait(
      |      ^
/scratch/bcjw/hla/cutlass/include/cutlass/arch/synclog.hpp:448:6: note: previous definition is here
  448 | void synclog_emit_cluster_barrier_test_wait(
      |      ^
/tmp/synclog.hpp-c77c3f.hip:474:6: error: redefinition of 'synclog_emit_cluster_barrier_try_wait'
  474 | void synclog_emit_cluster_barrier_try_wait(
      |      ^
/scratch/bcjw/hla/cutlass/include/cutlass/arch/synclog.hpp:474:6: note: previous definition is here
  474 | void synclog_emit_cluster_barrier_try_wait(
      |      ^
/tmp/synclog.hpp-c77c3f.hip:497:6: error: redefinition of 'synclog_emit_cluster_barrier_arrive_cluster'
  497 | void synclog_emit_cluster_barrier_arrive_cluster(
      |      ^
/scratch/bcjw/hla/cutlass/include/cutlass/arch/synclog.hpp:497:6: note: previous definition is here
  497 | void synclog_emit_cluster_barrier_arrive_cluster(
      |      ^
/tmp/synclog.hpp-c77c3f.hip:523:6: error: redefinition of 'synclog_emit_cluster_barrier_arrive'
  523 | void synclog_emit_cluster_barrier_arrive(
      |      ^
/scratch/bcjw/hla/cutlass/include/cutlass/arch/synclog.hpp:523:6: note: previous definition is here
  523 | void synclog_emit_cluster_barrier_arrive(
      |      ^
/tmp/synclog.hpp-c77c3f.hip:543:6: error: redefinition of 'synclog_emit_cluster_barrier_invalidate'
  543 | void synclog_emit_cluster_barrier_invalidate(
      |      ^
/scratch/bcjw/hla/cutlass/include/cutlass/arch/synclog.hpp:543:6: note: previous definition is here
  543 | void synclog_emit_cluster_barrier_invalidate(
      |      ^
/tmp/synclog.hpp-c77c3f.hip:563:6: error: redefinition of 'synclog_emit_cluster_transaction_barrier_arrive_and_expect_tx'
  563 | void synclog_emit_cluster_transaction_barrier_arrive_and_expect_tx(
      |      ^
/scratch/bcjw/hla/cutlass/include/cutlass/arch/synclog.hpp:563:6: note: previous definition is here
  563 | void synclog_emit_cluster_transaction_barrier_arrive_and_expect_tx(
      |      ^
/tmp/synclog.hpp-c77c3f.hip:586:6: error: redefinition of 'synclog_emit_cluster_transaction_barrier_arrive_and_expect_tx_cluster'
  586 | void synclog_emit_cluster_transaction_barrier_arrive_and_expect_tx_cluster(
      |      ^
/scratch/bcjw/hla/cutlass/include/cutlass/arch/synclog.hpp:586:6: note: previous definition is here
  586 | void synclog_emit_cluster_transaction_barrier_arrive_and_expect_tx_cluster(
      |      ^
/tmp/synclog.hpp-c77c3f.hip:615:6: error: redefinition of 'synclog_emit_cluster_transaction_barrier_expect_transaction'
  615 | void synclog_emit_cluster_transaction_barrier_expect_transaction(
      |      ^
/scratch/bcjw/hla/cutlass/include/cutlass/arch/synclog.hpp:615:6: note: previous definition is here
  615 | void synclog_emit_cluster_transaction_barrier_expect_transaction(
      |      ^
/tmp/synclog.hpp-c77c3f.hip:638:6: error: redefinition of 'synclog_emit_cluster_transaction_barrier_complete_transaction'
  638 | void synclog_emit_cluster_transaction_barrier_complete_transaction(
      |      ^
/scratch/bcjw/hla/cutlass/include/cutlass/arch/synclog.hpp:638:6: note: previous definition is here
  638 | void synclog_emit_cluster_transaction_barrier_complete_transaction(
      |      ^
/tmp/synclog.hpp-c77c3f.hip:667:6: error: redefinition of 'synclog_emit_fence_barrier_init'
  667 | void synclog_emit_fence_barrier_init(uint32_t line) {
      |      ^
/scratch/bcjw/hla/cutlass/include/cutlass/arch/synclog.hpp:667:6: note: previous definition is here
  667 | void synclog_emit_fence_barrier_init(uint32_t line) {
      |      ^
/tmp/synclog.hpp-c77c3f.hip:680:6: error: redefinition of 'synclog_emit_fence_view_async_shared'
  680 | void synclog_emit_fence_view_async_shared(uint32_t line) {
      |      ^
/scratch/bcjw/hla/cutlass/include/cutlass/arch/synclog.hpp:680:6: note: previous definition is here
  680 | void synclog_emit_fence_view_async_shared(uint32_t line) {
      |      ^
/tmp/synclog.hpp-c77c3f.hip:693:6: error: redefinition of 'synclog_emit_cp_async_wait'
  693 | void synclog_emit_cp_async_wait(
      |      ^
/scratch/bcjw/hla/cutlass/include/cutlass/arch/synclog.hpp:693:6: note: previous definition is here
  693 | void synclog_emit_cp_async_wait(
      |      ^
fatal error: too many errors emitted, stopping now [-ferror-limit=]
20 errors generated when compiling for host.
```