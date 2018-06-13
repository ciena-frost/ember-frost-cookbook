# Recipe for: Testing that a Mixin is used in a Component

## The Mixin in the Component

```javascript
import Ember from 'ember'
const {Component} = Ember
import layout from '../templates/components/my-example'
import SomeMixin from '../mixins/some-mixin'

/**
 * @module
 * @augments ember/Component
 * @augments module:mixins/some-mixin
 */
const component = Component.extend(SomeMixin, {
    // The component code
})
```

## The Mixin in the Component Test

```javascript
import {expect} from 'chai'
import {setupComponentTest} from 'ember-mocha'
import {beforeEach, describe, it} from 'mocha'
import SomeMixin from 'your-namespace/mixins/some-mixin';

describe('Unit / Component / my-example', function () {
  setupComponentTest('my-example')

  let component

  describe('when created with only defaults', function () {
    beforeEach(function () {
      component = this.subject()
    })


    it('should have the SomeMixin', function () {
      expect(SomeMixin.detect(component)).to.equal(true)
    })
  })
})
```