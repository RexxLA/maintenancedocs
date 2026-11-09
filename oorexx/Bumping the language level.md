# Bumping the ooRexx language level: a short guide for committers

ooRexx has two version numbers, and they move independently:

- The **interpreter version** (5.3.0): `REXX_CURRENT_INTERPRETER_VERSION` in
  `main/trunk/api/oorexxapi.h`. It changes with every release.
- The **language level** (6.07): `REXX_CURRENT_LANGUAGE_LEVEL`, in the same
  file. It changes only when the *language* changes.

The language level is what a program sees as the second word of
`PARSE VERSION` (`REXX-ooRexx_5.3.0(MT)_64-bit 6.07 9 Nov 2026`), as
`.RexxInfo~languageLevel`, and what a native program gets from the API's
`LanguageLevel()`. It is also stored in every program compiled with `rexxc`
(see section 4).

## 1. When the level must go up

The rule, as Rick McGuire gave it in [bug #2039](https://sourceforge.net/p/oorexx/bugs/2039/) (December 2025): the level
goes up when a release adds

- new built-in functions,
- new options to built-in functions,
- new instructions,
- new keywords on instructions,
- new directives.

New methods on existing classes typically do not bump it. Neither do bug
fixes or new error messages.

The level goes up **once per release**, not once per feature: the first
change of that kind after a release bumps it, and the following ones in the
same release do not. So before a release, the question is only "has
anything in the list above gone into trunk since the last release?"

## 2. History

| Level | Release | Notes |
|---|---|---|
| 6.04 | 4.2.0 | |
| 6.05 | 5.0.0 | Not bumped for 5.1.0, although (Rick, #2039) it probably should have been. |
| 6.06 | 5.2.0 | r13051 (Rony G. Flatscher, 17 Dec 2025), [bug #2039](https://sourceforge.net/p/oorexx/bugs/2039/). |
| 6.07 | 5.3.0 | r13448 (9 Nov 2026), [bug #2155](https://sourceforge.net/p/oorexx/bugs/2155/): ERRORTEXT accepts an error code nn.nnn (RFE #766). It also set the default to 6.07, which was wrong; fixed in r13449 ([bug #2156](https://sourceforge.net/p/oorexx/bugs/2156/), section 4). |

## 3. How to bump it

Since r13051 the level is no longer written by hand anywhere in the
interpreter: `PARSE VERSION` and `.RexxInfo` build it from
`REXX_CURRENT_LANGUAGE_LEVEL`. A bump touches two files in `main/trunk`
(the example goes from 6.06 to 6.07):

`api/oorexxapi.h`: add the new constant and make it current.

```c
#define REXX_LANGUAGE_6_06 0x00000606
#define REXX_LANGUAGE_6_07 0x00000607
#define REXX_CURRENT_LANGUAGE_LEVEL REXX_LANGUAGE_6_07
```

`interpreter/parser/LanguageLevel.hpp`: add the new level and make it the
maximum. **Leave the default and the minimum alone** (section 4 explains
both):

```c
    LanguageLevel0605 = 60500,                       // 060500
    LanguageLevel0606 = 60600,                       // 060600
    LanguageLevel0607 = 60700,                       // 060700
    DefaultLanguageLevel = LanguageLevel0605,        // the default language level for translated programs
    MinimumLanguageLevel = LanguageLevel0605,        // the lowest language level we can interpret
    MaximumLanguageLevel = LanguageLevel0607,        // the highest language level we can interpret
```

Then make the parser mark every package that uses the new feature, by
calling `requireLanguageLevel()` with the new level where the feature is
recognized (section 4).

Then rebuild and check `parse version v; say v` and
`say .RexxInfo~languageLevel`.

The documentation only uses the level in examples (rexxref: `PARSE VERSION`
in `instrc.xml`, `.RexxInfo` in `utilityclasses.xml`, `::ANNOTATE` in
`dire.xml`). They say "for example", so they need not change with every
bump, but it does no harm to bring them up to date.

## 4. Compiled programs (`rexxc`)

A compiled program carries the language level its package requires. When it
is loaded, the interpreter checks that this level lies between its
`MinimumLanguageLevel` and its `MaximumLanguageLevel`, and otherwise refuses
it with error 3.903, *Program "..." cannot be run by this version of the
REXX interpreter.*

The intention (Rick McGuire, in [bug #2039](https://sourceforge.net/p/oorexx/bugs/2039/)
and [bug #2155](https://sourceforge.net/p/oorexx/bugs/2155/)) is that
compiled programs move freely between releases in both directions:

- A program compiled with an older release runs on a newer one.
- A program compiled with a newer release runs on an older one, **as long
  as it uses no feature the older one lacks**. Take a program that works
  with an older version and compile it with the newer one: it still runs,
  unchanged, on the older version.

For that, every package starts at `DefaultLanguageLevel`, the base level
(6.05, ooRexx 5.0.0), and the parser raises it only when the program uses a
newer feature: when it recognizes one, it calls
`LanguageParser::requireLanguageLevel()` with the level where that feature
was introduced (the call only ever raises the level). The new built-in
function `GC` of 5.2.0, for instance, marks its package as requiring 6.06. So every feature that bumps the language level (section 1) also needs
its `requireLanguageLevel()` call.

This is what the interpreter does since r13449
([bug #2156](https://sourceforge.net/p/oorexx/bugs/2156/)). Before that,
`DefaultLanguageLevel` had been set to the current level (6.06 in r13051,
6.07 in r13448) and nothing called `requireLanguageLevel()`, so every
program compiled by 5.2.0 was refused by 5.0.0 and 5.1.0 (and, briefly, every
program compiled by trunk by every earlier release), even when it used nothing
new. The features marked since r13449:

| Level | Feature | Where it is marked |
|---|---|---|
| 6.06 | the `GC` built-in function | `LanguageParser::requireBuiltinLanguageLevel()`, called for a function call or `CALL` (also `CALL ON ... NAME`) once it is known to resolve to the built-in function and not to an internal label |
| 6.06 | `::OPTIONS NUMERIC INHERIT` and `NOINHERIT` | `DirectiveParser.cpp`, `SUBDIRECTIVE_NUMERIC` |

ERRORTEXT with an error code nn.nnn (6.07) is not marked: it depends on the
value of the argument, which the parser cannot see in general. On an older
release such a call fails with error 40.12, as it would if the program had
been written for that release.

The general rule (Rick McGuire, in
[bug #2155](https://sourceforge.net/p/oorexx/bugs/2155/)): the required level
exists to guarantee that the saved image can be turned back into live
objects. A new option that leaves the image valid, and that an older release
rejects with an appropriate error instead of crashing, does not need to be
marked. That covers options chosen by the value of an argument, which the
translator cannot detect, and usually new optional arguments of a built-in
function too (Rick is "on the fence" about these).

A new built-in function is marked by adding a `case` to
`requireBuiltinLanguageLevel()`; any other feature, by a
`requireLanguageLevel()` call where the parser recognizes it.

### Changes to the image layout

There is one more factor. A compiled program is a flattened image of the
interpreter's internal objects (the instructions, the expression terms...).
**Any change to the fields or the layout of a class whose objects are
stored in the image is an incompatible change**: the other release cannot
turn the image back into live objects. When the class appears in every
compiled program, `DefaultLanguageLevel` itself has to go up. According to
Rick McGuire ([bug #2156](https://sourceforge.net/p/oorexx/bugs/2156/)),
that happens with:

- extra fields in a fundamental class such as String or Array;
- changes to Package, which is especially exposed, because things tend to
  get added to it;
- any change to the object headers, which invalidates every image;
- a new class of object that gets stored in every compiled program.

Classes written in Rexx, such as the extensions (`.LogicalWrapper`, say),
are not affected: a compiled program does not store their objects, and
loads them with `::REQUIRES` when it runs.

When the class appears only in some programs, the change can usually be
avoided with a subclass. If a new feature needs an
extra field in, say, an instruction object, do not add the field to the
instruction class: add a subclass of it that has the field, have the parser
create an instance of the subclass only when the program uses the new
feature, and call `requireLanguageLevel()` there with the level where the
feature was introduced. Programs that do not use the feature keep the old
layout, and stay readable by older releases.

The image header also carries `METAVERSION`
(`interpreter/classes/support/ProgramMetaData.hpp`, 43 since r12349, 2022),
which must match exactly; changing it makes every image incompatible in
both directions.

## 5. Tests

`RexxInfo.testGroup` checks that `.RexxInfo~languageLevel` is not empty and
that `.RexxInfo~name` equals `PARSE VERSION`; `METHOD.testGroup` (API)
checks that `LanguageLevel()` agrees with `PARSE VERSION`. No test pins the
value itself, so a bump needs no test changes, and a forgotten bump is not
caught by the test suite. `rexxc.testGroup` `test_bug_2156` (r13450) compiles
small programs with `rexxc` and checks the level each one requires; add a
case there for every feature that gets a `requireLanguageLevel()` call.
