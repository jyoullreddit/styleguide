# reddit Puppet Style Guidelines

## style guide

We follow the standard puppet style guidelines

* <https://docs.puppetlabs.com/guides/style_guide.html>

with only minor modifications

### cron

For cron resources, both the `hour` and `minute` variables must always be
explicitly stated.  If you intend to unrestrict for that interval, use
`absent`.

The goal of this is to reduce the incidence of cron resources accidentally
getting `*` for hour / minute intervals unintentionally.


## puppetlint

The following puppet-lint command should validate our rules:

    puppet-lint --no-double_quoted_strings-check --no-single_quote_string_with_variables-check FILES
