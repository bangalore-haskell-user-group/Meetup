# Haskell Optimization Case Study - Unicode Normalization

Presented by [Harendra Kumar][presenter]

This talk is about how we started with a simple unicode normalization
implementation in Haskell which performed 68x slower than the best C++
implementation (ICU) and incrementally optimized it to bring it pretty close to
the ICU implementation performance.

See the notes for specific optimizations, corresponding code changes and to
what extent they helped. We also note down lessons learnt and some GHC issues
which can help us do even better.

## Links
- [Meetup Page][meetup]
- [Presentation Notes][notes]
- [unicode-transforms](https://www.github.com/harendra-kumar/unicode-transforms)

[presenter]: https://www.github.com/harendra-kumar
[meetup]: https://www.meetup.com/The-Bangalore-Haskell-User-Group/events/235671008/
[notes]: ./Notes.rst
