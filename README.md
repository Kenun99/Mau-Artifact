# Mau-Artifact

Code Base: 10.6084/m9.figshare.24229999

# Build

## Build EmCC from source code

1. install CUDA 11.4 at /usr/local/cuda-11.4/. Check [Cuda Document](https://developer.nvidia.com/cuda-downloads).

   Once installed, check `$nvidia-smi` to see whether the GPU device is activated.

2. install the dependency for LLVM

```bash
cd ./transformer && ./script/install_cmake.sh # install cmake 3.20.0
sudo apt-get update && apt-get install gcc-7 apt-get install g++-7 -y
export CC=/usr/bin/gcc-7
export CXX=/usr/bin/g++-7
sudo ln -s /usr/bin/python2.7 /usr/bin/python # link python if your are at ubuntu 20.04
```

3. build Ethsema

```bash
mkdir build && cd build
cmake -DCMAKE_BUILD_TYPE=RelWithDebInfo -DCUDAPATH=/usr/local/cuda-11.4/ .. # -DCMAKE_BUILD_TYPE=Release for release
cmake --build . --target evmtrans -j8 # use 8 cores here
```

`build/sema/src/standalone-ptxsema` and  `build/rt.o.bc` are generated 

## Build Evaluator from the soruce code

**We are planing to make MAU commercial. Currently, we only release the source code of translator.**

1. You need to install `libssl-dev` (OpenSSL) and `libz3-dev` (refer to [Z3 Installation](#z3-installation) section for instruction) installed.  

2. download the rust. 

   ```bash
   curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
   ```

3. install solc

   `solc` is needed for compiling smart contracts. You can use `solc-select` tool to manage the version of `solc`.

4. Build the cargo project

   You can enable certain debug gates in `Cargo.toml`. For instance, config for the abaltion study.

   ```bash
   # download move dependencies
   git submodule update --recursive --init
   cd cli/
   cargo build --release	
   ```

# Run An Example

Assume we have the smart contract written in Solidity. We can compile it to EVM bytecode first. Note that Mau does not need Solidity source code.

```bash
cd ./tests/multi-contract/
# include the library from ./solidity_utils for example
solc *.sol -o . --bin --abi --overwrite --base-path ../../
```

Convert to PTX code

```bash
build/sema/src/standalone-ptxsema ./tests/complex-condition/main.bin -o ./bytecode.ll --hex --dump && llvm-link build/rt.o.bc ./bytecode.ll -o ./kernel.bc && llvm-dis kernel.bc -o kernel.ll && llc-16 -mcpu=sm_86 kernel.bc -o kernel.ptx 
```

Run Fuzzer:

```bash
cd ./cli/
LD_LIBRARY_PATH=build/runner/ ./cli/target/release/cli -t './tests/complex-condition/*' --ptx-path kernel.ptx --gpu-dev 0
```

In case you want to disable GPU.

Run `./cli -t '../tests/multi-contract/*'`
