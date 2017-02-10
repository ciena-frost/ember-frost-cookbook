# Recipe for: Testing the default rendered state of a Component

In this test we are concerned with the default rendered state of the component (only populating the required properties)
- That default classes are applied
- That default properties/attributes (values) are applied
- The default characteristics of any mixins used by the component

## The Component Template (my-example.hbs)

```handlebars
<label for={{inputId}} class='label' tabIndex=0>
    {{label}}
    <svg xmlns='http://www.w3.org/2000/svg' viewBox='3 2 31 31'>
        <rect xmlns='http://www.w3.org/2000/svg'  x='10.7' y='15.3' transform='matrix(-0.7071 0.7071 -0.7071 -0.7071 35.6974 26.188)' fill='%23009EEF' width='3.5' height='10.3'/>
        <polygon xmlns='http://www.w3.org/2000/svg'  fill='%23009EEF' points='14.9,25.4 12.4,22.9 26.2,9.4 28.7,12.1'/>
  </svg>
</label>
```

## The Component

```javascript
import Ember from 'ember'
const {Component, computed} = Ember
import layout from '../templates/components/my-example'

/**
 * @module
 * @augments ember/Component
 */
const component = Component.extend({
    layout: layout,
    classNames: ['my-class'],
    classNameBindings: ['sizeClass'],

    /**
     * Sets the size for the classNameBinding
     *
     * @function
     * @returns {String} Defaults to "small"
     */
    sizeClass: computed('size', function () {
        return this.get('size') || 'small'
    })
})
```

## The Component Test

```javascript
import {expect} from 'chai'
import {setupComponentTest} from 'ember-mocha'
import {beforeEach, describe, it} from 'mocha'
import hbs from 'htmlbars-inline-precompile'


describe('Integration / Component / my-example', function () {
  setupComponentTest('my-example', {integration: true})

  describe('when rendered w/o any options', function () {
    beforeEach(function () {
      this.render(hbs`
        {{my-example}}
      `)
    })

    it('should have the class "small"', function () {
      expect(this.$('.my-class')).to.have.class('small')
    })

    it('should have the class "label"', function () {
      expect(this.$('.my-class label')).to.have.class('label')
    })

    it('should set "for" on the label to the "id" of the input', function () {
      const forValue = this.$('.my-class label').prop('for')
      const inputId = this.$('.my-class input').prop('id')
      expect(forValue).to.equal(inputId)
    })


    it('should start out with a 0 tabIndex on the label', function () {
      expect(this.$('.my-class label')).to.have.prop('tabIndex', 0)
    })

    it('should have a svg', function () {
      expect(this.$('.my-class svg')).to.have.length(1)
    })
  })
})
```