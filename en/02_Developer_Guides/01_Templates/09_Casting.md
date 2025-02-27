---
title: Formatting, Casting, and Escaping Variable Content
summary: Information on casting, security, modifying data before it's displayed to the user and how to format data within the template.
icon: code
---

# Formatting, casting, and escaping variable content

All objects that are being rendered in a template should be a [ViewableData](api:SilverStripe\View\ViewableData) instance such as `DataObject`,
`DBField` or `Controller`. From these objects, the template can include any method from the object in [scope](syntax#scope).

## Casting

The templating system casts variables and the result of method calls into one of the various [`DBField`](api:SilverStripe\ORM\FieldType\DBField)
classes before outputting them in the final rendered markup. Those `DBField` classes provide methods that developers can use to format data in
the template (e.g. choosing how to format a date), as well as defining whether HTML markup in that data will be escaped, stripped, or included
directly in the rendered result.

Methods which return data to the template should either return an explicit object instance describing the type of
content that method sends back, or provide a type (using the same format as the `$db` array described in
[Data types and casting](/developer_guides/model/data_types_and_casting/)) in the `$casting` array for the object. When rendering that method
to a template, Silverstripe CMS will ensure that the object is wrapped in the correct type. This provides you with all of the methods available in
that class, and ensures and values are safely escaped.

```php
namespace App\Data;

use SilverStripe\View\ViewableData;

class MyTemplatedObject extends ViewableData
{
    private static $casting = [
        'Header' => 'HTMLText',
    ];

    public function getHeader()
    {
        return '<h1>This is my header</h1>';
    }
}
```

When calling `$MyCustomMethod` on the above object from a template, Silverstripe CMS now has the context that this method contains HTML, so it
won't escape the value.

For every field used in templates, a casting helper will be applied. This will first check for any
`casting` helper on your model specific to that field, and will fall back to the `default_cast` config
in case none are specified.

[note]
By default, all content without a type explicitly defined in a `$casting` array will use the `ViewableData.default_cast` configuration. By default,
that configuration is set to `Text`, so HTML characters are escaped.
[/note]

### Common casting types

Values can be cast as any `DBField` instance, but these tend to be the most common:

- `Text` Which is a plain text string, and will be safely encoded via [`htmlspecialchars()`](https://www.php.net/manual/en/function.htmlspecialchars.php) when placed into
 a template.
- `Varchar` which is the same as `Text` but for single-line text that should not have line breaks.
- `HTMLFragment` is a block of raw HTML, which should not be escaped. Take care to sanitise any HTML
 value saved into the database.
- `HTMLText` is a `HTMLFragment`, but has shortcodes enabled. This should only be used for content
 that is modified via a TinyMCE editor, which will insert shortcodes.
- `Int` for integers.
- `Decimal` for floating point values.
- `Boolean` For boolean values.
- `Datetime` for date and time.

## Formatting

As has been mentioned in a few sections of documentation already, in a template you have access to all of the public properties and methods of any
object in scope. For instance, if we provide a [`DBHtmlText`](api:SilverStripe\ORM\FieldType\DBHtmlText) instance to the template (either directly
by returning it from a method, or by declaring it in the `$db` configuration for a `DataObject` model, or by declaring it in the `$casting` configuration),
we can use `$MyField.FirstParagraph` in the template. This will
output the result of the [`DBHtmlText::FirstParagraph()`](api:SilverStripe\ORM\FieldType\DBHtmlText::FirstParagraph()) method to the template.

```ss
<%-- app/src/Page.ss --%>

<%-- prints the result of DBHtmlText::FirstParagragh() --%>
$Content.FirstParagraph

<%-- prints the result of DBDatetime::Format("d/m/Y") --%>
$LastEdited.Format("d/m/Y")
```

Any public method from the object in scope can be called within the template. If that method returns another
`ViewableData` instance, you can chain the method calls.

```ss
<%-- prints the first paragraph of content for the first item in the list --%>
$MyList.First.Content.FirstParagraph

<%-- prints "Copyright 2023" --%>
<p>Copyright {$Now.Year}</p>

<%-- prints <div class="about-us"> --%>
<div class="$URLSegment.LowerCase">
```

### Commonly useful formatting methods

All `DBField` instances share the following useful methods for formatting their values:

- [`RAW()`](api:SilverStripe\ORM\FieldType\DBField::RAW()) - outputs the *raw* value into the template with no escaping.
- [`XML()`](api:SilverStripe\ORM\FieldType\DBField::XML()) (and its alias [`HTML()`](api:SilverStripe\ORM\FieldType\DBField::HTML())) - encodes the value (using [`htmlspecialchars()`](https://www.php.net/manual/en/function.htmlspecialchars.php)) before outputting it.
- [`CDATA`](api:SilverStripe\ORM\FieldType\DBField::CDATA()) - formats the value safely for insertion as a literal string in an XML file.
  - e.g. `<element>$Field.CDATA</element>` will ensure that the `<element>` body is safely escaped as a string.
- [`JS()`](api:SilverStripe\ORM\FieldType\DBField::JS()) - ensures that text is properly escaped for use in JavaScript.
  - e.g. `var fieldVal = '$Field.JS';` can be used in JavaScript defined in templates to encode values safely.
- [`ATT()`](api:SilverStripe\ORM\FieldType\DBField::ATT()) (and its alias [`HTMLATT()`](api:SilverStripe\ORM\FieldType\DBField::HTMLATT())) - formats the value appropriate for an HTML attribute string
  - e.g. `<div data-my-field="$MyField.HTMLATT"></div>`
- [`RAWURLATT()`](api:SilverStripe\ORM\FieldType\DBField::RAWURLATT()) - encodes strings for use in URLs via [`rawurlencode()`](https://www.php.net/manual/en/function.rawurlencode.php)
- [`URLATT()`](api:SilverStripe\ORM\FieldType\DBField::URLATT()) - encodes strings for use in URLs via [`urlencode()`](https://www.php.net/manual/en/function.urlencode.php)
- [`JSON()`](api:SilverStripe\ORM\FieldType\DBField::JSON()) - encodes the value as a JSON string via [`json_encode()`](https://www.php.net/manual/en/function.json-encode.php)

[info]
See [the API documentation](api:SilverStripe\ORM\FieldType) for all the formatting methods available to you for the various field types.
[/info]

## Escaping HTML values in templates {#escaping}

[notice]
For specific security advice related to escaping values, see the [Security](/developer_guides/security/secure_coding/#xss-cross-site-scripting)
documentation.
[/notice]

The concept of escaping values in templates is ultimately just a combination of formatting and casting.

Values are typically escaped (i.e. the special HTML characters are encoded) in templates by either not
declaring a casting type, or by defaulting to the `Text` casting type defined on `ViewableData`.

See the [casting](#casting) section above for
instructions on configuring your model to declare casting types for fields, and how some of the more common
casting types affect escaping.

[info]
In addition to escaping via casting, `DBField` instances have an `escape_type` configuration property which is
either set to `"xml"` or `"raw"`. This configuration tells you whether XML content will be escaped or not, but does
*not* actually directly affect the casting of the value in templates. That is determined by what is returned from
the `forTemplate()` method (or any method explicitly called from within the template).
[/info]

### Escape methods in templates

Within the template, fields can have their encoding customised at a certain level with format methods.

See the [formatting](#formatting) section above for some of the more common formatting methods available
and how they affect escaping.

[hint]
If you are unsure of whether the field has been cast to `HTMLText` but you know
it contains safe HTML content, you can use `.RAW` to ensure the HTML is not escaped.
[/hint]

[warning]
Be careful using `.RAW` on non HTML field types - if the value being formatted includes content provided
by the user you could be introducing attack vectors for cross-site scripting attacks.
[/warning]

## Cast summary methods

Certain subclasses of DBField also have additional summary or manipulations methods, each of
which can be chained in order to perform more complicated manipulations.

For instance, The following class methods can be used in templates for the below types:

Text / HTMLText methods:

- [`Plain()`](api:SilverStripe\ORM\FieldType\DBString::Plain()) Will convert any HTML to plain text version. For example, could be used for plain-text
  version of emails.
- [`LimitSentences(<num>)`](api:SilverStripe\ORM\FieldType\DBText::LimitSentences()) - limits output to the first `<num>` sentences in the content. This method internally calls `Plain()`,
  converting HTML content into plain text.

## Related lessons

- [Dealing with arbitrary template data](https://www.silverstripe.org/learn/lessons/v4/dealing-with-arbitrary-template-data-1)
