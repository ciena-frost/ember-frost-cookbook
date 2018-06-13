# Recipe for: Testing a Route

## The Route

```javascript
import Ember from 'ember'
const {RSVP} = Ember
import ProtectedRoute from '../../protected/route'
import DomainView from 'frost-orchestrate-ui/bunsen-views/domains/detail'
import {detailModel} from 'frost-orchestrate-ui/bunsen-models/model-from-property-definition'
import {
  getBunsenDomainViewId,
  mergeViewSchema,
  parseFile,
  createViewPartial
} from 'frost-orchestrate-ui/utils'
import _ from 'lodash'

const TYPE_PROPS_CONTAINER_KEY = 'typeProps'

export default ProtectedRoute.extend({
  pollInterval: 5000,
  subRoutes: [
    'product',
    'resource'
  ],

  pageTitle: 'Domain',

  activate () {
    document.title = this.get('pageTitle')
    Ember.$('body').addClass('scroll')

    const pollInterval = this.get('pollInterval')
    let poller = this.get('poller')

    if (!poller) {
      poller = this.get('pollboy').add(this, this.onPoll, pollInterval)
      this.set('poller', poller)
    }
  },

  deactivate () {
    Ember.$('body').removeClass('scroll')

    // Make sure we stop polling when route is left
    const poller = this.get('poller')
    this.get('pollboy').remove(poller)
  },

  model (params) {
    const id = params.id
    return this.store.findRecord('domain', id)
      .then((value) => {
        return this.store.findRecord('resource-provider', value.get('rpId'))
          .then((provider) => {
            const providerModel = provider.toJSON()
            const bunsenModel = detailModel(providerModel, 'domain')
            return this.store.findRecord('file', getBunsenDomainViewId(providerModel.domainType, 'detail'))
              .then((view) => {
                const bunsenView = mergeViewSchema(_.cloneDeep(DomainView), parseFile(view), TYPE_PROPS_CONTAINER_KEY)
                return {
                  bunsenModel,
                  bunsenView,
                  value,
                  id
                }
              })
              .catch(() => {
                const bunsenView = mergeViewSchema(
                  _.cloneDeep(DomainView),
                  createViewPartial(providerModel),
                  TYPE_PROPS_CONTAINER_KEY
                )

                return {
                  bunsenModel,
                  bunsenView,
                  value,
                  id
                }
              })
          })
      })
  },

  afterModel (model) {
    this.set('pageTitle', `${model.value.get('title')} - Domain`)
  },

  getProductCount () {
    const id = this.controller.get('model').id
    return this.store.query('product', {q: `domainId:${id}`}).then((products) => {
      this.controller.set('productCount', products.meta.total)
    })
  },

  getInventoryCount () {
    const id = this.controller.get('model').id
    return this.store.query('resource', {domainId: id}).then((resources) => {
      this.controller.set('inventoryCount', resources.meta.total)
    })
  },

  onPoll () {
    return RSVP.all([
      this.getProductCount(),
      this.getInventoryCount()
    ])
  },

  setupController (controller, model) {
    this._super(...arguments)
    this.getInventoryCount()
    this.getProductCount()
    controller.listenForEvents()
  },

  resetController (controller) {
    this._super(...arguments)
    controller.stopListeningForEvents()
  }
})
```

## The Route Test

