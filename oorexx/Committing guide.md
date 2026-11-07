# Committing to ooRexx: a guide for first-time committers

This guide is for someone who knows ooRexx and their operating system but has
never used Subversion (SVN), diffs or patches. It goes from creating a
SourceForge account to making a commit, checking it, and fixing it when
something goes wrong. The examples are for Windows (`cmd`); on Linux and
macOS the `svn` commands are the same, with `/` instead of `\` in paths.

## 1. The ideas you need

- **Repository.** The single, central copy of the ooRexx sources, with all
  their history. It lives at SourceForge. ooRexx's is called `code-0`.
- **Revision.** Every commit to the repository gets the next number: r13409,
  r13410, ... One number covers the whole repository, whatever part the
  commit touched. "HEAD" means the latest revision.
- **Working copy.** A folder on your computer with a checked-out copy of part
  of the repository, plus a hidden `.svn` folder that remembers which
  revision it came from. You edit files there.
- **Update** (`svn update`). Bring your working copy up to HEAD.
- **Commit** (`svn commit`). Send your changes to the repository. This
  creates a new revision at once, for everybody. Unlike Git, SVN has no local
  commits and no separate "push": **a commit is public the moment it is
  made**.
- **Diff.** A text listing of what changed between two versions of a file:
  lines starting with `-` were removed and lines starting with `+` were
  added, with a few unchanged lines around them for context. A **patch** is a
  diff saved to a file, so that someone else can apply the same change.

### How the ooRexx repository is laid out

```
code-0/
  main/trunk/    the interpreter, its libraries and extensions (C++, Rexx, CMake)
  test/trunk/    the test suite (testOORexx.rex, ooRexx/base/..., *.testGroup)
  docs/trunk/    the documentation (DocBook XML: rexxref, rexxpg, ...)
  main/branches/, main/releases/, ...   older lines and released versions
```

`main`, `test` and `docs` are separate trees. A change that touches the code
and its test is **two commits**, one in each tree. Do not check out the whole
repository: it holds every release ever made and is very large. Check out
only the three trunks.

## 2. Get a SourceForge account

Create an account at <https://sourceforge.net> (Join). Your **SourceForge
username** is what you will use everywhere below. It is not your email
address.

## 3. Create an SSH key and register it with SourceForge

Commits go over SSH. Instead of a password, you prove who you are with an
**SSH key pair**: a *private key*, which never leaves your computer, and a
*public key*, which you give to SourceForge.

Windows 10 and 11 come with OpenSSH. In `cmd`:

```
ssh-keygen -t ed25519 -C "USERNAME for sourceforge"
```

Accept the proposed file (`C:\Users\<you>\.ssh\id_ed25519`) and choose a
**passphrase**. The passphrase protects the private key if someone copies the
file. You will be asked for it at every commit, unless you use the agent (see
below). This creates two files:

- `id_ed25519`: the private key. Never send it or upload it anywhere.
- `id_ed25519.pub`: the public key, a single line of text starting with
  `ssh-ed25519`.

Register the public key with SourceForge:

1. Log in and go to your account's SSH settings,
   <https://sourceforge.net/auth/shell_services> (Account Settings, SSH
   Settings).
2. Open `id_ed25519.pub` in a text editor, copy the whole line, paste it into
   the form and click **Update**.

SourceForge recommends ED25519 or ECDSA keys. DSA keys are not recommended.

*Optional: stop typing the passphrase.* The OpenSSH agent keeps the
unlocked key in memory. In PowerShell, as administrator:
`Get-Service ssh-agent | Set-Service -StartupType Automatic; Start-Service
ssh-agent`. Then, as yourself, run `ssh-add` once and type the passphrase.
This has obvious security risks and is not recommended: anyone who can use
your session can commit as you. Never do it on a computer that someone else
can use.

## 4. Install TortoiseSVN, with the command-line tools

TortoiseSVN (<https://tortoisesvn.net>) adds SVN commands to the Explorer
right-click menu. It can also install the command-line client, `svn.exe`,
which this guide uses, because its output can be read, copied and pasted into
a message when you need help.

1. Run the installer. On the page where you choose features, **select
   "command line client tools"**. It is not installed by default.
2. Open a new `cmd` window and check: `svn --version`.

*Optional.* The repository has a client configuration file,
<https://svn.code.sf.net/p/oorexx/code-0/svnconfig/config>, whose
`[auto-props]` section makes `svn add` set some properties by itself. You do
not need it: section 9 shows how to set them by hand.

### Make SVN use your OpenSSH key

`svn.exe` runs the `ssh` command it finds on the PATH, which on Windows is
OpenSSH. Nothing needs configuring: it will use `id_ed25519` and ask for its
passphrase.

For TortoiseSVN's own right-click commands, go to Settings, Network, SSH
client, and enter `C:\Windows\System32\OpenSSH\ssh.exe`. TortoiseSVN's
default client, TortoisePlink, cannot read OpenSSH keys. If you prefer
PuTTY's tools instead (PuTTYgen, Pageant), SourceForge documents that route
in "Using SSH Keys with SVN on Windows".

### Check that the connection works

```
svn ls svn+ssh://USERNAME@svn.code.sf.net/p/oorexx/code-0/
```

The first time, SSH shows the server's fingerprint and asks whether to trust
it. You can compare it with SourceForge's "SSH Key Fingerprints" page. Type
`yes`. If you then see a list of folders (`docs/`, `main/`, `test/`, ...),
everything works.

## 5. Check out the three trunks

Choose a folder, for example `C:\oorexx`, and from it:

```
svn checkout svn+ssh://USERNAME@svn.code.sf.net/p/oorexx/code-0/main/trunk main\trunk
svn checkout svn+ssh://USERNAME@svn.code.sf.net/p/oorexx/code-0/test/trunk test\trunk
svn checkout svn+ssh://USERNAME@svn.code.sf.net/p/oorexx/code-0/docs/trunk docs\trunk
```

(`svn co` is the short form.) Each command prints the files as it downloads
them and ends with `Checked out revision NNNNN.`

If you already have a read-only checkout made with an `https://` URL, there
is no need to download again. From inside it, switch it to SSH:

