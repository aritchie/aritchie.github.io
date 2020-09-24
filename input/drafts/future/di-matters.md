Title: Dependency Injection Matters to Mobile
Published: 2/9/2019
Tags:
    - Xamarin
    - Mobile
    - Architecture
    - Dependency Injection
---

I've been educating younger devs for a few years on the importance of decoupled software and things like dependency injection.  Recently, I had a young dev show a lot of excitement and tell me it was about building "robust software".  I immediately turned to him and said, buzzzz no guy!!!  It is about writing testable and maintainable components that separates responsibilities

Mistakes I've seen
1) Don't abstract away all of the tech, abstract away your busy.  Instead of SQLite.GimmieSomeBusinessData(), do IDataThingie.GimmieSomeBusinessData()