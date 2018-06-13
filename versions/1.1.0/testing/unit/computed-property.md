# Recipe for: Testing a Computed Property in a Component

## The Computed Property

```javascript
import Ember from 'ember'
const {Component, computed} = Ember
import layout from '../templates/components/my-example'

/**
 * @module
 * @augments ember/Component
 */
const component = Component.extend({

  /**
   * Whether this slot should be yielded
   *
   * @function
   * @returns {Boolean}
   */
  isSlotYield: computed('name', 'yieldedSlotName', function () {
    return this.get('name') === this.get('yieldedSlotName')
  })
})
```

## The Computed Property Test

```javascript
import {expect} from 'chai'
import {setupComponentTest} from 'ember-mocha'
import {beforeEach, describe, it} from 'mocha'

describe('Unit / Component / my-example', function () {
  setupComponentTest('my-example')

  let component
  describe('"isSlotYield" computed property', function () {
    describe('when "name" and "yieldSlotName" are equivalent', function () {
      beforeEach(function () {
        component = this.subject({
          name: 'wrongName',
          yieldedSlotName: 'testName'
        })
      })

      it('should set "isSlotYield" to true', function () {
        Ember.run(() => component.set('name', 'testName'))

        expect(component.get('isSlotYield')).to.equal(true)
      })
    })

    describe('when when "name" and "yieldedSlotName" are not equivalent', function () {
      beforeEach(function () {
        component = this.subject({
          name: 'wrongName',
          yieldedSlotName: 'testName'
        })
      })

      it('should sets "isSlotYield" to false', function () {
        expect(component.get('isSlotYield')).to.equal(false)
      })
    })
  })
})
```