```
svn relocate svn+ssh://USERNAME@svn.code.sf.net/p/oorexx/code-0/main/trunk
```

## 6. The commands you will use every day

All of them are run from inside a working copy, usually from its root
(`main\trunk`, `test\trunk` or `docs\trunk`).

| Command | What it does |
|---|---|
| `svn update` (`svn up`) | Bring the working copy up to HEAD. Always do this before you start working, and again before you commit. |
| `svn status` (`svn st`) | List what you changed. Fast, and needs no network. Run it before every commit. |
| `svn diff` | Show your changes as a diff. `svn diff file` shows the changes to one file. |
| `svn diff > change.patch` | Save your changes as a patch. |
| `svn log -l 5` | The last 5 commits that touched this folder. |
| `svn log -v -r 13409` | One commit: author, date, message, and (`-v`) the files it changed. |
| `svn info` | URL, current revision and other details of the working copy. |
| `svn revert file` | Throw away your changes to a file. **No undo.** |
| `svn add file` | Put a new file under version control. Nothing is sent until you commit. |
| `svn delete file` | Remove a file. It disappears from the repository at the next commit. |
| `svn move old new` | Rename or move a file, keeping its history. |
| `svn cleanup` | Repair a working copy left locked by an interrupted command. |
| `svn help command` | Help on any command. |

### Reading `svn status`

```
M       interpreter\execution\RexxActivation.cpp
A       ooRexx\base\bif\SETLOCAL.testGroup
?       msg-main.txt
```

| Letter | Meaning |
|---|---|
| `M` | Modified by you. |
| `A` | Added (`svn add`); goes in at the next commit. |
| `D` | Deleted (`svn delete`). |
| `?` | Not under version control: SVN ignores it unless you `svn add` it. Build output and your own notes appear here. That is fine. |
| `!` | Under version control but missing from disk (deleted without `svn delete`). |
| `C` | Conflict (section 10). |
| `L` | Locked by an interrupted command: run `svn cleanup`. |

A letter in the **second column** (`␠M`) means that a *property* changed,
not the content.

## 7. Making a commit, step by step

1. **Update first:** `svn up`. You will be working on the current sources,
   and `svn ci` will not complain later that a file is out of date.
2. **Make your change**, build, and run the test suite (with and without
   your change, and compare the lists of failing tests). In ooRexx, a fix
   comes with a test.
