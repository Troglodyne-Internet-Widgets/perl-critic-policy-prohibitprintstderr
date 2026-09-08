# NAME

Perl::Critic::Policy::ProhibitPrintSTDERR - Print to STDERR throws away the one thing a diagnostic is for.

# VERSION

version 1.000

# Perl::Critic::Policy::ProhibitPrintSTDERR

`warn` and `print STDERR` put the same text on the same handle.  They differ
in what else they carry:

```
print STDERR "could not read $file\n";   # that, and nothing more
warn "could not read $file";             # ... at Foo.pm line 42.
```

`warn` appends the file and line for free, goes through `$SIG{__WARN__}` so a
program can route or count its own diagnostics, and can be promoted to fatal.
A print is text on a filehandle: whoever reads the log gets the sentence and no
way back to the line that produced it.  The two are the same amount of typing
and only one of them is still useful at three in the morning.

So this policy exists to make the print the deliberate choice rather than the
default one -- because the print is what you reach for when you are thinking
about the message, and `warn` is what you want when somebody is thinking about
the failure.

## PROHIBITED

Every spelling of the same thing, since a policy that only knew one would just
be an argument for using another:

```
print STDERR "oh no\n";
printf STDERR "%s\n", $why;
say STDERR 'oh no';
print {*STDERR} "oh no\n";
print {\*STDERR} "oh no\n";
print *STDERR "oh no\n";
print(STDERR "oh no\n");
STDERR->print("oh no\n");
```

## ALLOWED

Anything not aimed at STDERR:

```
print "ordinary output\n";
print STDOUT "ordinary output\n";
print {$fh} "into a file\n";
warn "the thing this policy is asking for";
```

### Usage and progress text

There is a real exception, and it is not diagnostics.  A script writing its
`--help`, or narrating its progress so the transcript is readable, is using
STDERR as a channel to a person rather than reporting a failure -- and `warn`
would staple a source location onto every line of it.

That is what the standard signoff is for, on the statement:

```
print STDERR "Restarted $domain\n";    ## no critic (ProhibitPrintSTDERR)
```

or over a run of them, which is usually what usage text is:

```perl
## no critic (ProhibitPrintSTDERR)
print STDERR "usage: $0 [--verbose] DOMAIN\n";
print STDERR "  --verbose   say what is happening\n";
## use critic
```

Both leave a mark saying somebody decided, which is the whole point.  The
question the reviewer is then asking is not "why is this not warn" but "is this
really talking to a person", and that one can be answered by reading it.

## CONFIGURATION

This policy is not configurable except for the standard options.

Deliberately: an exemption for "usage subs" or a list of blessed filenames
would be a second mechanism for something `## no critic` already does, and it
would exempt the file rather than the line -- so the next diagnostic added to a
script full of usage text would inherit the exemption without anybody
deciding.

## CAVEATS

The handle has to be visible in the source.  A print through a copy of it --

```perl
my $err = \*STDERR;
print {$err} "invisible to this policy\n";
```

\-- reads as an ordinary filehandle here, because it is one by the time it gets
to the print.  The policy reads source, not data flow.

Likewise `STDERR->say(...)` and the rest of [IO::Handle](https://metacpan.org/pod/IO%3A%3AHandle) beyond `print`:
only the `print` method is recognised, since that is the one anybody writes.

## METHODS

### supported\_parameters

None.

### default\_severity

SEVERITY\_MEDIUM

### default\_themes

maintenance, bugs

### applies\_to

PPI::Token::Word

### violates

Standard [Perl::Critic::Policy](https://metacpan.org/pod/Perl%3A%3ACritic%3A%3APolicy) interface.  Returns a violation for a `print`,
`printf` or `say` whose filehandle is STDERR, however it is spelled, and
nothing for one aimed anywhere else.

# AUTHORS

Current Maintainers:

- George S. Baugh <george@troglodyne.net>

# COPYRIGHT AND LICENSE

Copyright (c) 2026 Troglodyne LLC

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:
The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.
THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
