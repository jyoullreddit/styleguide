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

### templates

[Don't use lookup functions in templates](https://puppet.com/docs/hiera/3.3/puppet.html#dont-use-the-lookup-functions-from-templates)
if it can be avoided. Variables in templates should be sourced from the
manifest calling the template.

Example:

init.pp
```puppet
$host = hiera('harold::host')
file { '/etc/foo':
  contents => template('myclass/template.erb'),
}
```

template.erb
```erb
Good:
Host=<%= @host %>

Bad:
Host=<%= scope.function_hiera(['harold::host'])
```


## puppetlint

The following puppet-lint command should validate our rules:

    puppet-lint --no-double_quoted_strings-check --no-single_quote_string_with_variables-check FILES
