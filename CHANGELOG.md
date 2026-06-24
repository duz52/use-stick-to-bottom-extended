# 1.2.0

Fork release maintained by https://github.com/duz52/.

- Renamed the package metadata to `use-stick-to-bottom-extended`.
- Added `resize: false` / `resize={false}` to disable ResizeObserver-driven automatic scroll-to-bottom after the initial layout.
- Added `append` / `append={...}` to control automatic scroll-to-bottom when a new direct child is appended to the content element.
- Documented one-shot send scrolling for chat UIs that should scroll only when the user was already at the bottom and should not keep sticking during window resize.

# 1.1.0

Stable - Bumped version to signify a release with the new pausing feature https://x.com/samddenty/status/1907636864409817120. Also to start using proper semver, I was using the wrong semver versions for previous releases.

# 1.0.52

Stable - added pausing feature.

- Also addressed in one of these releases, the `targetScrollTop` option and made it more functional.
  - This is used in https://github.com/samdenty/react-ai-flow to delay the scrolling of the hook to the last revealed animated element

# 1.0.47 - 1.0.51

Testing publishes that are a bit broken, as I was testing. Will mark as beta tag from now on instead of latest.
