# Example 2 of using the galaxy-k8s-action with ABM 

This is an example repository showing how to use the [galaxy-k8s-action](https://github.com/ksuderman/galaxy-k8s-action), used for installing Galaxy onto Kubernetes, with [abm](https://github.com/galaxyproject/gxabm) used to run Galaxy workflows.

This example includes `abm` configuration files in the respository.  See [this repository](https://github.com/ksuderman/galaxy-k8s-action-example-1) for an exmple of using `galaxy-k8s-action` with `abm` using configuration defined in the workflow.

```yaml
name: Run a Galaxy benchmark with ABM
on:
  workflow_dispatch:
    inputs:
      values-file:
        description: 'Values file to use for the Galaxy Helm chart'
        required: false
jobs:
  install-galaxy:
    name: Install Galaxy and run a benchmark with ABM
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          persist-credentials: false

      - uses: ksuderman/github-action-galaxy-k8s@dev
        with:
          api-key: secret
          values-file: ${{ inputs.values-file }}
          admin-users: admin@example.com

      - name: Get Helm values
        shell: bash
        run: |
          helm get values galaxy -n galaxy --all --output yaml

      - name: Install ABM
        shell: bash
        run: |
          pip install gxabm
          abm --version

      - name: Run ABM using configuration defined in the repository
        shell: bash
        run: |
          abm local user create admin admin@example.com galaxypassword
          key=$(abm local user key admin)
          abm config key local $key
          # Import a workflow using an alias defined in ~/.abm/workflows.yml 
          abm local workflow import variant
          # Import a history using an alias defined in ~/.abm/histories.yml 
          abm local history import variant-2g
          # Validate and run the benchmark
          abm local benchmark validate benchmark.yml          
          abm local benchmark run benchmark.yml 

      - name: List the results
        shell: bash
        run: |
          abm local jobs list
          abm local history list

```