3. **See what you are about to send:**

   ```
   svn status
   svn diff
   ```

   The only `M`/`A`/`D` lines should be the files you meant to change, and
   the diff only the lines you meant to change. Look for stray debugging
   lines, accidental edits and changes to line endings. A diff in which
   every line of a file appears as removed and added again means that the
   line endings changed.
4. **Write the log message to a file**, for example `msg.txt`. It is easier
   than typing it on the command line, and you can check it before
   committing. The format used in ooRexx:

   ```
   [bugs:#2146] SETLOCAL/ENDLOCAL crashed the interpreter on Unix

   One or more lines explaining what was wrong and what changes, in
   English. Who reported it, if someone else did.
   ```

   `[bugs:#NNNN]` at the start links the commit to ticket NNNN in
   SourceForge (`[feature-requests:#NNN]` for feature requests). Open the
   ticket **first** and put its real number in the message. A placeholder
   such as `#NNNN` ends up in the log if you forget to change it.
5. **Commit, naming the files:**

   ```
   svn ci -F msg.txt interpreter\execution\RexxActivation.cpp interpreter\platform\unix\ExternalFunctions.cpp
   ```

   Naming them guarantees that nothing else goes in, such as `msg.txt`
   itself, a file you forgot you had touched, or a half-finished change in
   another file. `svn ci` with no file names commits **every** modified file
   under the current folder.

   SVN asks for your key's passphrase (unless the agent has it), and ends
   with:

   ```
   Committed revision 13409.
   ```

6. **Check the commit:**

   ```
   svn log -v -r 13409
   ```

   Check that the message and the list of files are right. Every commit is
   also mailed to the `oorexx-svn` list, diff included.
7. **For a change in two trees** (code in `main`, its test in `test`), repeat
   steps 3 to 6 in the other working copy. Then close the ticket with a
   comment such as `Committed in [r13409] (code) and [r13410] (test).`
   `[rNNNNN]` becomes a link in SourceForge.

**Do one step at a time, and read the output of each command before you run
the next.** Most mistakes are cheap to fix if you catch them right after the
command that made them.

## 8. Patches

A patch is how you show a change to someone, or receive one, without
committing it.

- **Make one:** `svn diff > fix.patch`, or `svn diff file1 file2 >
  fix.patch` for some files only. Attach it to the ticket.
- **Apply one** that someone gave you: from the root of the same working
  copy (for example `main\trunk`), `svn patch fix.patch`. Then use `svn st`
  and `svn diff` to look at what it did, and test before committing. In
  TortoiseSVN: right-click the folder, TortoiseSVN, Apply patch.
- **Read one:** `--- file` / `+++ file` name the file; `@@ -120,7 +120,9 @@`
  says the change is near line 120 (7 lines before, 9 after); then `-`
  lines go, `+` lines come, and the rest is context.

A patch made against an older revision may not apply cleanly to HEAD. When
that happens, `svn patch` says which parts ("hunks") it could not apply.

## 9. File properties

SVN stores some properties with each file. Three of them matter in ooRexx:

| Property | Value | For |
|---|---|---|
| `svn:eol-style` | `native` | Every text file: sources, headers, `.rex`, `.cls`, `.testGroup`, `.xml`, `CMakeLists.txt`... The file gets Windows line endings in a Windows working copy and Unix ones in a Unix working copy. Without it, a file edited on Windows can end up with CRLF line endings, and diffs become a mess. |
| `svn:executable` | (any value) | Scripts run directly on Unix: `*.rex`, `*.sh`, and **`*.testGroup`** (the test suite runs them). |
| `svn:mime-type` | e.g. `image/png` | Binary files, so that SVN never tries to diff them or change their line endings. |

Set them when you add a file, before committing it. The full sequence for a
new test group:

```
svn add ooRexx\base\bif\SETLOCAL.testGroup
svn propset svn:eol-style native ooRexx\base\bif\SETLOCAL.testGroup
svn propset svn:executable ON ooRexx\base\bif\SETLOCAL.testGroup
svn status
```

For a new source file, header or document, only the `svn:eol-style` line is
needed. For a new `.rex` or `.sh` script, both.

