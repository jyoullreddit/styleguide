# Reddit General Style Guidelines

Check out the [reddit/reddit wiki](https://github.com/reddit/reddit/wiki) for
additional resources.

## The Golden Rule

Follow the style of the file you're in. It's more important to be *contextually*
correct than it is to be *absolutely* correct.

## "Reddit"

Reddit should be uppercase.

## Git workflow

### Commits

Individual commits should be atomic changes to the codebase. Specifically:

* A commit should not leave the code in a broken state waiting on a later
  commit to fix it.
* Changes that aren't related should be split into separate commits. This
  must be taken with the previous rule in mind to help determine what's
  related or not.  Bugfixes that are prerequisites to your change but not
  the main point of the change should be made in separate commits that come
  first.


### Commit Messages

Follow the [git commit message standard](http://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html):

```
Capitalized, short (50 chars or less) summary

More detailed explanatory text, if necessary.  Wrap it to about 72
characters or so.  In some contexts, the first line is treated as the
subject of an email and the rest of the text as the body.  The blank
line separating the summary from the body is critical (unless you omit
the body entirely); tools like rebase can get confused if you run the
two together.

Write your commit message in the imperative: "Fix bug" and not "Fixed bug"
or "Fixes bug."  This convention matches up with commit messages generated
by commands like git merge and git revert.

Further paragraphs come after blank lines.

- Bullet points are okay, too

- Typically a hyphen or asterisk is used for the bullet, preceded by a
  single space, with blank lines in between, but conventions vary here

- Use a hanging indent


```

A more in-depth explanation of good commit messages is written up here:
<https://chris.beams.io/posts/git-commit>.

Prefixing the commit message summary with the subsystem being affected
can be helpful sometimes, e.g. `/dev/api: Document subreddit endpoints.`.

It's a good idea to include the rationale for a change in the explanatory
text for non-trivial changes:

> This is why commit messages are important - they explain *why* the
code was changed. Hence the code review process is not just about
the code - the reviewers need to understand why the code
is being changed and the process that led to the
change being proposed. The commit history is going to be the only
record of why this code exists in 10 years time, so the explanation
needs to be correct.

More info: <https://lwn.net/Articles/637896/>.

### Make sure changes are rebased on latest master.

Ensure your code is up-to-date to prevent merge conflicts and make testing
easier.

### gitignore

In general, a project's gitignore should ignore files that are related to that
project. For example, a Python project might put `*.pyc` in its gitignore.

Things that are specific to your development environment (like vim's `*.swo`,
or IDE-specific directories, or `.DS_Store` on OS X) should be added to your
[personal global gitignore]. This makes it so all your projects ignore that
file and projects don't need to have an explosion of every different
environment's ignores.

[personal global gitignore]: https://help.github.com/articles/ignoring-files/#create-a-global-gitignore
