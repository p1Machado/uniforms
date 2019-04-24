---
id: bridges-simple-schema
title: SimpleSchema
---

# SimpleSchema definition

**Note:** remember to import `uniforms-bridge-simple-schema` or `uniforms-bridge-simple-schema-2` package first.

```js
const PersonSchema = new SimpleSchema({
  // ...

  aboutMe: {
    type: String,
    uniforms: MyText, // Component...
    uniforms: {
      // ...or object...
      component: MyText, // ...with component...
      propA: 1 // ...and/or extra props.
    }
  }
});
```