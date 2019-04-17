# JSON Forms Specification v1.0

- [Terminology](#terminology)

## Terminology

- **Field**

  In this document a field refers to any item that will be appear in your form.

  A field in this context can include (but is not limited to):
      
  - Form inputs (text, select, checkbox)
  - Custom form widgets such as an address typeahead or a date picker.
  - Structural form elements like sections/fieldsets.
  - Layout elements such as a bootstrap style row or column.
  - Rich text or html blocks.

  The type of field is driven by the `type` property of the field. See the documentation for [fields](#Field).

- **Field type**

- **Form rule**

  In this document a form rule refers to a rule applied to a property of a field when a condition is met.  

  An example of a form rule is a 'show when' rule, which will conditionally show/hide a field.

  See the documentation for [form rules](#Form_Rules).

  A form rule is **NOT** a validation rule. See [validators](#Validators) for validation rules.

## Addressing nested fields

There are many places where you will need to reference a field within the form as part of a [form rule](#Form_rules) or a [validator](#Validators).

The syntax for referencing a field is tied to it's path in the nested field structure.

For example, a simple non-nested field can be referenced by it's id:

```js
{
  {
  "version": "0",
  "id": "form-id",
  "fields": [
    { 
      "id": "firstName", // This field can be referenced by 'firstName'
      "type": "text"
    }
  ]
}
```

Nested fields can be referenced by their path in the nested tree, each id seperated by a period.

```js
{
  {
  "version": "0",
  "id": "form-id",
  "fields": [
    { 
      "id": "details", // This field can be referenced by 'details'
      "type": "section"
      "fields": [
        "id": "name", // This field can be referenced by 'details.name'
        "type": "section"
        "fields": [
          "id": "first", // This field can be referenced by 'details.name.first'
          "type": "section"
        ]
      ]
    }
  ]
}
```

This follows the same convention as defined in the section on [data format](#data_format).

## JSON Structure

### Top level structure

A form **MUST** have these top level members:

- `version`: an integer representing the version number of the JSON forms spec
- `id`: a string used to identify the form. **MUST** be unique among other forms.
- `fields`: an array of [field](#Field) objects, representing the fields that should be rendered by the form builder.

A form **MAY** have these top level members:

- `initialValues`: an object containing the initial values of the form, see the section on [values and error format](#data_format) for the expected structure of this object.
- `data`: an object used for form level configuration. This would used differently on a per project basis.

  Examples of form level configuration might include:
  - a 'formType' flag that controls how the form would render
  - an 'endpoint' flag that controls where the form submits to

### Fields

A field **MUST** have these properties:

- `id`: a string used to identify the field. **MUST** be unique among it's siblings. See the section on [Values and error format](#data_format) for how this id is used.
- `type`: a string representing the type of field this is. The type will determine both the rendering of the field and the options it can take.

A field **MAY** have these properties:

- `rules`: an array of [form rule](#Form_rules) objects.
- `required`: a boolean value determining whether this field needs to be completed to submit the form. Defaults to `false`.
- `requiredMessage`: either a string or a [text node](#TextNodes) object used to customise the required field error message. If omitted this will default to a standard message.
- `validators`: an array of [field validator](#Validators) objects.
- `fields`: an array of nested [field](#Fields) objects. This would used differently for different field types, but where possible you should use [these existing conventions](#Conventions).
- `data`: an object used for field level configuration. This would used differently for different field types, but where possible you should use [these existing conventions](#Conventions).

#### Examples

A simple text input may look like this:

```js
{
  "id": "myFieldId",
  "type": "text",
  "data": {
    // Arbitrary data suited to your needs
    // This might include labels, tooltips etc.
  }
}
```

A structural form 'section' may look like this, with nested fields:

```js
{
  "id": "myFieldId",
  "type": "section",
  "data": {
    // Arbitrary data suited to your needs
    // This might include a section title and description.
  },
  "fields": [
    ...fields that appears inside the section
  ]
}
```

### TextNodes

A TextNode is a standard object used to represent content for labels, tooltips etc.

A TextNode **MUST** have these properties:

- `type`: Can be `text` or `richtext`, this is used to tell the client how to render the content.
- `content`: a string which should contain escaped html or plain text

> **A note on rich text content**
> 
> When escaping embedded html inside a json string, there are a number of rules to follow.
> 
> Visit this link for more details [https://www.freeformatter.com/json-escape.html](https://www.freeformatter.com/json-escape.html)

#### Examples

*Plain text*

```js
{
    "type": "text",
    "content": "Any plain text content"
}
```

*Rich text*

```js
{
    "type": "richtext",
    "content": "Whatever <span class=\"highlight\">rich text<\/span> you <a>need<\/a>."
}
```

### Form Rules

Use rules when you need to add conditionally logic to a field based on the values of other fields.

> Rules can be read as: do `action` when `condition`
>
> ```js
> {
>    "action": "show",
>    "when": Condition
> }
> ```

A rule **MUST** have these properties:

- `action`: a string used to determines which behaviour to use, see the table below for defined actions.
- `when`: a [condition](#Conditions) object which describes when the action should apply.

#### Action types

The actions defined are:

| action | description                                                |
|:-------|:-----------------------------------------------------------|
| show   | Show the field only when the condition evaluates to `true` |
| hide   | Hide the field when the condition evaluates to `true`      |

> Fields that are not shown must not be validated, and their values won't be sent on submission.

#### Conditions

A condition can take the following forms:

The **eq** condition compares the value of a field to another value. The condition evaluates to `true` if the values match (using the `===` operator in javascript).

- `eq`: An array containing the following parameters:
  - `0`: the `id` of the field to be compared against
  - `1`: the value to compare
```js
{ 'eq': [ '{field-id}', value ] }
```

---

The **all** condition takes in one or more other conditions and evaluates to `true` if and only if all it's conditions evaluate to `true`. (You can think of this as 'and' or 'every')

- `all`: An array of condition objects
```js
{ 'all': [ ...Condition ] }
```

---

The **any** condition takes in one or more other conditions and evaluates to `true` if at least one of it's conditions evaluate to `true`. (You can think of this as 'or' or 'some')

- `any`: An array of condition objects
```js
{ 'any': [ ...Condition ] }
```

---

The **not** condition takes in a single condition and simply negates the condition.

- `not`: A single condition
```js
{ 'not': Condition }
```

#### Examples

Conditions can be be combined to create complex rules. The rule below says:

`Show me when section1.field1 EQUALS "myValue" AND section1.field2 does NOT EQUAL "myOtherValue"`

```js
{
  "action": "show",
  "when": {
    "all": [
      { 
        "eq": ["section1.field1", "myValue"] 
      },
      { 
        "not": {
          "eq": ["section1.field2", "myOtherValue"] 
        }
      }
    ]
  }
}
```

### Validators

A Validator MUST have these properties:

- `type`: a string determining the type/name of the validation rule to use.

A Validator MAY have these properties:

- `message`: either a string or a [text node](#TextNodes) object used to customise the error message displayed to the user.
- `options`: an object used to configure the validation rule. This would vary between each type of validator.

#### Examples

The validator below will show a custom message to the user if their first name is over 20 characters long.

```js
{
  "type": "stringMax",
  "message": "First name must be less than 20 characters long",
  "options": {
    "max": 20
  }
}
```

### Data format

A common format will be used when sending or recieving values or errors in a form.

The shape of the data being sent/received will match the nested shape of the fields configuration.

The field's id should match the id in the field config.

For example, say your fields are configured as below:

```js
{
  // ... rest of the form json omitted for clarity
  // ...

  "fields": [
    {
      "id": "phone",
      "type": "tel"
    },
    {
      "id": "name",
      "type": "section",
      "fields": [
        {
          "id": "first",
          "type": "text"
        },
        {
          "id": "last",
          "type": "text"
        }
      ]
    }
  ]
}
```

The values returned on submit of the form would look like this:

```js
{
  "phone": "0433111222",
  "name": {
    "first": "Joan",
    "last": "Smith"
  }
}
```

This format is currently used in the following scenarios:

- Setting initial form values (See [Top level structure](#Top_level_structure))
- Once the form has been submitted by the user, the form values will be available in this format to send to API.
- Errors from an API can be sent back to the form buider in this format, with the error message as the value.

### Conventions

New field types and validators can be registered or overridden in the front end form builder, which gives the full flexibility needed per implementation.

There is however some conventions that should be followed where possible, particurly when your client is a browser:

#### 'input' type names should be reserved for fields that implement those those html inputs.

Types such as:

- 'text'
- 'checkbox'
- 'radio'
- 'tel'
- 'number'
- or any other [html5 input type](#https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input)

should usually be reserved for fields that are implemented as an `<input type="{the_type}" />` to prevent any confusion.

#### Common field properties such as labels, tooltips etc. should use a standard format.

While this is not enforced, it's recommended that any common field 'content' properties are added to the `data` property in the following format:

```js
{
  "id": "name",
  "type": "text",
  "data": {
    "label": String | TextNode,
    "tooltip": String | TextNode,
    "helpText": {
      "above": String | TextNode,
      "below": String | TextNode,
    },
    "placeholder": String,
  }
}
```

#### Fields with multiple predefined values to choose from (select, radio group etc.) should use a standard `options` array.


```js
{
  "id": "name",
  "type": "select",
  "data": {
    "options": [
      {
        "value": "value_1",
        "label": String || TextNode,
      },
      {
        "value": "value_2",
        "label": String || TextNode,
        // ... other properties could be added here as needed, an 'icon' example. 
      }
    ],
  }
}
```
