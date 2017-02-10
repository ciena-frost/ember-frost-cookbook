# Recipe for: Testing an Adapter

## The Adapter

```javascript
import DS from 'ember-data'
import Ember from 'ember'
import DataAdapterMixin from 'frost-authentication/mixins/data-adapter-mixin'

export default DS.RESTAdapter.extend(DataAdapterMixin, {

  /**
   * Endpoint prefix
   * @type {String}
   */
  namespace: '/bpocore/market/api/v1',

  /**
   * Determine path name for a given type
   * @param {String} modelName - name of model
   * @returns {String} path name for a given type
   */
  pathForType (modelName) {
    return Ember.String.pluralize(modelName)
  },

  sortQueryParams (params) {
    /**
     * TODO: remove me once backend supports functionality
     *
     * If a partial query is made and the value is empty the API chokes,
     * so we will simply remove the partial query parameter under that
     * scenario.
     *
     * @see https://agile-jira.ciena.com/browse/BPSO-35333
     */
    if (params.p && /(.+):(,|$)/.test(params.p)) {
      delete params.p
    }

    /**
     * TODO: remove me once backend supports functionality
     *
     * If a partial query is made with a space between the search word and an
     * asterisk the backend will choke and return a 500 status code with a
     * gnarly stacktrace.
     *
     * @see https://agile-jira.ciena.com/browse/BPSO-35332
     */
    if (params.p) {
      // Make sure there is no space before asterisk or API will choke
      params.p = params.p.replace(' *', '*')
    }

    return params
  }
})
```

## The Adapter Test

```javascript
import {expect} from 'chai'
import {setupTest} from 'ember-mocha'
import {beforeEach, describe, it} from 'mocha'

describe('Unit / Adapter / application', function () {
  let adapter

  setupTest('adapter:application', {
    unit: true
  })

  beforeEach(function () {
    adapter = this.subject()
  })

  it('has correct namespace', function () {
    expect(adapter.namespace).to.equal('/bpocore/market/api/v1')
  })

  it('.pathForType() returns pluralized non-camelized model name', function () {
    expect(adapter.pathForType('resource-type')).to.equal('resource-types')
  })

  it('removes invalid partial search', function () {
    expect(
      adapter.sortQueryParams({
        p: 'test:'
      })
    )
      .to.eql({})

    expect(
      adapter.sortQueryParams({
        p: 'test:,'
      })
    )
      .to.eql({})
  })

  it('removes space between partial search and asterisk', function () {
    expect(
      adapter.sortQueryParams({
        p: 'test *'
      })
    )
      .to.eql({
        p: 'test*'
      })
  })
})
```
