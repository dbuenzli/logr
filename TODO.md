* Rewrite that in a decent functional programming language.
* `$LOG_TITLE` should be escaped in templates.
* Support for entry preparation. Add `log/published` in entries.
  Only those entries are build on a `logr publish`. Any are
  build on `logr build` for preview. Allow to `logr renew DIR` 
  unpublished entries, this will mv the dir to current date
  and update its `log/stamp`.
* Tags ? Not in the ring spirit but why not.