```javascript
/**
 * Details domain route test
 */

import $ from 'jquery'
import _ from 'lodash'
import {expect} from 'chai'
import {setupTest} from 'ember-mocha'
import {afterEach, beforeEach, describe, it} from 'mocha'
import Ember from 'ember'
const {Object: EmberObject, Service, run, RSVP} = Ember
import sinon from 'sinon'

import DomainView from 'frost-orchestrate-ui/bunsen-views/domains/detail'
import {detailModel} from 'frost-orchestrate-ui/bunsen-models/model-from-property-definition'
import {mergeViewSchema, parseFile, createViewPartial} from 'frost-orchestrate-ui/utils'

const jQueryPrototype = Object.getPrototypeOf($('body'))

describe('Unit / Route / details/domain', function () {
  setupTest('route:details/domain', {
    unit: true
  })

  let route, sandbox

  beforeEach(function () {
    sandbox = sinon.sandbox.create()
    route = this.subject()
  })

  afterEach(function () {
    sandbox.restore()
  })

  it('exists', function () {
    expect(route).not.to.equal(undefined)
  })

  describe('.activate()', function () {
    let pollboy

    beforeEach(function () {
      pollboy = Service.create({
        add: sandbox.spy()
      })

      sandbox.spy(jQueryPrototype, 'addClass')
      route.set('pollboy', pollboy)

      document.title = ''
    })

    describe('when poller already present', function () {
      beforeEach(function () {
        route.set('poller', EmberObject.create({}))
        route.activate()
      })

      it('inits page title', function () {
        expect(document.title).to.eql('Domain')
      })

      it('adds scroll class to body', function () {
        expect(jQueryPrototype.addClass.lastCall.args).to.eql(['scroll'])
      })

      it('does not add poller', function () {
        expect(pollboy.add.callCount).to.equal(0)
      })
    })

    describe('when poller not already present', function () {
      beforeEach(function () {
        route.activate()
      })

      it('inits page title', function () {
        expect(document.title).to.eql('Domain')
      })

      it('adds scroll class to body', function () {
        expect(jQueryPrototype.addClass.lastCall.args).to.eql(['scroll'])
      })

      it('adds poller', function () {
        expect(pollboy.add.callCount).to.equal(1)
        expect(pollboy.add.lastCall.args).to.eql([route, route.onPoll, route.get('pollInterval')])
      })
    })
  })

  describe('.afterModel()', function () {
    beforeEach(function () {
      const model = {
        value: EmberObject.create({title: 'test-title'})
      }
      document.title = ''
      route.afterModel(model)
    })

    it('sets item page title', function () {
      expect(route.get('pageTitle')).to.eql('test-title - Domain')
    })
  })

  describe('.deactivate()', function () {
    let pollboy, poller

    beforeEach(function () {
      pollboy = Service.create({
        remove: sandbox.spy()
      })

      poller = EmberObject.create({})

      sandbox.spy(jQueryPrototype, 'removeClass')

      route.setProperties({
        pollboy,
        poller
      })

      route.deactivate()
    })

    it('removes scroll class from body', function () {
      expect(jQueryPrototype.removeClass.lastCall.args).to.eql(['scroll'])
    })

    it('removes poller', function () {
      expect(pollboy.remove.callCount).to.equal(1)
      expect(pollboy.remove.lastCall.args).to.eql([poller])
    })
  })

  describe('.getProductCount(), .getInventoryCount', function () {
    beforeEach(function () {
      route.controller = EmberObject.create({ model: { id: 1 } })
      route.store = {
        query: () => {
          return new RSVP.Promise((resolve, reject) => {
            resolve(EmberObject.create({
              length: 1,
              meta: {
                total: 3
              }
            }))
          })
        }
      }
    })

    it('gets product count', function (done) {
      route.getProductCount()
      run.later(() => {
        expect(route.controller.get('productCount')).to.eql(3)
        done()
      })
    })

    it('gets inventory count', function (done) {
      route.getInventoryCount()
      run.later(() => {
        expect(route.controller.get('inventoryCount')).to.eql(3)
        done()
      })
    })
  })

  describe('.model()', function () {
    let domain, providerModel, file, fileCopy, bunsenView, bunsenModel
    const params = {id: 1}
    beforeEach(function () {
      providerModel = {
        toJSON: () => {
          return {
            id: 'some-id',
            domainType: 'some-type',
            title: 'test-provider',
            properties: {
              fizz: {
                type: 'string'
              },
              buzz: {
                type: 'number'
              }
            }
          }
        }
      }

      domain = {
        id: 1,
        name: 'test',
        title: 'test',
        get: () => {
          return 'anything'
        }
      }

      const fileContents = {
        path: 'some-path',
        content: JSON.stringify({
          containers: [
            {
              id: 'accessInformation',
              rows: [
                [
                  {
                    model: 'properties.userName'
                  }
                ],
                [
                  {
                    model: 'properties.password',
                    properties: {
                      type: 'password'
                    }
                  }
                ],
                [
                  {
                    model: 'properties.projectName'
                  }
                ]
              ]
            }
          ]
        })
      }

      file = EmberObject.create(fileContents)
      fileCopy = EmberObject.create(_.cloneDeep(fileContents))
      bunsenModel = detailModel(providerModel.toJSON(), 'domain')
    })

    it('returns expected model when schema file is there', function (done) {
      bunsenView = mergeViewSchema(_.cloneDeep(DomainView), parseFile(fileCopy), 'typeProps')

      sandbox.stub(route.store, 'findRecord', (endpoint) => {
        switch (endpoint) {
          case 'domain':
            return RSVP.resolve(domain)

          case 'resource-provider':
            return RSVP.resolve(providerModel)

          case 'file':
            return RSVP.resolve(file)
        }
      })

      return route.model(params)
        .then((res) => {
          expect(JSON.stringify(res.bunsenModel)).to.eql(JSON.stringify(bunsenModel))
          expect(res.bunsenView).to.eql(bunsenView)
          expect(res.value).to.eql(domain)
          done()
        })
    })

    it('returns expected model when schema file isn\'t there', function () {
      bunsenView = mergeViewSchema(
        _.cloneDeep(DomainView),
        createViewPartial(providerModel.toJSON(), true),
        'typeProps'
      )

      sandbox.stub(route.store, 'findRecord', (endpoint) => {
        switch (endpoint) {
          case 'domain':
            return RSVP.resolve(domain)

          case 'resource-provider':
            return RSVP.resolve(providerModel)

          case 'file':
            return RSVP.reject()
        }
      })

      return route.model(params)
        .then((res) => {
          expect(JSON.stringify(res.bunsenModel)).to.eql(JSON.stringify(bunsenModel))
          expect(res.bunsenView).to.eql(bunsenView)
          expect(res.value).to.eql(domain)
        })
    })
  })

  describe('.onPoll()', function () {
    let inventoryDeferred, productDeferred, resolved

    beforeEach(function () {
      inventoryDeferred = RSVP.defer()
      productDeferred = RSVP.defer()
      resolved = false

      sandbox.stub(route, 'getInventoryCount').returns(inventoryDeferred.promise)
      sandbox.stub(route, 'getProductCount').returns(productDeferred.promise)

      route.onPoll()
        .then(() => {
          resolved = true
        })
    })

    it('does not resolve promise returned by onPoll', function () {
      expect(resolved).to.equal(false)
    })

    describe('when inventory count promise resolves', function () {
      beforeEach(function () {
        inventoryDeferred.resolve()
      })

      it('does not resolve promise returned by onPoll', function () {
        expect(resolved).to.equal(false)
      })

      describe('when product count promise resolved', function () {
        beforeEach(function () {
          productDeferred.resolve()
        })

        it('resolves promise returned by onPoll', function () {
          expect(resolved).to.equal(true)
        })
      })
    })

    describe('when product count promise resolves', function () {
      beforeEach(function () {
        productDeferred.resolve()
      })

      it('does not resolve promise returned by onPoll', function () {
        expect(resolved).to.equal(false)
      })

      describe('when inventory count promise resolved', function () {
        beforeEach(function () {
          inventoryDeferred.resolve()
        })

        it('resolves promise returned by onPoll', function () {
          expect(resolved).to.equal(true)
        })
      })
    })
  })

  describe('.resetController()', function () {
    let controller

    beforeEach(function () {
      controller = {
        stopListeningForEvents: sandbox.spy()
      }

      route.resetController(controller)
    })

    it('tells controller to stop listening for events', function () {
      expect(controller.stopListeningForEvents.callCount).to.equal(1)
    })
  })

  describe('.setupController()', function () {
    let controller, model

    beforeEach(function () {
      controller = {
        foo: 'bar',
        listenForEvents: sandbox.spy()
      }
      model = {bar: 'baz'}

      sandbox.stub(route, 'getInventoryCount')
      sandbox.stub(route, 'getProductCount')

      route.setupController(controller, model)
    })

    it('gets inventory count', function () {
      expect(route.getInventoryCount.callCount).to.equal(1)
    })

    it('gets product count', function () {
      expect(route.getProductCount.callCount).to.equal(1)
    })

    it('tells controller to listen for events', function () {
      expect(controller.listenForEvents.callCount).to.equal(1)
    })
  })
})
```