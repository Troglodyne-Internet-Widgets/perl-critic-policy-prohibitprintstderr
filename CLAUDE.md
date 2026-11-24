# CLAUDE.md

This file is the procedure for work in this repository.

The repository holds one Perl::Critic policy, `ProhibitPrintSTDERR`. The
policy is in `lib/Perl/Critic/Policy/ProhibitPrintSTDERR.pm`. Its tests are in
`t/main.t`. `README.md` says what the policy does and why.

## Which skills, and when

The hooks of the perl-slop plugin hold you to the procedure. They refuse an
edit to Perl until `perl-slop:reading-perl` is loaded. They refuse a commit of
Perl until `data-perl`, `testing-perl` and `reviewing-perl` are loaded, each
one after the last commit. A refusal names the skills that are missing.

The hooks cannot see one case. Before you answer a question about the code,
load `perl-slop:reading-perl`. Load it for that as well as before an edit.

## Read the code before you change it

Most of the code here is older than the conversation about it. A line that
looks pointless is usually the scar of something that went wrong once. The
reason is in the commit, not in the file. So read the history before the first
edit, not after the tests fail.

## Finishing a changeset

Apply the three skills of the commit gate in this order: data-perl,
testing-perl, then reviewing-perl against the whole diff. To load a skill is
not to apply it.

Then run `podchecker` on each changed file.

Do not run `perltidy`, `perlcritic` or `perl -c` yourself. The pre-commit hook
tidies the Perl that you staged, runs perlcritic on it, and compiles it. Then
it runs the tests that the commit can break. If a step fails, the hook stops
the commit and prints the reason.

The hook names each test that failed, but not the reason. If a test fails, run
that file with `-v` and read the output:

    prove -lv t/main.t

The hub that runs the agents installs the hooks. Do not install them again.
The post-commit, post-merge and post-rewrite hooks run `refresh-coverage`. It
updates the records of `tests-covering` in the background. A path that
`.tests-covering-map.pl` cannot place runs every test.

## Commits and pull requests

Make a branch. Do not commit to `master`.

A commit message says what was wrong and why the change fixes it. Write it in
prose and in the imperative mood. Name the behavior, not the diff. The why goes
in the message and not in a comment, because a person who runs `git blame` two
years from now finds the message.

After you make sure that something works, say what you ran and what it
printed. Quote that output with the site-specific names taken out, for example
`$domain` and `$host`. This repository is public.
`perl-slop:information-security` has the full rule.

Add each user-visible change to `Changes`. Open every pull request against
`master`, and use a closing keyword such as `Closes #N` for the issue that it
fixes.
