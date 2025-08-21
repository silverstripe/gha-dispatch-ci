# gha-dispatch-ci

Used to trigger CI workflows on a schedule. This workflow allows multiple branches to have CI workflows run on a schedule, rather than just the default branch.

For modules that are to be scheduled once per week, the cron must be run three times per week on consecutive days. This is done because there is logic in action.yml to run one jobs on consecutive days for:

- Current major, next-minor e.g. "6"
- Current major, next-patch e.g. "6.0"
- Previous major, next-patch e.g. "5.4"

### Usage

**.github/workflows/dispatch-ci.yml**
```yml
name: Dispatch CI

on:
  # Every Tuesday,Wednesday,Thursday at 1:00pm UTC
  schedule:
    - cron: '0 13 * * 2,3,4'
  workflow_dispatch:
    inputs:
      major_type:
        description: 'Major branch type'
        required: true
        type: choice
        options:
          - 'dynamic'
          - 'current'
          - 'next'
          - 'previous'
        default: 'dynamic'
      minor_type:
        description: 'Minor branch type'
        required: true
        type: choice
        options:
          - 'dynamic'
          - 'next-minor'
          - 'next-patch'
        default: 'dynamic'

permissions: {}

jobs:
  dispatch-ci:
    name: Dispatch CI
    # Only run cron on the silverstripe account
    if: (github.event_name == 'schedule' && github.repository_owner == 'silverstripe') || (github.event_name != 'schedule')
    runs-on: ubuntu-latest
    permissions:
      contents: read
      actions: write
    steps:
      - name: Dispatch CI
        uses: silverstripe/gha-dispatch-ci@v1
        with:
          major_type: ${{ inputs.major_type }}
          minor_type: ${{ inputs.minor_type }}
```

### Inputs:

#### Major Type
The major version type to target, can be `(blank)`, `dynamic`, `current`, `next`, `previous`. Default is `dynamic`.
`major_type: current`

#### Minor Type
The minor version type to target, can be `(blank)`, `dynamic`, `next-minor`, `next-patch`. Default is `dynamic`.
`minor_type: next-minor`
