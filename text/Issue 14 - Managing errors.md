# What's the problem we're trying to solve ?

* Display errors to the user in the web application
* Give hints to the user about how to fix these errors
* Only the foxbox should know how to handle errors (example: automatically restart an adapter ?)

# Concrete issue: starting adapters

For reference, here is the current code: https://github.com/fxbox/foxbox/blob/998b6e0e1e1fd3cba3b711783eaa7000c2f66a0d/src/adapters/mod.rs#L61-L69

Basically we unwrap on any error. Some adapters are not returning an Error for some errors because of this.

The end-user can't see panics because he only accesses the app through a web browser.
So we need another way to log the error instead and let the user act on it (especially
we might need to be able to restart the foxbox from the browser -- but this is for another issue, I think).

# Some solutions

## 1. Error levels

* init methods return a taxonomied error enum.
* this error would show error levels: eg: Warn, Fatal, Critical.
* this error would contain the cause. The cause should be a real error that implements Display/Debug/Error.
* the box would log the result somewhere the user can look at (but this can be in a separate RFC). For now we could simply log warnings and panic at fatal and critical errors.

```rust
#[derive(Debug, Clone, Copy)]
enum ErrorLevel {
  Warn,
  Fatal,
  Critical,
}

#[derive(Debug)]
enum Error {
  AdapterStartError { level: ErrorLevel, cause: Error },
  CannotCreateTLSCert { level: ErrorLevel, cause: Error },
  ...
}

use std::error;
impl error::Error for Error {
...
}
```

Adapter init methods could return a Taxonomy-defined method that would be automatically converted to a Foxbox' error using a `From<TaxError>` implementation.

## 2. Custom Logger

* we replace env_logger with a logger that would keep the logs in a central place to display it through a web browser
* the adapters init function directly uses this to log what needs to be logged
* the adapters init function returns a Error only for Critical errors.

Q: how to scale to other errors ?
