# PIPELINED RISC-V PROCESSOR

## Overview

This project implements a high-performance pipelined RISC-V processor that extends the single-cycle design with a 3-stage pipeline architecture. The processor includes advanced features such as hazard detection, forwarding, and branch resolution to maximize performance while maintaining instruction compatibility.

## Features

- **3-stage pipeline architecture** for improved performance
- **Hazard detection and forwarding** to minimize stalls
- **Branch resolution in first stage** to avoid pipeline bubbles
- **Maximum clock frequency**: 140 MHz (significant improvement from 70 MHz single-cycle)
- **Runtime optimization**: 507 ns for factorial 6 (vs 1014 ns single-cycle)
- **Comprehensive instruction support** including all RISC-V instruction types

## Pipeline Architecture

### Stage 1: IF/ID (Instruction Fetch/Decode)
- **Program Counter** - Instruction sequencing and branch resolution
- **Registers** - Register file access and data forwarding
- **Hazard Detection** - RAW hazard identification
- **Control Unit** - Instruction decoding and control signal generation
- **Immediate Generator** - Immediate value extraction and sign extension

### Stage 2: EX (Execute)
- **ALU** - Arithmetic and logical operations
- **Multiplier** - Dedicated multiplication unit
- **Branch Comparison** - Early branch resolution

### Stage 3: MEM/WB (Memory/Write Back)
- **Memory Interface** - Load/store operations
- **Write Back Logic** - Register file updates

## Performance Improvements

### Single-Cycle vs Pipelined Comparison
| Metric | Single-Cycle | Pipelined | Improvement |
|--------|---------------|-----------|-------------|
| **Clock Frequency** | 70 MHz | 140 MHz | 2x |
| **Runtime (factorial 6)** | 1014 ns | 507 ns | 50% reduction |
| **Cycles (factorial 6)** | 71 cycles | 71 cycles | Same |
| **CPI** | 1.0 | 1.0 (ideal) | Maintained |

### Key Optimizations
- **Early branch resolution** - Branches resolved in first stage
- **Forwarding paths** - Eliminates most RAW hazards
- **Optimized critical path** - Focused on PC increment and register comparison
- **Memory timing** - Dual-edge clocking for memory operations

## Hazard Handling

### RAW (Read After Write) Hazards
- **Detection**: Hardware hazard detector monitors register dependencies
- **Forwarding**: Direct data paths from EX and MEM stages to resolve hazards
- **Stalling**: Processor stalls when forwarding cannot resolve hazards

### Types of Hazards Detected
1. **Load-Use Hazards**: When instruction needs data from previous load
2. **Compute-Use Hazards**: When instruction needs ALU result from previous instruction
3. **Branch Hazards**: Resolved through early branch resolution

### Example Hazard Resolution
```assembly
lw x7, 0(x3)     # Load instruction
add x8, x7, x4   # Dependent instruction (load-use hazard)
```
- **Without forwarding**: 2-cycle stall required
- **With forwarding**: Data forwarded from MEM stage, no stall

## Supported Instructions

### Arithmetic Operations
- `ADD`, `ADDI` - Addition operations
- `SUB` - Subtraction
- `MUL` - Multiplication (pipelined)
- `SLLI` - Shift left logical immediate

### Logical Operations
- `AND`, `OR` - Bitwise operations

### Memory Operations
- `LW` - Load word from memory
- `SW` - Store word to memory

### Control Flow
- `BEQ` - Branch if equal (resolved in stage 1)
- `BNE` - Branch if not equal (resolved in stage 1)
- `JAL` - Jump and link (resolved in stage 1)
- `JALR` - Jump and link register (resolved in stage 1)

### System
- `HALT` - Stop execution

## Branch Resolution Strategy

### Early Branch Resolution
- **Location**: First pipeline stage (IF/ID)
- **Comparison**: Register data compared immediately after read
- **Jump Handling**: JAL and JALR resolved with immediate PC update
- **Benefit**: Eliminates branch delay penalties

### Implementation Details
```verilog
// Branch comparison in stage 1
zero = (reg_data1 == reg_data2);
branch_taken = (beq & zero) | (bne & ~zero);
pc_next = branch_taken ? (pc + immediate) : (pc + 4);
```

