# Recipe for: Testing that action handlers on a component are wired up correctly

## The Component Template (my-example.hbs)

```handlebars
<label for={{inputId}} class='label' onBlur={{action 'onBlur'}} tabIndex=0>
        {{label}}
</label>
```

## The Component

```javascript
import Ember from 'ember'
const {Component} = Ember
import layout from '../templates/components/my-example'

/**
 * @module
 * @augments ember/Component
 */
const component = Component.extend({

    /** @type {Object} */
    actions: {

        /**
         * Calls the 'onBlur' bound action when the input loses focus
         *
         * @function actions:onBlur
         * @returns {undefined}
         */
        onBlur () {
            const onBlur = this.get('onBlur')
            if (onBlur) {
                onBlur()
            }
        }
    }
})
```

## The Component Test

```javascript
import {expect} from 'chai'
import {setupComponentTest} from 'ember-mocha'
import hbs from 'htmlbars-inline-precompile'
import {beforeEach, describe, it} from 'mocha'
import sinon from 'sinon'

describe('Integration / Component / my-example', function () {
  setupComponentTest('my-example', {integration: true})

  describe('when onBlur handler is set', function () {
    let blurHandler
    beforeEach(function () {
      blurHandler = sinon.stub()
      this.setProperties({blurHandler})
      this.render(hbs`
          {{my-example
          onBlur=(action blurHandler)
          }}
      `)
    })

    describe('when focus is lost', function() {
      beforeEach(function () {
        this.$('.my-class label').trigger('blur')
      })

      it('should call the blur handler', function () {
        expect(blurHandler).to.have.callCount(1)
      })
    })
  })
})
```