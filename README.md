# Add Paths to Release

## Description

This action updates release notes to mention which paths were modified by each PR. You have to generate the release notes separately (for example, with https://github.com/release-drafter/release-drafter).

I've implemented this for smoothing out the release of shared Terraform modules, which have a lot of different authors making changes, and then get pulled into a `live` repository to update infrastructure environments. We have jobs that update all environments (to keep them reasonably fresh), and thus it's extremely helpful for the person reviewing the auto-update PRs to match the plan output up with what's actually changed.

## Usage

Here's a simple workflow:
```
name: Draft Release
                                                             
on:                                                          
  push:                                                      
    branches:                                                
      - main                                                 
  pull_request:                                              
    types: [opened, reopened, synchronize]                   
                                                             
jobs:                                                        
  update-release-draft:                                      
    runs-on: ubuntu-latest                                   
    steps:                                                   
      - uses: release-drafter/release-drafter@v5             
        id: release-draft                                    
        env:                                                 
          GITHUB_TOKEN: ${{ github.token }}                  
      - name: Add changed paths                              
        uses: fac/action-add-paths-to-release@v1
        with:                                                
          release-id: ${{ steps.release-draft.outputs.id }}  
          exclude-pattern: ^(examples/|[^/]*.[.]md)
        env:
          GITHUB_TOKEN: ${{ github.token }}                  
```

Assuming there were a few PRs merged since the last release which altered these paths:

* PR 1
  `services/foo`, `examples/foo`
* PR 2
  `services/bar`, `docs/bar.md`
* PR 3
  `README.md`

And using a simple template for Release Drafter, the notes for the draft release would look like:

```
## What's changed?

* `services/foo` PR 1 (#1)
* `docs/bar.md`, `services/bar` PR 2 (#2)
* PR 3 (#3)
```

## Inputs

* `release-id`
  Which release to add affected paths to PRs listed in the release's body
* `path-prefix-parts`
  How much of the path to add (default is 2)
* `exclude-pattern`
  A regular expression that matches paths to not list as affected by a PR. Test it out by echoing lines to `grep -Ev <pattern>`.

## Outputs

* `body`
  The rewritten body of the release, quoted and escaped suitable fo `printf`ing.

## Environment Variables

* `GITHUB_TOKEN`
  A token for interacting with the GitHub API, for retrieving the PRs and release & updating the release notes. Pass `${{ secrets.GITHUB_TOKEN }}`!

## Licence

```
Copyright 2021 FreeAgent Central Ltd.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

   http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```
