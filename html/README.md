`<!DOCTYPE html>`
=================

*An html style guide to turn that :&lt; into a :>*

## Document Guidelines

### Doctype

Use the `html5` doctype at all times at the top of your html document:
`<!DOCTYPE html>`.

### `<html>`

Add a `lang` attribute to your `<html>` tag.

### `<head>`

* Always include a title tag.
* Include `<meta>` tags for `keywords` and `description` on search-engine-friendly pages.
* Include a favicon.
* Use IE conditional comments to load IE-specific stylesheets.
* Load as few stylesheets as prudent.
* Don't load javascript files in the head unless absolutely necessary.

### `<body>`

* Optionally include a class on body descriptive of the current page.
* Load javascript at the bottom.
* Use the most specific element possible; `ol` for ordered lists, `a` for links, `button` for buttons.
  Use appropriate html5 structure elements, such as `aside` and `article`.
* Prefer event handlers defined in JS over `on*` attributes

### `<form>`

* Every form input should have a `<label>` tag, *especially radio and checkbox elements*.

## Style

* 2 space indentation.
* Always use quotes for attributes.
* Use "double quotes" for attributes.
* Don't use a closing `/` in self-closing tags, such as `<br>`.
* CSS classes should be lowercase, hyphenated, and descriptive. `.upvote-arrow-active`, not `.upvoted`.

## Template Guidelines

* Use minimal logic in templates.
* Keep html in templates, rather than controllers.
* Make sure all text is i18n'd.
* Prefer `<%-` over `<%=` in Underscore templates to ensure proper escaping.
* HTML templates should only be used to template HTML, using templating inside `on*` event handlers, CSS, JS, or other templates can lead to subtle security issues:
  ```mako
  # unsafe, multiple ways to achieve XSS here
  <div onclick="alert('${untrusted_str}')"></div>

  # unsafe, `untrusted_str` will be unescaped when the template's read from the attr.
  <div data-template="<%- ${untrusted_str} -%>"></div>
  ```

* Any untrusted strings (including strings from gettext) must be escaped when used inside `unsafe()`:

  ```mako
  # bad
  ${unsafe(_("foo %s") % "<b>safe</b>")}

  # good
  # _ws() is shorthand for websafe(_())
  ${unsafe(_ws("foo %s") % "<b>safe</b>")}
  ```

* Arguments to utility functions should not be treated as raw HTML unless the caller specifically requested it, or it's obvious to the caller that the function will do so:

  ```mako
  # bad, easy to make a mistake with escaping
  <%def name="smileify(content)">
    ${unsafe(content.replace(" ", "&#x1f60e;"))}
  </%def>
  ${smileify("<b>safe</b>")}
  ${smileify(websafe("<b>user-controlled</b>"))}

  # good, `mako_websafe` will call `websafe` if not passed an `_Unsafe` instance.
  <%def name="smileify(content)">
    ${unsafe(mako_websafe(content).replace(" ", "&#x1f60e;"))}
  </%def>
  ${smileify(unsafe("<b>safe</b>"))}
  ${smileify("<b>user-controlled</b>")}

  # ok, function makes clear that it's unsafe
  <%def name="unsafe_smileify(content)">
    ${unsafe(content.replace(" ", "&#x1f60e;"))}
  </%def>
  ${unsafe_smileify("<b>safe</b>")}
  ${unsafe_smileify(websafe("<b>user-controlled</b>"))}
  ```

* For legacy reasons, Mako won't automatically escape single quotes. This shouldn't be abused, since it can lead to subtle issues. Use `tags()` for inline templating of attributes:

  ```mako
  # bad, `untrusted_str` can escape `alt`
  <a ${("alt='%s' " % untrusted_str) if untrusted_str else ""} ...

  # good
  <a ${tags(alt=untrusted_str)} ...
  ```
