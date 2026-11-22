# pep621: parent `pixi.lock` not updated for pixi workspace members

## Setup

- `pixi.toml` and `pixi.lock` are at the repository root.
- `pixi.toml` installs `subpackage/` as an editable path dependency:

  ```toml
  [pypi-dependencies]
  subpackage = { path = "subpackage", editable = true }
  ```

- `subpackage/pyproject.toml` pins `six==1.16.0` and has no `[tool.pixi]` section.
- As a control, `pixi.toml` itself pins `idna==3.6`.
- CI runs `pixi install --locked`, which fails when `pixi.lock` is out of date.
- `requires-pixi` pins the pixi version so Renovate and CI write and check `pixi.lock` with the same pixi.

pixi records the `requires_dist` of path dependencies in `pixi.lock`, so changing a dependency in `subpackage/pyproject.toml` requires `pixi.lock` to be regenerated.

## Current behavior

- The `pixi` manager updates `idna` in `pixi.toml` and the PR also updates `pixi.lock`.
  CI passes.
- The `pep621` manager updates `six` in `subpackage/pyproject.toml`, but the PR does not update `pixi.lock`.
  CI fails with:

  ```
  Error:   × lock file not up-to-date with the workspace
  ```

The `pep621` manager only looks for a `pixi.lock` next to the `pyproject.toml` it changed, so it never finds the workspace lock file in the parent directory.

## Expected behavior

The PR that updates `six` in `subpackage/pyproject.toml` also updates the root `pixi.lock`, and CI passes.

A `pyproject.toml` below a pixi workspace that the workspace does not reference as a path dependency should not cause `pixi.lock` to be regenerated.

## Link to the Renovate issue or Discussion

<!-- TODO: link to the Renovate PR or Discussion -->
