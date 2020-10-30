<div align="center"><img src="https://res.cloudinary.com/adonis-js/image/upload/q_100/v1600679850/edge-banner_wao6ex.png" width="600px"></div>

# Edge syntax

This repo is a living document which outlines the **whitelisted** syntax for Edge template engine. 

## Fundamentals

Edge attempts to optimize for following.

1. Keep syntax closer to Javascript
2. Easier to type
3. Unified
4. Work with any markup language
5. Respect newlines & whitespace

### Keep syntax closer to Javascript
We do not want you to learn a new syntax when working with Edge. Almost every Javascript expression is supported by Edge and works as if you are writing Javascript.

This approach has handful of benefits.

1. Small learning curve for Edge users.
2. Easier for us to build syntax highlighters code editors, since we can use in-built Javascript engine to parse parts of your template.


By looking at the following code from **Edge** and **Nunjucks**, you can see that Edge reads like Javascript and **Nunjucks** reads like Python.

**nunjucks**

```
{{ "true" if foo else "false" }}
```

**edge**

```
{{ foo ? 'true' : 'false' }}
```


### Easier to type

It is relative easier to type Edge primitives over many other template engines. Look at the following comparisons.

**nunjucks**

```
{% if variable %}
  It is true
{% endif %}
```

**handlebars**

```
{{#if variable}}
  It is true
{{/if}}
```

**edge**

```
@if(variable)
@endif
```

Or even shorter 

```
@if(variable)
@end
```

### Unified

There is no custom vocabulary in Edge for every different thing. Everything revolves around the concept of `Tags`. Logical entities like `@if`, `@else` and tags like `@include`, `@component` all are tags.

### Work with any markup language

Edge itself is not tied to HTML like [MarkoJs](https://markojs.com/). You can write any markup language inside Edge and it will output it consistently by preserving the whitespaces and newlines.


### Respect newlines & whitespace

Since Edge can be used with any markup language, we should be sensitive with the whitespace and the newlines. One should be able to use Edge to generate a YAML file with proper indentation and new lines.

--- 

## Whitelisted syntax

Under the hood, there are 3 entities in Edge as follows:

1. Tags
2. Mustache
3. Comments

There are different ways to write **Tags** and **Mustache** blocks and this guide covers all of them.



### Tags (block)

1. Every block level tag starts with `@` symbol followed by the tagName.
2. It is important to close a block level using `@end<tagName>`
3. `@` and `tagName` cannot have spaces between them.

**VALID**

```
@if(username)
@endif
```

**VALID**

```
@if(
  username
)
@endif
```

**VALID**

```
@if(
  (
    2 + 2
  )
  ===
  4
)
```

**INVALID**
The opening brace must be in it's own line.

```
@if
(
 username
)
```

**INVALID**

The opening of the tag must be in it's own line and so do the closing one.

```
@if(username) Hello @endif
```

**INVALID**

```
@if(
  username
) <p> Hello </p>
@endif
```

### Tags (inline)
The inline tags doesn't contain any children and hence requires no `@end` statement is required.

**VALID**

```
@include('header')
```

**VALID**

```
@include(
  'header'
)
```

**INVALID**
The opening brace must be in its own line.

```
@include
(
  'header'
)
```


### Tags (block-self closed)

At times block level tags can work fine without any body inside them. To keep the syntax concise, you can **self-close** a block level tag.

**NORMAL**

```
@component('button', text = 'Submit')
@endcomponent
```

**SELF CLOSED**

```
@!component('button', text = 'Submit')
```


### Mustache
The mustache braces `{{` are used to define inline Javascript expressions. The lexer allows:

1. Multiline expressions.
2. A valid Javascript expression, that yields to a value.
3. The HTML output from mustache will be escaped, so make sure to use `{{{ '<p>' Hello world </p> }}}` for rendering HTML nodes.

**VALID**

```
{{ username }}
```

**VALID**

```
{{
 username
}}
```

**VALID**

```
Your friends are {{
  users.map((user) => {
    return user.username
  }).join(',')
}}
```

**VALID**

```
{{{ '<p>' Hello world </p> }}}
```

**INVALID**

The starting curly brace, must be in one line.

```
{
{
  username
}
}
```

## Escaping

The backslash `\` has a special meaning in Javascript, that's why we make use of `@` to escape Edge statements.

> There is no need to escape the `@end` statements, since they are tightly mapped with the start statements. So if a start statement doesn't exists, the end statement will be considered as a raw node.

**ESCAPED**

```
@@if(username)
@endif
```

In the same fashion, the mustache braces can be escaped using `@`.

**ESCAPED**

```
Hello @{{ username }}
```

## Comments

The comments in Edge disappears before the template is rendered.

Comment starts with a **double curly brace** and then immediately followed by two dashes `--`

```
{{-- This is a comment --}}
``` 

Comments can also appear inline with other text. For example:

```
Hello world {{-- Greeting --}}
```

```
{{-- Greet user --}} Hello {{ username }}
```

**MULTILINE COMMENT**

```
{{--
 This is a multiline comment
--}}
```

**INVALID**

The `dashes (--)` must in the same line as the opening and the closing curly braces.

```
{{
--
 This is a multiline comment
--
}}

```

## End using shorthand `@end`

The `@end` keyword can be used in place of using `@end<tagName>`. For example:

```
@if(username)
@end

@component('button')
  <p> Foo </p>
@end
```

## Swallow new lines

Since tags are written in their own line, they can create unnecessary some newlines. Based upon the desired outcome, you can remove the newlines after a tag by using the tilde `~` expression.

### WITHOUT TILDE

**TEMPLATE**

```
<p> Hello
@if(username)
 {{ username }}
@endif
</p>
```

**OUTPUT**

```
<p> Hello
 virk
</p>
```


### WITH TILDE

**TEMPLATE**

```
<p>Hello
@if(username)~
 {{ username }}
@endif~
</p>
```

**OUTPUT**

```
<p>Hello virk</p>
```
