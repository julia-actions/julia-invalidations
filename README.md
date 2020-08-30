# julia-invalidations
Uses [`SnoopCompile.@snoopr`](https://timholy.github.io/SnoopCompile.jl/stable/snoopr/) to evaluate number of invalidations caused by `using Package` or a provided script


## Usage

This is a composite github action, that can be inserted into a github action on the target repo to evaluate number of invalidations

For instance, this will evaluate number of invalidations that branch/PR has vs. master, and fail if the number increases. Both runs happen on the same julia nightly, so that the comparison in # invalidations is less sensitive to changes in base.

- Create an action file in the desired repo. i.e. `.github/workflows/InvalidationFlagger.yml`

```
name: Invalidations
on: [push]

jobs:
  evaluate:
    runs-on: ubuntu-latest
    steps:
    - uses: julia-actions/setup-julia@v1
      with:
        version: 'nightly'
    - uses: actions/checkout@v1.0.0
    - uses: julia-actions/julia-buildpkg@latest
    - uses: julia-actions/julia-invalidations@master
      id: invs_pr
    
    - uses: actions/checkout@v1.0.0
      with:
        ref: 'master'
    - uses: julia-actions/julia-buildpkg@latest
    - uses: julia-actions/julia-invalidations@master
      id: invs_master
    
    - name: Report invalidation counts
      run: |
        echo "Invalidations on master: ${{ steps.invs_master.outputs.total }} (${{ steps.invs_master.outputs.deps }} via deps)"
        echo "This branch: ${{ steps.invs_pr.outputs.total }} (${{ steps.invs_pr.outputs.deps }} via deps)"
      shell: bash
    - name: PR doesn't increase number of invalidations
      run: |
        if (( ${{ steps.invs_pr.outputs.total }} > ${{ steps.invs_master.outputs.total }} )); then
            exit 1
        fi
      shell: bash
```

By default, the action will evaluate `using Package` where `Package` is the name of the julia repo that the action runs on.
A custom script can be provided by passing `test_script`. Note that both runs should be given the same script

i.e.
```
- uses: julia-actions/julia-invalidations@master
  id: invs_pr
  with:
    test_script: 'using Package; Package.foo(1)'
```
