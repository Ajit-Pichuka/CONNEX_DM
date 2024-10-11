# Changelog
## [1.0.5] - 2022-10-04
### Added
- Dispatch-dry workflow
``` This workflow will run automatically on PR open, letting you know on which scripts gonna be executed after merge. ```
## [1.0.4] - 2022-07-11
### Updated
- Github-hosted runners

```We'no longer dispatching events to orchestration with self-hosted runners. No breaking changes```
## [1.0.3] - 2022-06-06
### Added
- UAT environment

```Backend is ready to support UAT environment. Feel free to create branch "UAT" and deploy your DDL's there.```
## [1.0.2] - 2022-03-13
### Bugfix
- Cherry-pick

```Fixed a bug when cherry-picking wasn't working properly with "merge commit" type.```
## [1.0.1] - 2022-03-10
### Updated
- Cherry-pick configs

```All the configs, regardless of the environment, will be stored inside config directory. Transfer your existing configs from qa, prod directories one level up, and delete these directories.```

## [1.0.0] - 2022-01-28
### Added
- Cherry-Pick.
- Forced "V" script deployment & execution.

```Previously it was only possible to promote changes by merging source and target branches. Sometimes you want to pick and choose which changes should go and which ones stay. Cherry-Pick workflow will help you to achieve that. ```



