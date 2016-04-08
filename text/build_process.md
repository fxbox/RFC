# Build process and dependency management

## Initial problems

This RFC is intended to answer to these questions:
> * Where should an adapter live?
> * How can we prevent a change in Taxonomy to require an update of many repos?
> * How can we make upgrades to the latest rust nightly easier (aka "rust up")?


Currently, Foxbox is built with this dependency tree:
```
foxbox ─┬─> adapter_0   ─┬─> taxonomy
        ├─> adapter_1   ─┤
        ├─> adapter_n   ─┤
        ├─> thinkerbell ─┤
        └────────────────┘
```
Each of these dependencies lives in a separate git repository, and as a corollary, in a separate crate.

We link these dependencies with each other thanks to git revisions. When cargo deals with git revisions, it will *always* build a crate with the specified revision. In other words, when `adapter_0` depends on taxonomy revision A but `foxbox` depends on revision B (which is more recent), cargo will first build revision A, then revision B. In addition to slowing down the compile time, we recurrently face compile errors due to revision. Moreover, if you're not familiar with either cargo dependency resolution or our dependency tree: you have to watch closely the cargo build steps to figure out revision A is being compiled again, and producing this error which should have been fixed by revision B.

As a consequence, if a base dependency (like taxonomy) needs to be updated, we can end up having 3 or 4 PR to write, and schedule landing.


## Criteria

The best solution should comply with these criteria (not ordered):

* Daily development tasks
 * Fast compile time
 * Easy automated testing
 * Easy dependency management
 * Easy upgrade to the latest rustc nightly
 * Should spot breakages between crates (while we stabilize the API)
 * Fast CI results
* Fallbacks
  * Should not depend too much on Github (crates.io as another package repository)
  * Should support forks (e.g.: iron-fork)
* Community participation
  * Writing a new adapter should be easy
  * Contribution flow should be unique
* Initial set up effort

## Solutions

### Keep small repos as is

#### Use git submodules

Git submodules have the same effect as pinning the git revision with Cargo.

#### Always point to master

One immediate solution could be to always point our dependencies to the latest master. This has 2 major consequences:

* If a bottom-level dependency lands on master, we might break the compilation of another crate. Hence time a bottom-level dependency is updated, we have to run the tests of each crates depending on it. As a consequence, the bottom-level dependency has to know what actors are using it.

* Rust ups will still have to be meticulously scheduled.

#### Use Semantic Versioning

Cargo [supports SemVer](http://doc.crates.io/crates-io.html#using-cratesio-based-crates). This less permissive approach would allow us to be explicit on when our API changes. For example, in Cargo.toml:
``` toml
taxonomy = "^0.4"
```

Based on an offline discussion with one Servo dev, it is worth to mention that Cargo might take 2 versions of the same crate. Consider this example:
```toml
taxonomy = "^0.4"
```
```toml
taxonomy = "0.4.1"
```
If the most recent version of taxonomy is 0.4.2, Cargo will pick that one for the first build and pick 0.4.1 for the second one. Cargo won't pick the version that matches every constraint (which should have been 0.4.1). Therefore, we might build taxonomy twice, just like with master.

Likewise, if somebody inadvertently breaks the API without increasing the major version (v0.5, in the example), we will break the other dependency. Thenceforth, we will need to run the tests of every actor using that dependency.

#### Automate revisions bumps

We could write a bumper script that will create a PR for each repo that depends on a bottom-level dependency and automatically lands it once all the PRs are green and an r+ was given on the initial PR. This seems to be the hardest solution in terms of initial set up. We will also rely more on Github than we currently do.

### Have one repo with one single crate

Bringing back taxonomy, thinkerbell and the adapters in foxbox will give a convenient way to find integration errors, as all the tests can be run locally without any particular inter-repository configuration.

One major drawback would be the compile time. For more details about crate compile time, see [fxbox/foxbox#113](https://github.com/fxbox/foxbox/issues/113).

### Have one main repo with many crates

Everything that is tightly coupled or subject to heavy changes will live in the main repo. Other dependencies will remain in other repos. As of today, the repos which are candidates to be in the main repo are:
* foxbox (aka the main repo)
* openzwave-adapter
* thinkerbell
* taxonomy

`cargo build` in the root directory will spot compile errors. A script to run `cargo test` in each submodule will be needed. If one would like to build/test only one module, we just need to cd to the right folder. Nonetheless, our CI jobs will likely take longer as more tests will need to be run in the same Travis job.

Even if they are in the same repo, we can publish module on crates.io. E.g: [regex](https://github.com/rust-lang-nursery/regex) lives in the same repo as [regex_macros](https://github.com/rust-lang-nursery/regex/tree/master/regex_macros) and both are on crates.io (see [here](https://crates.io/crates/regex) and [there](https://crates.io/crates/regex_macros)).

Foxbox would be the main entry point for contributors and we could describe the whole flow there.



## Criterium/solution matrix

| Criterium                  | Git submodules     | Point to master    | SemVer             | Automate bumps     | 1 repo / 1 crate   | 1 repo / many crates |
| -------------------------- | ------------------ | ------------------ | ------------------ | ------------------ | ------------------ | -------------------- |
| Fast compile time          | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :x:                | :white_check_mark:   |
| Easy dependency management | :x:                | :white_check_mark: | :white_check_mark: | :x:                | :white_check_mark: | :white_check_mark:   |
| Easy rust up               | :x:                | :x:                | :x:                | :grey_question:    | :white_check_mark: | :white_check_mark:   |
| Easy component testing     | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :x:                | :white_check_mark:   |
| Easily spot integration breakage   | :x:        | :x:                | :x:                | :white_check_mark: | :white_check_mark: | :white_check_mark:   |
| Fast CI results            | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :x:                | :x:   |
| Low github dependency      | :x:                | :x:                | :white_check_mark: | :x:                | :white_check_mark: | :white_check_mark:   |
| Easily write new adapters  | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :x:                | :grey_question:      |
| Unique contribution flow   | :x:                | :x:                | :x:                | :x:                | :white_check_mark: | :white_check_mark:   |
| Set up effort              | :white_check_mark: | :white_check_mark: | :white_check_mark: | :x:                | :white_check_mark: | :white_check_mark:   |

## Conclusion: Chosen Solution

TBD
