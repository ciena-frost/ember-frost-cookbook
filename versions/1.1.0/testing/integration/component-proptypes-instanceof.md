# Recipe for: Satisfying an ember-prop-types instanceOf model constraint

## The Component

```javascript
import Ember from 'ember'
import PropTypeMixin, {PropTypes} from 'ember-prop-types'
import ResourceModel from '../models/resource'

const {Component} = Ember

/**
 * @module
 * @augments ember/Component
 * @augments ember-prop-types/PropTypeMixin
 */
export default Component.extend(PropTypeMixin, {
  propTypes: {
    resource: PropTypes.instanceOf(ResourceModel).isRequired
  }
})
```

## The Component Test

```javascript
import {expect} from 'chai'
import Ember from 'ember'
import {integration} from 'ember-test-utils/test-support/setup-component-test'
import hbs from 'htmlbars-inline-precompile'
import ResourceModel from '../models/resource'
import {beforeEach, describe, it} from 'mocha'

const {getOwner} = Ember

const test = integration('my-example')

describe(test.label, function () {
  test.setup()

  describe('dialog', function () {
    beforeEach(function () {
      const store = getOwner(this).lookup('service:store')

      this.registry.register('model:resource', ResourceModel)

      this.set('resource', store.createRecord('resource'))

      this.render(hbs`
        {{my-example
          resource=resource
        }}
      `)
    })

    it('should work', function () {
      ...
    })
  })
```
