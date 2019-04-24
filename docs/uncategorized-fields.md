---
id: uncategorized-fields
title: Fields
---

## `AutoField` algorithm

```js
let component = props.component;
if (component === undefined) {
  if (props.allowedValues) {
    if (props.checkboxes && props.fieldType !== Array) {
      component = RadioField;
    } else {
      component = SelectField;
    }
  } else {
    switch (props.fieldType) {
      case Date:
        component = DateField;
        break;
      case Array:
        component = ListField;
        break;
      case Number:
        component = NumField;
        break;
      case Object:
        component = NestField;
        break;
      case String:
        component = TextField;
        break;
      case Boolean:
        component = BoolField;
        break;
    }

    invariant(component, 'Unsupported field type: %s', props.fieldType.toString());
  }
}
```

## Guaranteed props

**Note:** These are **not** the only props that a field will receive - these are guaranteed for all fields created with `connectField` helper.

|      Name      |         Type          |              Description               |
| :------------: | :-------------------: | :------------------------------------: |
|   `changed`    |        `bool`         |           Has field changed?           |
|   `disabled`   |        `bool`         |           Is field disabled?           |
|    `error`     |       `object`        | Field scoped part of validation error. |
| `errorMessage` |       `string`        | Field scoped validation error message. |
|    `field`     |       `object`        |     Field definition from schema.      |
|    `fields`    |   `arrayOf(string)`   |            Subfields names.            |
|  `fieldType`   |        `func`         |              Field type.               |
|  `findError`   |     `func(name)`      |      Request another field error.      |
|  `findField`   |     `func(name)`      |      Request another field field.      |
|  `findValue`   |     `func(name)`      |      Request another field value.      |
|      `id`      |       `string`        |      Field id - given or random.       |
|    `label`     |       `string`        |              Field label.              |
|     `name`     |       `string`        |              Field name.               |
|   `onChange`   | `func(value, [name])` |          Change field value.           |
|    `parent`    |       `object`        |          Parent field props.           |
| `placeholder`  |       `string`        |           Field placeholder.           |
|    `value`     |         `any`         |              Field value.              |

## Props propagation

Few props propagate in a very special way. These are `label`, `placeholder` and `disabled`.

**Example:**

```js
<TextField />                    // default label | no      placeholder
<TextField label="Text" />       // custom  label | no      placeholder
<TextField label={false} />      // no      label | no      placeholder
<TextField placeholder />        // default label | default placeholder
<TextField placeholder="Text" /> // default label | custom  placeholder

<NestField label={null}> // null = no label but the children have their labels
    <TextField />
</NestField>

<NestField label={false}> // false = no label and the children have no labels
    <TextField />
</NestField>

<ListField name="authors" disabled>          // Additions are disabled...
    <ListItemField name="$" disabled>        // ...deletion too
        <NestField disabled={false} name=""> // ...but editing is not.
            <TextField name="name" />
            <NumField  name="age" />
        </NestField>
    </ListItemField>
</ListField>
```

**Note:** `label`, `placeholder` and `disabled` are cast to `Boolean` before being passed to nested fields.

## Example: `CompositeField`

**Note:** This example uses `connectField` helper. To read more see [API](https://github.com/vazco/uniforms/blob/master/API.md#connectfield).

```js
import AutoField from 'uniforms/AutoField';
import React from 'react';
import connectField from 'uniforms/connectField';

// This field is a kind of a shortcut for few fields. You can also access all
// field props here, like value or onChange for some extra logic.
const Composite = () => (
  <section>
    <AutoField field="firstName" />
    <AutoField field="lastName" />
    <AutoField field="age" />
  </section>
);
export default connectField(Composite);
```

## Example: `CustomAutoField`

These are two _standard_ options to define a custom `AutoField`: either using `connectField` or simply taking the code from the [original one](https://github.com/vazco/uniforms/blob/master/packages/uniforms-unstyled/src/AutoField.js#L14-L47) _(theme doesn't matter)_ and simply apply own components and/or rules to render components. Below an example with `connectField`.

**Note:** This example uses `connectField` helper. To read more see [API](https://github.com/vazco/uniforms/blob/master/API.md#connectfield).

```js
// Remember to choose a correct theme package
import AutoField from 'uniforms-unstyled/AutoField';

const CustomAuto = props => {
  // This way we don't care about unhandled cases - we use default
  // AutoField as a fallback component.
  const Component = determineComponentFromProps(props) || AutoField;

  return <Component {...props} />;
};

const CustomAutoField = connectField(CustomAuto, {
  ensureValue: false,
  includeInChain: false,
  initialValue: false
});
```

You can also tell your `AutoForm`/`QuickForm`/`ValidatedQuickForm` to use it.

```js
<AutoForm {...props} autoField={CustomAutoField} />
```

## Example: `CycleField`

**Note:** This example uses `connectField` helper. To read more see [API](https://github.com/vazco/uniforms/blob/master/API.md#connectfield).

```js
import React from 'react';
import classnames from 'classnames';
import connectField from 'uniforms/connectField';

// This field works as follows: iterate all allowed values and optionally no-value
// state if the field is not required. This one uses Semantic-UI.
const Cycle = ({allowedValues, disabled, label, required, value, onChange}) => (
  <a
    className={classnames('ui', !value && 'basic', 'label')}
    onClick={() =>
      onChange(
        value
          ? allowedValues.indexOf(value) === allowedValues.length - 1
            ? required
              ? allowedValues[0]
              : null
            : allowedValues[allowedValues.indexOf(value) + 1]
          : allowedValues[0]
      )
    }
  >
    {value || label}
  </a>
);
export default connectField(Cycle);
```

## Example: `RangeField`

**Note:** This example uses `connectField` helper. To read more see [API](https://github.com/vazco/uniforms/blob/master/API.md#connectfield).

```js
import React from 'react';
import connectField from 'uniforms/connectField';

// This field works as follows: two datepickers are bound to each other. Value is
// a {start, stop} object.
const Range = ({onChange, value: {start, stop}}) => (
  <section>
    <DatePicker max={stop} value={start} onChange={start => onChange(start, stop)} />
    <DatePicker min={start} value={stop} onChange={stop => onChange(start, stop)} />
  </section>
);
export default connectField(Range);
```

## Example: `RatingField`

**Note:** This example uses `connectField` helper. To read more see [API](https://github.com/vazco/uniforms/blob/master/API.md#connectfield).

```js
import React from 'react';
import classnames from 'classnames';
import connectField from 'uniforms/connectField';

// This field works as follows: render stars for each rating and mark them as
// filled, if rating (value) is greater. This one uses Semantic-UI.
const Rating = ({className, disabled, max = 5, required, value, onChange}) => (
  <section className={classnames('ui', {disabled, required}, className, 'rating')}>
    {[...Array(max)]
      .map((_, index) => index + 1)
      .map(index => (
        <i
          key={index}
          className={classnames(index <= value && 'active', 'icon')}
          onClick={() => disabled || onChange(!required && value === index ? null : index)}
        />
      ))}
  </section>
);
export default connectField(Rating);
```