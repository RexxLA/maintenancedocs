# Adding or changing an ooRexx error message: a short guide for committers

All error messages live in one file, `main/trunk/interpreter/messages/rexxmsg.xml`.
Nine files are generated from it with XSLT and **committed together with it**,
so that nobody needs an XSLT processor to build ooRexx. Two of the rules
below come from that: the generated files must be regenerated and committed
with every change to `rexxmsg.xml`, and they must never be edited by hand.

## 1. Edit `rexxmsg.xml`

A new message is a `SubMessage` under its `Message` (error code):

```xml
<SubMessage>
  <Code>25</Code>
  <Subcode>938</Subcode>
  <MessageNumber>843</MessageNumber>
  <Component>Rexx</Component>
  <Severity>Warning</Severity>
  <SymbolicName>Error_Invalid_subkeyword_options_form</SymbolicName>
  <Text>FORM must be followed by ...; found <q><Sub position="1" name="word"/></q>.</Text>
</SubMessage>
```

- `Subcode`: the next free one under that code.
- `MessageNumber`: global and unique. Take the next after the highest in the
  whole file, not just in that code.
- `SymbolicName`: the name the C++ code uses (`syntaxError(Error_..., ...)`).
- `<Sub position="n" name="..."/>`: substitution `&n`. The name appears only
  in the documentation, so choose one that reads well there.

## 2. Regenerate the nine files

CMake regenerates them **into the source tree**, but only when it finds Xalan
(`XALAN_EXECUTABLE`, Xalan-C). If it does not find Xalan, nothing is
regenerated and there is no warning. In that case the build compiles the old
headers and the new symbolic name is undefined.

With Xalan installed (on Debian/Ubuntu: `apt-get install xalan`), a build
does the regeneration for you, or you can run it by hand from the root of
`main/trunk`:

```sh
M=interpreter/messages
Xalan -o $M/RexxErrorCodes.h     $M/rexxmsg.xml $M/RexxErrorCodes.xsl
Xalan -o $M/RexxMessageNumbers.h $M/rexxmsg.xml $M/RexxMessageNumbers.xsl
Xalan -o $M/RexxMessageTable.h   $M/rexxmsg.xml $M/RexxMessageTable.xsl
Xalan -o $M/RexxErrorMessages.h  $M/rexxmsg.xml $M/RexxErrorMessages.xsl
Xalan -o api/oorexxerrors.h      $M/rexxmsg.xml $M/ApiErrorCodes.xsl
Xalan -o $M/errnums.xml          $M/rexxmsg.xml $M/errnums.xsl
Xalan -o $M/errnumsrxqueue.xml   $M/rexxmsg.xml $M/errnumsrxqueue.xsl
Xalan -o $M/errnumssubcom.xml    $M/rexxmsg.xml $M/errnumssubcom.xsl
Xalan -o $M/errnumsrexxc.xml     $M/rexxmsg.xml $M/errnumsrexxc.xsl
```

A useful check before you start: with `rexxmsg.xml` unchanged, regenerating
must leave `svn status` clean. If it does not, the generated files in SVN are
already out of date. Commit that catch-up separately, or say so in your log
message, so your own diff stays yours.

Do not touch the copyright lines of the generated files. They come from the
`.xsl` files.

## 3. Commit in `main/trunk`

Commit `rexxmsg.xml`, every generated file that changed, and the code that
uses the new message, all in **one** commit. A new message changes six of the
nine files: the four headers in `interpreter/messages`, `api/oorexxerrors.h`
and `errnums.xml`. The other three `errnums*.xml` files belong to rxqueue,
rxsubcom and rexxc, and change only when one of their messages does.
`svn status` shows which files changed.

## 4. Copy the documentation in `docs/trunk`

`docs/trunk/rexxref/en-US/errnums.xml`, `errnumsrxqueue.xml`,
`errnumssubcom.xml` and `errnumsrexxc.xml` are copies of the generated files,
identical byte for byte (the reference pulls them in through `errnum.xml`).
Copy the ones that changed over and commit them in `docs/trunk`, citing the
`main/trunk` revision they come from. That is a separate commit, because it
is a separate tree.

## 5. Tests

Changing the code that a condition raises breaks every test that expects the
old one. Grep `test/trunk` for the old code (for example `25.11`) and for the
directive or instruction involved, then update the assertions in the same
test commit.

## In short

1. Edit `rexxmsg.xml`.
2. Regenerate the nine files with Xalan.
3. Commit `rexxmsg.xml`, the changed generated files and the code together in
   `main/trunk`.
4. Copy the changed `errnums*.xml` files to `docs/trunk/rexxref/en-US` and
   commit them there.
5. Update the tests in `test/trunk`.
6. Never edit a generated file by hand.

## The same applies to `PrimitiveClasses.xml`

`interpreter/behaviour/PrimitiveClasses.xml` works the same way. Four files
are generated from it, also into the source tree and also only when Xalan is
found: `ClassTypeCodes.h`, `PrimitiveBehaviourNames.h`,
`PrimitiveBehaviours.cpp` and `VirtualFunctionTable.cpp`. Here, too, a change
made only in the generated file is lost at the next regeneration. Make it in
the corresponding `.xsl` file, then regenerate.
