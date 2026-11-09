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
| 6.07 | 5.3.0 | r13448 (9 Nov 2026), [bug #2155](https://sourceforge.net/p/oorexx/bugs/2155/): ERRORTEXT accepts an error code nn.nnn (RFE #766). |

## 3. How to bump it

Since r13051 the level is no longer written by hand anywhere in the
interpreter: `PARSE VERSION` and `.RexxInfo` build it from
`REXX_CURRENT_LANGUAGE_LEVEL`. A bump touches two files in `main/trunk`
(the example is r13448, from 6.06 to 6.07):

`api/oorexxapi.h`: add the new constant and make it current.

```c
#define REXX_LANGUAGE_6_06 0x00000606
#define REXX_LANGUAGE_6_07 0x00000607
#define REXX_CURRENT_LANGUAGE_LEVEL REXX_LANGUAGE_6_07
```

`interpreter/parser/LanguageLevel.hpp`: add the new level, and make it the
default and the maximum. **Leave the minimum alone**: it is the lowest
level whose compiled programs this interpreter can still run.

```c
    LanguageLevel0605 = 60500,                       // 060500
    LanguageLevel0606 = 60600,                       // 060600
    LanguageLevel0607 = 60700,                       // 060700
    DefaultLanguageLevel = LanguageLevel0607,        // the default language level for translated programs
    MinimumLanguageLevel = LanguageLevel0605,        // the lowest language level we can interpret
    MaximumLanguageLevel = LanguageLevel0607,        // the highest language level we can interpret
```

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

The design (Rick, #2039) is that the parser raises a package's required
level only when the program uses a newer feature, through
`LanguageParser::requireLanguageLevel()`, so that a compiled program that
uses no new feature still runs on older interpreters. As of r13448, nothing
calls `requireLanguageLevel()`: every package starts at
`DefaultLanguageLevel`, so every compiled program requires exactly the
level of the interpreter that compiled it. In practice, after a bump, a
program compiled with the new release is refused by the previous one, even
if it uses nothing new. Programs compiled with older releases keep running,
as long as their level is not below the minimum.

## 5. Tests

`RexxInfo.testGroup` checks that `.RexxInfo~languageLevel` is not empty and
that `.RexxInfo~name` equals `PARSE VERSION`; `METHOD.testGroup` (API)
checks that `LanguageLevel()` agrees with `PARSE VERSION`. No test pins the
value itself, so a bump needs no test changes, and a forgotten bump is not
caught by the test suite.
