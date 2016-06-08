# Wrapping cargo with mach, the command dispatcher

## Initial problems

* You don't get an early warning when you're compiling against the wrong rustc compiler: [fxbox/foxbox#516](https://github.com/fxbox/foxbox/issues/516)
* `cargo test` doesn't execute the tests in `components/**`: [fxbox/foxbox#449](https://github.com/fxbox/foxbox/issues/449)
* There are many independent scripts to execute tests. See `tools/execute-*.sh`
* Dependencies have to be installed manually [fxbox/foxbox#157](https://github.com/fxbox/foxbox/issues/157)
* We have `run.sh` (not under `tools/`), now that `cargo run` doesn't launch foxbox: [fxbox/foxbox#440](https://github.com/fxbox/foxbox/pull/440)

## Pros

* mach becomes the central starting point.
* commands can be done prior to `cargo build`, allowing rustc revision check, and dependencies installation (if distribution and package manager are correctly identified)
* `./mach test` will pipe every test that can be executed (making `tools/execute-all-rust-tests.sh` useless)
* Incidentally, `./mach run` will take over `./run.sh`
* We could also get rid of `toosl/travis-*.sh` in favor of commands like `./mach travis linux-cross-compile`

## Cons

* There is no central repository to fork. We will have to import our own version, just [like Servo did](https://github.com/servo/servo/pull/3230).
* Python code will be added to the repo. mach core is [a python package](https://dxr.mozilla.org/mozilla-central/rev/ec20b463c04f57a4bfca1edb987fcb9e9707c364/python/mach/docs/index.rst#43), and we have to write python to define commands. The driver part can be copied from [mozilla-central](https://dxr.mozilla.org/mozilla-central/source/mach), or in a simpler form, from [servo](https://github.com/servo/servo/blob/master/mach)

## Alternatives

* Write a smaller command dispatcher, in one of the language already living in the repo (rust, javascript, shell).

## Conclusion: Should we go with it?
