# Recipe for: Testing the setting properties on a component cause DOM changes

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
import hbs from 'htmlbars-inline-precompile'
import {beforeEach, describe, it} from 'mocha'

describe('Integration / Component / my-example', function () {
  setupComponentTest('my-example', {integration: true})

  describe('Size property', function () {
    describe('when it is medium', function () {
      beforeEach(function () {
        this.render(hbs`
          {{my-example
            size='medium'
          }}
        `)
      })

      it('should have "medium" class', function () {
        expect(this.$('.my-class')).to.have.class('medium')
      })
    })

    describe('when it is large', function () {
      beforeEach(function () {
        this.render(hbs`
          {{my-example
            size='large'
          }}
        `)
      })

      it('should have "large" class', function () {
        expect(this.$('.my-class')).to.have.class('large')
      })
    })
  })

  describe('Label property', function () {
    beforeEach(function () {
      this.render(hbs`
        {{my-example
          label='lorem ipsum'
        }}
      `)
    })

    it('should set the "label" property', function () {
      expect(
        this.$('.my-class').find('label').text().trim()
      ).to.eql('lorem ipsum')
    })
  })
})
```