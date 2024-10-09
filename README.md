# NOTE ON HIPIFYING CUTLASS:


Run on Version: cuda(11.8), LLVM(15.0.0)


1. git clone https://github.com/NVIDIA/cutlass.git
2. cd cutlass

## remove unnecessary folder (i have to remove it bc these error in the hipification. -- if we actually need these folders, we will work through these errors) 
2. rm -rf test
3. rm -rf examples
4. rm -rf tools/profiler 

## run hipify-clang to recursively hipify all files inside this folder. 
## Inplace 
## -I is to include the header files that hipify-clang need to hipify 
## [/scratch/bcjw/hla/cutlass/] is my path to cutlass folder. If you run it, replace by yours 
## -std=c++20: add this flag to make clang work for C++20. 

5. find . -name '*.cu' -exec hipify-clang -inplace {} -I /scratch/bcjw/hla/cutlass/include -I /scratch/bcjw/hla/cutlass/tools/library/include -I /scratch/bcjw/hla/cutlass/tools/library/src  -I /scratch/bcjw/hla/cutlass/include/cutlass -I /scratch/bcjw/hla/cutlass/test/unit/common -I /scratch/bcjw/hla/cutlass/tools/util/include/ -I /scratch/bcjw/hla/cutlass/tools/profiler/include/  -I /scratch/bcjw/hla/googletest/googletest/include  -- -std=c++20 \;

### running this will give 2 warnings, 0 errors: 
[hla@dt-login03 cutlass]$ find . -name '*.cu' -exec hipify-clang -inplace {} -I /scratch/bcjw/hla/cutlass/include -I /scratch/bcjw/hla/cutlass/tools/library/include -I /scratch/bcjw/hla/cutlass/tools/library/src  -I /scratch/bcjw/hla/cutlass/include/cutlass -I /scratch/bcjw/hla/cutlass/test/unit/common -I /scratch/bcjw/hla/cutlass/tools/util/include/ -I /scratch/bcjw/hla/cutlass/tools/profiler/include/  -I /scratch/bcjw/hla/googletest/googletest/include  -- -std=c++20 \;
In file included from /tmp/reduction_device.cu-1677a5.hip:39:
In file included from /scratch/bcjw/hla/cutlass/tools/library/src/reduction/reduction_operation.h:40:
/scratch/bcjw/hla/cutlass/include/cutlass/reduction/thread/reduction_operators.h:175:12: warning: 'integer_subbyte' is deprecated: Implicit conversion is deprecated; please use explicit construction instead [-Wdeprecated-declarations]
  175 |     return uint1b_t(!item);
      |            ^
/scratch/bcjw/hla/cutlass/include/cutlass/integer_subbyte.h:79:3: note: 'integer_subbyte' has been explicitly marked deprecated here
   79 |   integer_subbyte(T value)
      |   ^
In file included from /tmp/reduction_device.cu-1677a5.hip:39:
In file included from /scratch/bcjw/hla/cutlass/tools/library/src/reduction/reduction_operation.h:40:
/scratch/bcjw/hla/cutlass/include/cutlass/reduction/thread/reduction_operators.h:198:12: warning: 'integer_subbyte' is deprecated: Implicit conversion is deprecated; please use explicit construction instead [-Wdeprecated-declarations]
  198 |     return uint1b_t(item);
      |            ^
/scratch/bcjw/hla/cutlass/include/cutlass/integer_subbyte.h:79:3: note: 'integer_subbyte' has been explicitly marked deprecated here
   79 |   integer_subbyte(T value)
      |   ^
2 warnings generated when compiling for host.


### so I think we are close!


### Then convert .cu to .cu.hip
