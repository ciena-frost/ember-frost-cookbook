# Recipe for: Testing a Service

## The Service

```javascript
import Ember from 'ember'
const {
  Service,
  run: {
    scheduleOnce
  }
 } = Ember

export default Service.extend({

  // == State properties ======================================================

  isActive: false,
  noBlur: false,

  // == Functions =============================================================

  setState (modalName, isVisible, noBlur) {
    scheduleOnce('sync', this, function () {
      this.setProperties({
        isActive: isVisible,
        noBlur
      })
    })
  }
})
```

## The Service Test

```javascript
import {expect} from 'chai'
import {setupTest} from 'ember-mocha'
import {beforeEach, describe, it} from 'mocha'

describe('Unit / Service / frost-modal', function () {
  setupTest('service:frost-modal', {
    unit: true
  })

  let service

  beforeEach(function () {
    service = this.subject()
  })

  it('should default isActive to false', function () {
    expect(service.get('isActive')).to.equal(false)
  })

  it('should default noBlur to false', function () {
    expect(service.get('noBlur')).to.equal(false)
  })

  describe('setState()', function () {
    beforeEach(function () {
      service.setState('myTestModal', true, true)
    })

    it('sets isActive to passed value', function () {
      expect(service.get('isActive')).to.equal(true)
    })

    it('sets noBlur to passed value', function () {
      expect(service.get('noBlur')).to.equal(true)
    })
  })
})
```