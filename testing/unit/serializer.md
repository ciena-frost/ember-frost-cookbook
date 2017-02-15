# Recipe for: Testing a Serializer

## The API Response

```javascript
{
  "items": [
    {"id": "1"},
    {"id": "2"}
  ],
  "limit": 10,
  "nextPageToken": null,
  "offset": 0,
  "total": 2
}
```

## The Serializer

```javascript
import _ from 'lodash'
import Ember from 'ember'
import DS from 'ember-data'

export default DS.RESTSerializer.extend({
  /* DS.RESTSerializer method */
  normalizeArrayResponse (store, primaryModelClass, payload, id, requestType) {
    _.assign(payload, {
      domains: payload.items, // Ember Data expects records to live on key based on model name
      meta: {} // Ember Data expects meta data to live on an object with the key "meta"
    })

    delete payload.items;

    // Move meta data to "meta" key to satisfy Ember Data
    ['limit', 'nextPageToken', 'offset', 'total'].forEach((key) => {
      payload.meta[key] = payload[key]
      delete payload[key]
    })

    return this._normalizeResponse(store, primaryModelClass, payload, id, requestType, false)
  }
})
```

## The Serializer Test

```javascript
import {expect} from 'chai'
import DS from 'ember-data'
import {setupTest} from 'ember-mocha'
import {beforeEach, describe, it} from 'mocha'

import setupStore from '../../helpers/store'
import Domain from '../../../app/models/domain'

describe('Unit: Serializer | domain', function () {
  setupTest('serializer:domain', {
    unit: true
  })

  let env, mockDomainClass, serializer

  beforeEach(function () {
    serializer = this.subject()
    env = setupStore({
      adapter: DS.RESTAdapter,
      domain: Domain,
      serializer
    })
    mockDomainClass = env.store.modelFor('domain')
  })

  it('specifies proper modelName', function () {
    const actual = serializer.get('modelName')
    expect(actual).to.equal('domain')
  })

  describe('.normalizeArrayResponse()', function () {
    let payload

    beforeEach(function () {
      const items = [
        {id: '1'},
        {id: '2'}
      ]
      payload = {
        items,
        limit: 10,
        nextPageToken: null,
        offset: 0,
        total: items.length
      }
    })

    it('errors when items is missing', function () {
      delete payload.items
      expect(function () {
        serializer.normalizeArrayResponse(env.store, mockDomainClass, payload, null, null)
      }).to.throw()
    })

    ;['limit', 'nextPageToken', 'offset', 'total'].forEach((key) => {
      it(`does not error when ${key} is missing`, function () {
        delete payload[key]
        expect(function () {
          serializer.normalizeArrayResponse(env.store, mockDomainClass, payload, null, null)
        }).not.to.throw()
      })
    })

    describe('response', function () {
      let response

      beforeEach(function () {
        response = serializer.normalizeArrayResponse(env.store, mockDomainClass, payload, null, null)
      })

      it('contains expected data', function () {
        expect(response.data).to.eql([
          {id: '1', type: 'domain', attributes: {}, relationships: {}},
          {id: '2', type: 'domain', attributes: {}, relationships: {}}
        ])
      })

      it('contains expected included data', function () {
        expect(response.included).to.eql([])
      })

      it('contains expected meta data', function () {
        expect(response.meta).to.eql({
          limit: 10,
          nextPageToken: null,
          offset: 0,
          total: 2
        })
      })
    })
  })
})
```

### setupStore

```javascript
/**
 * @reference https://github.com/emberjs/data/blob/master/tests/helpers/store.js
 */

import Ember from 'ember'
const {Container, Registry} = Ember
import DS from 'ember-data'

import Owner from './owner'

// FIXME: was overly complex before adding eslint rule
/* eslint-disable complexity */
export default function setupStore (options) {
  var container, registry, owner
  var env = {}
  options = options || {}

  if (Registry) {
    registry = env.registry = new Registry()
    owner = Owner.create({
      __registry__: registry
    })
    container = env.container = registry.container({
      owner: owner
    })
    owner.__container__ = container
  } else {
    container = env.container = new Container()
    registry = env.registry = container
  }

  env.replaceContainerNormalize = function replaceContainerNormalize (fn) {
    if (env.registry) {
      env.registry.normalize = fn
    } else {
      env.container.normalize = fn
    }
  }

  var adapter = env.adapter = (options.adapter || '-default')
  delete options.adapter

  if (typeof adapter !== 'string') {
    env.registry.register('adapter:-ember-data-test-custom', adapter)
    adapter = '-ember-data-test-custom'
  }

  for (var prop in options) {
    registry.register('model:' + Ember.String.dasherize(prop), options[prop])
  }

  registry.register('service:store', DS.Store.extend({
    adapter: adapter
  }))

  registry.optionsForType('serializer', {singleton: false})
  registry.optionsForType('adapter', {singleton: false})
  registry.register('adapter:-default', DS.Adapter)

  registry.register('serializer:-default', DS.JSONSerializer)
  registry.register('serializer:-rest', DS.RESTSerializer)

  registry.register('adapter:-rest', DS.RESTAdapter)

  registry.register('adapter:-json-api', DS.JSONAPIAdapter)
  registry.register('serializer:-json-api', DS.JSONAPISerializer)

  env.restSerializer = container.lookup('serializer:-rest')
  env.store = container.lookup('service:store')
  env.serializer = env.store.serializerFor('-default')
  env.adapter = env.store.get('defaultAdapter')

  return env
}
/* eslint-enable complexity */

export {setupStore}

export function createStore (options) {
  return setupStore(options).store
}
```

### Owner

```javascript
/**
 * @reference https://github.com/emberjs/data/blob/master/tests/helpers/owner.js
 */

import Ember from 'ember'
const {_ContainerProxyMixin, _RegistryProxyMixin} = Ember

let Owner

if (_RegistryProxyMixin && _ContainerProxyMixin) {
  Owner = Ember.Object.extend(_RegistryProxyMixin, _ContainerProxyMixin)
} else {
  Owner = Ember.Object.extend()
}

export default Owner
```
