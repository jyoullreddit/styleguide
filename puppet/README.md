# reddit Puppet Style Guidelines

## style guide

We follow the standard puppet style guidelines

* <https://docs.puppetlabs.com/guides/style_guide.html>

with only minor modifications

### Quoting

Instead of

> All strings must be enclosed in single quotes, unless they contain variables or single quotes.

just always use double quotes (`"`)


## puppetlint

The following puppet-lint command should validate our rules:

    puppet-lint --no-double_quoted_strings-check --no-single_quote_string_with_variables-check FILES