> **Never write `svn propset svn:executable "*" file` on Windows.** It is
> what the SVN book says, and on Unix it is fine, but on Windows `svn.exe`
> expands the `*` itself, even inside quotes, and sets the property on
> **every file in the folder**. Write `ON` instead: SVN stores `*` whatever
> value you give it. If it happens, `svn status` shows a `␠M` for every file
> in the folder. Undo it with `svn revert` on those files before committing,
> or with `svn propdel svn:executable file` if they were already committed.

Useful commands:

```
svn proplist -v file          list a file's properties and their values
svn propdel svn:executable file
```

The properties of a new file go in with the commit that adds it. Check them
afterwards with `svn proplist -v` on the repository URL of the file.

## 10. When things go wrong

| What happened | What to do |
|---|---|
| You changed a file by mistake, before committing | `svn revert file` (or `svn revert -R .` for everything under the folder: careful). |
| You added a file by mistake, before committing | `svn revert file`. The file stays on disk, no longer added. |
| `svn ci` says **"File ... is out of date"** | Someone committed that file after your last update. `svn up`, check with `svn diff` that your change is still what you meant, and commit again. |
| `svn up` shows **`C`** (conflict) | Someone changed the same lines as you. SVN writes both versions into the file between `<<<<<<<`, `=======` and `>>>>>>>` markers, plus copies `.mine`, `.rNNNN`. Edit the file until it is right, remove the markers, then `svn resolve --accept working file`. |
| **"Working copy locked"**, or a command was interrupted | `svn cleanup`, then `svn status`. |
| The commit was interrupted (timeout, network) and you do not know whether it went in | Do not commit again blindly. Look at the repository: `svn log -l 3 -v` with the repository URL, or check the `oorexx-svn` list. If the revision is there, the commit went in. Then run `svn cleanup` and `svn up` in your working copy, and check that `svn status` is clean. |
| **Wrong log message** | It can be changed after the commit. Write the right message to `msg.txt`, then: `svn propset svn:log --revprop -r 13409 -F msg.txt .` This works in ooRexx's repository. |
| You committed a file you did not mean to, or a wrong change | Commit the fix as a new revision. To undo a whole commit: `svn merge -c -13409 .` (note the minus sign: it applies the commit in reverse), check with `svn diff`, and commit with a message that says what it reverts and why. History is never rewritten: the mistaken revision stays, followed by the one that undoes it. |
| You forgot a file | Commit it separately, with the same `[bugs:#NNNN]` and a line saying it belongs to the previous revision. |
| Wrong properties on a committed file | `svn propset` or `svn propdel`, then commit (`svn status` shows `␠M`). |

Two habits avoid most of these: `svn status` and `svn diff` before every
commit, and naming the files on `svn ci`.

## 11. TortoiseSVN equivalents

Everything above can also be done from Explorer: right-click a working-copy
folder or file, then TortoiseSVN. SVN Update, SVN Commit, Check for
modifications (= `svn status`), Diff, Show log, Add, Revert, Properties,
Apply patch, Clean up. The commit dialog lists the modified files with check
boxes. Untick everything you do not mean to commit, and double-click a file
to see its diff before you press OK. The command line and TortoiseSVN share
the same working copy and the same configuration, so you can mix them.

## 12. The ooRexx habits, in short

1. Open or find the ticket first; put `[bugs:#NNNN]` at the start of the log
   message.
2. `svn up`, change, build, run the test suite before and after, add a test.
3. `svn status` and `svn diff` before every commit.
4. Write the log message to a file and use `svn ci -F msg.txt` with the file
   names.
5. One commit per tree: code in `main/trunk`, tests in `test/trunk`,
   documentation in `docs/trunk`.
6. New files: `svn add`, `svn:eol-style native`, and `svn:executable ON` for
   scripts and test groups (never `"*"`).
7. After committing, `svn log -v -r N`; then close the ticket citing the
   revisions.
8. Changing an error message has its own procedure: see "Adding or changing
   an error message".

## References

- SourceForge, "Subversion": <https://sourceforge.net/p/forge/documentation/svn>
- SourceForge, "SSH Keys": <https://sourceforge.net/p/forge/documentation/SSH%20Keys>
- SourceForge, "Using SSH Keys with SVN on Windows":
  <https://sourceforge.net/p/forge/documentation/Using%20SSH%20Keys%20with%20SVN%20on%20Windows>
- The Subversion book (free): <https://svnbook.red-bean.com>
- TortoiseSVN: <https://tortoisesvn.net>