## Test Programs

### Factorial Program
Calculates factorial of 6 using recursive function calls with proper stack management.

**Performance Results:**
- **Cycles**: 71 (consistent with single-cycle)
- **Runtime**: 507 ns at 140 MHz
- **Memory verification**: Matches expected factorial results

### Verification Program
Custom test program exercising hazard detection:
- **Compute-use hazards**: BEQ instructions with dependent registers
- **Load-use hazards**: Instructions using data from previous load
- **Performance**: 14 cycles with hazard detection, 18 without

## Resource Utilization

### FPGA Resources
- **ALMs**: 2,012 (Adaptive Logic Modules)
- **Dedicated Logic Registers**: 1,580
- **DSPs**: 2 (Digital Signal Processors)
- **PLLs**: 1 (Phase-Locked Loop)

### Area vs Performance Trade-offs
- **Optimization focus**: Speed over area
- **Synthesis settings**: Optimized for maximum frequency
- **Pipeline registers**: Additional storage for inter-stage data

## Getting Started

### Prerequisites
- Intel Quartus Prime (for FPGA synthesis)
- ModelSim (for simulation)
- TimingQuest Analyzer (for timing analysis)
- Compatible FPGA board

### Setup Instructions
1. Extract the `.qar` file in Quartus Prime
2. Configure PLL for 140 MHz operation (25% duty cycle)
3. Compile the project with speed optimization settings
4. Program the FPGA board
5. Load test programs and verify execution

### Running Test Programs
1. Load assembly programs into instruction memory
2. Initialize cycle counter (register x30)
3. Execute program and monitor completion
4. Read memory contents using In-System Memory Content Editor
5. Verify cycle count and results

## Design Methodology

### Development Phases
1. **Phase A**: Pipeline without hazard handling (manual NOP insertion)
2. **Phase B**: Hardware hazard detection with stalling
3. **Phase C**: Forwarding implementation to minimize stalls

### Verification Strategy
- **Functional verification**: Same programs as single-cycle processor
- **Performance verification**: Cycle count analysis and timing measurements
- **Hardware validation**: FPGA testing with frequency scaling

## Timing Analysis

### Critical Path Analysis
- **Single-cycle critical path**: ALU multiplier (determining 70 MHz limit)
- **Pipelined critical path**: PC increment and register comparison
- **Optimization result**: 2x frequency improvement through pipeline partitioning

### TimingQuest Results
- **Estimated Fmax**: 71 MHz (conservative estimate)
- **Actual Fmax**: 140 MHz (board testing)
- **Discrepancy reasons**: Optimization settings, PLL strategy, manufacturing variation

## Advanced Features

### Forwarding Paths
- **EX to EX**: ALU result forwarding
- **MEM to EX**: Memory data forwarding
- **WB to EX**: Write-back data forwarding

### Memory Optimization
- **Dual-edge clocking**: Read on positive edge, write on negative edge
- **Phase-shifted access**: Optimized for pipeline timing
- **Bandwidth utilization**: Maximized memory throughput

## Limitations

- **3-stage pipeline**: Balanced complexity vs performance
- **In-order execution**: No out-of-order capabilities
- **Single-issue**: One instruction per cycle
- **Limited forwarding**: Cannot resolve all hazard types

## Future Enhancements

### Potential Improvements
- **Deeper pipeline**: 5-stage or more for higher frequencies
- **Branch prediction**: Reduce branch penalty
- **Superscalar execution**: Multiple instructions per cycle
- **Out-of-order execution**: Improved instruction-level parallelism
- **Cache memory**: Reduced memory access latency

### Advanced Features
- **Dynamic scheduling**: Hardware instruction reordering
- **Speculative execution**: Performance improvement through speculation
- **Vector processing**: SIMD instruction support
- **Floating-point unit**: Hardware FP operations

## Performance Analysis

### Execution Time Calculation
```
Runtime = Cycles × Clock Period
Factorial 6: 71 cycles × 7.14 ns = 507 ns
```

### Speed-up Achievement
```
Speed-up = Single-cycle Runtime / Pipelined Runtime
Speed-up = 1014 ns / 507 ns = 2.0x
```
