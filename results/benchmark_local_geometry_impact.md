# LocalGeometry Performance Benchmark Results

## Execution Time Results

### Section 1: Basic Geometry Access

| Operation | Time (μs) | Overhead vs Baseline |
|-----------|-----------|----------------------|
| baseline simple | 16.38 | 0.0% |
| full lg jacobian | 18.40 | 12.3% |
| 2b pointwise lg j | 17.82 | 8.8% |
| 2c pointwise lg j stack | 18.66 | 13.9% |
| 2d pointwise lg j noinline | 19.61 | 19.7% |
| 2e fd localgeom constructor | 14.52 | -11.4% |
| 2f f x lg | 16.70 | 2.0% |
| 2g lambda f x lg | 17.14 | 4.6% |
| 2h f x lg noinline | 17.94 | 9.5% |
| 2i lambda f x lg noinline | 18.20 | 11.1% |
| full lg multiple | 20.20 | 23.3% |
| extracted j | 17.92 | 9.4% |
| simplified lg | 18.07 | 10.3% |

### Section 2: Struct Size Impact on Inlining

| Struct Type | Size (bytes) | Time (μs) | Overhead vs Baseline |
|-------------|--------------|-----------|----------------------|
| two field access | 16 | 17.47 | 6.7% |
| four field access | 32 | 18.35 | 12.0% |
| eight field access | 64 | 17.97 | 9.7% |
| sixteen field access | 128 | 18.09 | 10.4% |

### Section 3: Projection Operations

| Operation | Time (μs) | Overhead vs Vector Baseline |
|-----------|-----------|----------------------------|
| vector baseline | 18.06 | 0.0% |
| project full lg | 17.63 | -2.4% |
| multiple scalar access | 19.45 | 7.7% |

## Memory Footprint

| Structure | Total Size (MB) | Size per Point (bytes) | Ratio vs Scalar |
|-----------|-----------------|------------------------|-----------------|
| Scalar field | 0.000 | 128.0 | 1.0x |
| TwoFieldGeom | 0.000 | 256.0 | 2.0x |
| FourFieldGeom | 0.000 | 512.0 | 4.0x |
| EightFieldGeom | 0.001 | 1024.0 | 8.0x |
| SixteenFieldGeom | 0.002 | 2048.0 | 16.0x |
| Full LocalGeometry | 0.003 | 2688.0 | 21.0x |

## Key Findings

### Basic Geometry Access
- Full LocalGeometry (J only) overhead: 12.3%
- Extracted J overhead: 9.4%

### Struct Size Impact
- TwoFieldGeom (16 bytes): 6.7%
- FourFieldGeom (32 bytes): 12.0%
- EightFieldGeom (64 bytes): 9.7%
- SixteenFieldGeom (128 bytes): 10.4%

### Projection Operations
- Covariant to Contravariant: -2.4%

## Assessment

⚠️ **SIGNIFICANT OVERHEAD** - Refactoring recommended

- Consider extracting commonly-used fields at kernel entry
- Profile real physics kernels with nsys for detailed analysis

## Next Steps

1. **Profile real physics kernels**: These synthetic benchmarks test individual operations. Real kernels may show different behavior.

2. **Use nsys for detailed analysis**:
   ```bash
   ./scripts/run-nsys.sh --output=results/nsys/benchmark_lg \
       julia --project scripts/benchmark_local_geometry_impact.jl
   nsys stats results/nsys/benchmark_lg.nsys-rep
   ```

3. **Verify inlining**: Use `@code_llvm` and `@device_code_ptx` to inspect compiler output.

4. **Test with realistic field operations**: Include projections in gradient/divergence operators.
