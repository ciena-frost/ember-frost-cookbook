# Recipe for: Testing a Model

## The Model

```javascript
import DS from 'ember-data'
const {attr} = DS
import HashModel from './hash-model'

export default HashModel.extend({
  accessUrl: attr('string'),
  address: attr(undefined, {default: {longitude: 0, latitude: 0}}),
  description: attr('string'),
  domainType: attr('string'),
  operationMode: attr('string'),
  properties: attr(),
  rpId: attr('string'),
  tenantId: attr('string'),
  title: attr('string'),
  lastConnected: attr('string'),
  reason: attr('string'),
  connectionStatus: attr('string')
})
```

## The Model Test

```javascript
import {expect} from 'chai'
import {setupModelTest} from 'ember-mocha'
import {beforeEach, describe, it} from 'mocha'

const expectedAttrs = [
  '_hash',
  'accessUrl',
  'address',
  'description',
  'domainType',
  'operationMode',
  'properties',
  'rpId',
  'tenantId',
  'title',
  'lastConnected',
  'connectionStatus',
  'reason'
]

describe('Unit / Model / domain', function () {
  setupModelTest('domain', {
    needs: ['model:resource-provider']
  })

  let actualAttrs, model

  beforeEach(function () {
    model = this.subject()
    actualAttrs = Object.keys(model.toJSON())
  })

  it('defines expected attributes', function () {
    expectedAttrs.forEach((name) => {
      expect(
        actualAttrs,
        `defines attribute "${name}"`
      )
        .to.include(name)
    })
  })
})
```