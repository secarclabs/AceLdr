# AceLdr Build Requirements

## Build Failure Analysis

The `make` command fails due to **missing build dependencies**.

## Required Dependencies

### 1. NASM (Netwide Assembler)
- **Status**: ❌ NOT INSTALLED
- **Purpose**: Assembles x64 assembly files (.asm)
- **Used for**: 
  - `src/asm/start.asm` - Entry point and stack alignment
  - `src/asm/misc.asm` - GetIp() and stub data structure
  - `src/asm/spoof.asm` - Return address spoofing gadget
- **Error**: `nasm: No such file or directory`

### 2. pefile (Python Module)
- **Status**: ❌ NOT INSTALLED
- **Purpose**: Parse and manipulate PE (Portable Executable) files
- **Used for**: `scripts/extract.py` extracts shellcode from compiled PE
- **Error**: `ModuleNotFoundError: No module named 'pefile'`

### 3. MinGW-w64 GCC (x86_64)
- **Status**: ✅ INSTALLED
- **Location**: `/usr/bin/x86_64-w64-mingw32-gcc`
- **Purpose**: Cross-compile C code for Windows x64

### 4. Python 3
- **Status**: ✅ INSTALLED
- **Location**: `/usr/bin/python3`
- **Purpose**: Run extraction script

## Installation Instructions

### Quick Install (Recommended)

```bash
sudo apt update && sudo apt install -y nasm python3-pefile
```

### Individual Installation

#### Install NASM
```bash
sudo apt update
sudo apt install -y nasm
```

#### Install pefile (Option 1: System Package)
```bash
sudo apt install -y python3-pefile
```

#### Install pefile (Option 2: pip)
```bash
pip3 install pefile
```

## Verification

After installing dependencies, verify:

```bash
# Check NASM
nasm -v
# Expected: NASM version 2.xx.xx compiled on ...

# Check pefile
python3 -c "import pefile; print('pefile OK')"
# Expected: pefile OK

# Check MinGW GCC
x86_64-w64-mingw32-gcc --version
# Expected: x86_64-w64-mingw32-gcc (GCC) ...
```

## Build Process

Once dependencies are installed:

```bash
# Clean previous build artifacts
make clean

# Build the loader
make

# Expected output file
ls -lh bin/AceLdr.x64.bin
```

## Build Steps Explained

1. **Assemble ASM files** → Generate .tmp.o object files
   ```
   nasm -Werror=all -f win64 src/asm/start.asm -o bin/start.tmp.o
   nasm -Werror=all -f win64 src/asm/misc.asm -o bin/misc.tmp.o
   nasm -Werror=all -f win64 src/asm/spoof.asm -o bin/spoof.tmp.o
   ```

2. **Compile and Link** → Generate PE executable
   ```
   x86_64-w64-mingw32-gcc src/*.c bin/*.tmp.o src/hooks/*.c \
       -o bin/AceLdr.x64.exe [CFLAGS] [LFLAGS]
   ```

3. **Extract Shellcode** → Generate raw binary
   ```
   python3 scripts/extract.py -f bin/AceLdr.x64.exe -o bin/AceLdr.x64.bin
   ```

4. **Cleanup** → Remove temporary files
   ```
   rm bin/*.tmp.o bin/AceLdr.x64.exe
   ```

## Expected Artifacts

After successful build:
- `bin/AceLdr.x64.bin` - Position-independent shellcode (~8-12 KB)
- Ready to use with Cobalt Strike via `bin/AceLdr.cna`

## Troubleshooting

### Build still fails after installing dependencies

1. **Verify installation**:
   ```bash
   which nasm x86_64-w64-mingw32-gcc python3
   python3 -c "import pefile"
   ```

2. **Check for syntax errors**:
   ```bash
   make --dry-run
   ```

3. **Clean and rebuild**:
   ```bash
   make clean
   make
   ```

### Permission errors

Ensure you have write permissions to the `bin/` directory:
```bash
ls -ld bin/
chmod 755 bin/
```

## System Information

- **OS**: Linux Mint 22.2 (Zara)
- **Package Manager**: apt
- **Architecture**: x86_64

## Additional Notes

- Build requires Windows cross-compilation toolchain (MinGW-w64)
- Output is Windows x64 shellcode (not Linux native)
- Shellcode is position-independent and can load from any address
- NASM version 2.13+ recommended
- pefile version 2019.4.18+ recommended

