Taken from comment ( https://github.com/conda-forge/cuda-version-feedstock/issues/1#issuecomment-1462998398 ) summarizing the intended usage of this package.

# Summary

## In-scope for `cuda-version`

Questions/demands listed below are addressed:
- Which CUDA versions a package would support? 
- How a particular CUDA library is tied to a specific CUDA version?
- How to pull in dependencies that align with a particular CUDA Toolkit release?
- What CUDA compiler version a package was built with?
- The ability to get a specific shipped full toolkit at a given version, i.e. 12.0 or 12.1.
- The ability to get specific packages at a given toolkit version, i.e. 12.0 or 12.1.
- Not to allow solving to an incompatible environment

## Out-of-scope for `cuda-version`

> What CUDA driver version requirements might exist?

Driver requirement should be set up using the `__cuda` virtual package. For package maintainers/users who wish to overwrite (at their own risk), use the environment variable `CONDA_OVERRIDE_CUDA`.

> How CUDA version tracking support ties into [CUDA compatibility](https://docs.nvidia.com/deploy/cuda-compatibility/index.html)

- For CUDA components from CTK, we currently disallow mix-n-match. This constraint is something we will revisit in the future. 
- For downstream packages depending on CUDA, use `cuda-version >=X.0,<X+1` to allow minor version compatibility (see below).

> The ability to get specific packages not tied to a given toolkit version, i.e. I may want to get Thrust 2.0.1

Unclear if possible under the "tight constraint" scheme that we plan to adopt.

## Scheme

- This feedstock (`cuda-version`):
  - Set up the driver requirement using `__cuda` for a given CTK major version X (see https://github.com/conda-forge/cuda-version-feedstock/issues/1#issuecomment-1462752780).
- nvcc compiler:
  - Pin a range of `cuda-version`, with the lower bound  being the CTK version X.Y that it comes with
- CTK components:
  - Pin at the exact `cuda-version` corresponding to the CTK version X.Y that they come with. Exceptions should be discussed on a case-by-case basis.
  - This applies to all flavors (`lib`, `lib-dev` and `lib-static`)
- Downstream packages:
  - Depend on needed CTK components without version constraints, and set the version constraint on CUDA by depending only on `cuda-version` (plus `__cuda`, if applicable)
- Downstream users:
  - Use `cuda-version` as the CTK version selector to set up a consistent environment

## Examples

### For maintainers of nvcc

Example: nvcc from the CTK version X.Y
```yaml
- run_exports:
  - cuda-version >=X.Y,<X+1
```

### For maintainers of all CTK components

Example: cuBLAS from the CTK version X.Y
```yaml
- host:
  - cuda-version X.Y
- run:
  - {{ pin_compatible("cuda-version", max_pin="x.x") }}
```

Example: cuSOLVER depends on cuBLAS from the same CTK version X.Y
```yaml
- host:
  - cuda-version X.Y
  - libcublas
- run:
  - {{ pin_compatible("cuda-version", max_pin="x.x") }}
  - {{ pin_compatible("libcublas", max_pin="x.x") }}
```

### For maintainers of GPU packages depending on CUDA

Example: A package depends on cuBLAS and supports CUDA minor version compatibility
```yaml
- run:
  - cuda-version >=X.0,<X+1
  - libcublas
```

### For users of GPU packages

Example: Set up an environment compatible with CTK 12.1
```
conda install -c conda-forge cupy rmm "cuda-version=12.1"
```

Example: Set up an environment compatible with CTK 12 (of any minor version)
```
conda install -c conda-forge cupy rmm "cuda-version=12"
```

Example: Just set up a legit CUDA environment 
```
conda install -c conda-forge cupy rmm
```
