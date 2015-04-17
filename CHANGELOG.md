# ember-watson Changelog

### 0.4.0

This version includes a new command which normalizes how you reference
models in `belongsTo` and `hasMany` relationships to their dasherized
from. Looking up via camelCase, variable, or App.Post will be removed
for Ember Data 1.0,

Running `ember watson:convert-ember-data-model-lookups` will transform
models looking like the following:

```
export default DS.Model.extend({
  postComments: DS.hasMany('postComment', {async: true})
});

```

To:

```
export default DS.Model.extend({
  postComments: DS.hasMany('post-comment', {async: true})
});
```

Big thanks to [Stanley Stuart](https://github.com/fivetanley) for his
work on this command.

### 0.3.1

A bug was introduced in `0.3.0` causing `import Ember from 'ember'` to
be included in any JavaScript file independently if prototype
extensions were used or not. This release fixes the issue so the
import declaration is only included if prototype extensions are
detected.

### 0.3.0

This release include  some improvements to existing transformations
and namespace all the commands.

Before you would call the commands like:

```
ember upgrade-qunit-tests
ember convert-prototype-extensions
```

Now they are namespaced under `watson:`

```
ember watson:upgrade-qunit-tests
ember watson:convert-prototype-extensions
```

#### QUnit transforms

Adds support to yield assert into the
beforeEach/afterEach callbacks [#7](https://github.com/abuiles/ember-watson/pull/7)

The following

```
module('Acceptance: FriendsNew', {
  setup: function() {
    App = startApp();
    ok(true, 'app started');
  },
  teardown: function() {
    Ember.run(App, 'destroy');
  }
});
```

Will be replaced with:

```
module('Acceptance: FriendsNew', {
  beforeEach: function(assert) {
    App = startApp();
    assert.ok(true, 'app started');
  },
  afterEach: function() {
    Ember.run(App, 'destroy');
  }
});
```

#### Prototype Extensions

Extends the transformations to include `.on` event observers, the
following

```
chainedObserver: function(property) {
  this.set('baz', true);
}.observes('foo', 'bar').on('init'),

onInit: function() {
  this.set('foobar', true);
}.on('init')
```

Is replaced with:

```
chainedObserver: Ember.on('init', Ember.observer('foo', 'bar', function(property) {
  this.set('baz', true);
})),

onInit: Ember.on('init', function() {
  this.set('foobar', true);
})
```

Also if `Ember` is not imported, the import will be added for you:

```
import Ember from 'ember'
```