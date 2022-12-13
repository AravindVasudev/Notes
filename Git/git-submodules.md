# [7.11 Git Tools - Submodules](https://git-scm.com/book/en/v2/Git-Tools-Submodules)

- Submodules allow you to keep a Git repository as a subdirecotry of another
  Git repository.
- This lets you clone another repository into your project and keep your commits
  separate.

## Starting with Submodules
- To add a new submodule you use the `git submodule add` command with the
  absolute or relative URL of the project you would like to start tracking.
- By default, submodules will add the subproject into a directory named the same
  as the repository.
- The mapping between the project's URL and the local subdirectory is stored in
  `.gitmodules` config.
- `.gitmodules` is version-controlled.
- When you run `git diff`, git doesn't track files from submodules but instead
  sees it as a particular commit from that repository.
- `git diff --cached --submodule` -> diffs add some submodule info.
- When you clone a project with submodules, you get directories that contain
  submodules but none of the files within them.
  - To get them, run two commands:
    - `git submodule init` -> Init local config file.
    - `git submodule update` -> Fetch all the data from that project.
    - `git submodule update --init` -> shorthand for the above two commands.
  - `git clone --recurse-submodules` -> Will init all submodules.

## Working on a Project with Submodules
- To check for new work in a submodule, you can go into the directory and run
  `git fetch` and `git merge` the upstream branch to update the local code.
- `git submodule update --remote` -> Git will go into your submodules and fetch
  and update for you.

## Working on a Submodule
- If you haven’t committed your changes in your submodule and you run a
  submodule update that would cause issues, Git will fetch the changes but not
  overwrite unsaved work in your submodule directory.

## Publishing Submodule Changes
- `git push --recurse-submodules=check` -> Ask Git to check that all your
  submodules have been pushed properly before pushing the main project.
  - `--recurse-submodules=check|on-demand`
    - `check` -> will make push simply fail if any of the committed submodule
      changes haven’t been pushed.
    - `on-demand` -> pushes all the submodules for you.

## Submodule Tips
- `git submodule foreach 'some_command'` -> Helper to run some command for every
  submodule.
