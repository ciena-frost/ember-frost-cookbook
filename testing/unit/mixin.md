# Recipe for: Testing an Mixin

## The Mixin

```javascript
import Ember from 'ember'
const {Mixin, on} = Ember
import FrostListCoreMixin from 'ember-frost-list/mixins/frost-list-core-mixin'

export default Mixin.create(FrostListCoreMixin, {
  // == Event =================================================================
  initListExpansionMixin: on('init', function () {
    this.set('expandedItems', Ember.Object.create())
  }),

  // == Actions ================================================================
  actions: {
    collapseItems () {
      let records = this.get('_listItems')
      let expandedItems = this.get('expandedItems')
      records.map((record) => {
        delete expandedItems[record.id]
      })
      this.notifyPropertyChange('expandedItems')
    },

    expandItems () {
      let records = this.get('_listItems')
      let expandedItems = this.get('expandedItems')
      records.map((record) => {
        expandedItems.set(record.id, true)
      })
      this.notifyPropertyChange('expandedItems')
    },

    collapseItem () {

    },

    expandItem () {

    }
  }
})
```

## The Mixin Test

```javascript
import {expect} from 'chai'
import Ember from 'ember'
const {Controller, Object} = Ember
import FrostListCoreMixin from 'ember-frost-list/mixins/frost-list-core-mixin'
import FrostListSelectionMixin from 'ember-frost-list/mixins/frost-list-selection-mixin'
import * as utils from 'ember-frost-list/utils/utils'
import {afterEach, beforeEach, describe, it} from 'mocha'
import sinon from 'sinon'

describe('Unit / Mixin / frost-list-selection-mixin', function () {
  let sandbox

  const testItems = [
    {
      id: '1',
      isSelected: false
    }
  ]
  let subject

  beforeEach(function () {
    sandbox = sinon.sandbox.create()
    let testObject = Controller.extend(FrostListSelectionMixin)
    subject = testObject.create({
      listConfig: {
        items: 'model'
      }
    })

    subject.set('model', testItems)
  })

  afterEach(function () {
    sandbox.restore()
  })

  it('sets up "selectedItems" with an empty object', function () {
    expect(
      subject.get('selectedItems')
    ).to.eql(Object.create())
  })

  it('has the expect Mixins', function () {
    expect(
      FrostListCoreMixin.detect(subject)
    ).to.equal(true)
  })

  describe('"selectedItem()" action', function () {
    it('udpates selectedItems', function () {
      sandbox.stub(utils, 'updateSelectedItemsHash').returns({ 1: true })

      subject.send('selectItem', {})

      expect(
        subject.get('selectedItems')
      ).to.eql({ 1: true })
    })

    it('calls "updateSelectedItemsHash()" with correct parameters', function () {
      const updateSelectedItemsHashSpy =
        sandbox.stub(utils, 'updateSelectedItemsHash').returns({ 1: true })

      subject.set('selectedItems', { 1: true })

      subject.send('selectItem', {})

      expect(
        updateSelectedItemsHashSpy.calledWith({ 1: true }, {})
      ).to.equal(true)
    })

    it('calls "notifyPropertyChange" with correct parameter', function () {
      const notifyPropertyChangeSpy = sandbox.spy(subject, 'notifyPropertyChange')
      sandbox.stub(utils, 'updateSelectedItemsHash').returns({ 1: true })

      subject.send('selectItem', {})

      expect(
        notifyPropertyChangeSpy.calledWith('selectedItems')
      ).to.equal(true)
    })
  })
})

```