# Recipe for: An acceptance test using hooks (ember-hook), page objects (ember-cli-page-object) and data mocking (ember-data-factory-guy)

## The Page Objects for a Users Page (pages/users.js)

```javascript
import {clickable, count, create, visitable} from 'ember-cli-page-object'
import {hook} from 'ember-hook'

export default create({
  submit: clickable(hook('createButton')),
  userCount: count(hook('userRecord')),
  visit: visitable('/users')
})
```

## The User Model (user.js)

```javascript
import DS from 'ember-data'
const {attr, belongsTo, hasMany, Model} = DS

export default Model.extend({
  dateJoined: attr('date'),
  directory: attr('string'),
  description: attr('string'),
  email: attr('string'),
  firstName: attr('string'),
  isActive: attr('boolean'),
  isInternal: attr('boolean'),
  lastLogin: attr('date'),
  lastName: attr('string'),
  roles: hasMany('role'),
  tenant: belongsTo('tenant'),
  username: attr('string')
})
```

## The ember-data-factory-guy factory user.js for the user model

```javascript
// tests/dummy/app/tests/factories/user.js

import FactoryGuy from 'ember-data-factory-guy'

FactoryGuy.define('user', {
  sequences: {
    directory: (num) => {
      let items = ['Local', 'LDAP', 'RADIUS']
      return items[num % items.length]
    }
  },

  default: {
    dateJoined: () => faker.date.past(),
    directory: FactoryGuy.generate('directory'),
    description: () => faker.lorem.sentence(),
    email: () => faker.internet.email(),
    firstName: () => faker.name.firstName(),
    isActive: () => faker.random.boolean(),
    isInternal: () => faker.random.boolean(),
    lastLoginDetail: {
      ipAddress: () => {
        return faker.internet.ip()
      },
      time: () => {
        return faker.date.past()
      }
    },
    lastName: () => faker.name.lastName(),
    roles: FactoryGuy.hasMany('role'),
    tenant: FactoryGuy.belongsTo('tenant'),
    tokenExpirationTime: null,
    usergroups: [],
    username: () => faker.internet.userName()
  }
})
```

## The Role Model (role.js)

```javascript
import DS from 'ember-data'
const {attr, Model} = DS

export default Model.extend({
  createdTime: attr('date'),
  description: attr('string'),
  displayName: attr('string'),
  isInternal: attr('boolean'),
  modifiedTime: attr('date'),
  name: attr('string')
})
```

## The ember-data-factory-guy factory role.js for the role model

```javascript
// tests/dummy/app/tests/factories/role.js

import FactoryGuy from 'ember-data-factory-guy'

FactoryGuy.define('role', {
  sequences: {
    name: (num) => {
      let items = ['app', 'uacuiapp', 'admin', 'sysadmin', 'user']
      return items[num % items.length]
    }
  },

  default: {
    name: FactoryGuy.generate('name')
  },

  role_all_attrs: {
    application: () => faker.random.uuid(),
    createdTime: () => faker.date.past(),
    description: () => faker.lorem.sentences(),
    displayName: FactoryGuy.generate('name'),
    isInternal: () => faker.random.boolean(),
    modifiedTime: () => faker.date.past(),
    name: FactoryGuy.generate('name'),
    parents: []
  }
})
```

## The Tenant Model (tenant.js)

```javascript
import DS from 'ember-data'
const {attr, Model} = DS

export default Model.extend({
  createdTime: attr('date'),
  description: attr('string'),
  displayName: attr('string'),
  isActive: attr('boolean'),
  isMaster: attr('boolean'),
  modifiedTime: attr('date'),
  name: attr('string')
})
```

## The ember-data-factory-guy factory tenant.js for the tenant model

```javascript
// tests/dummy/app/tests/factories/tenant.js

import FactoryGuy from 'ember-data-factory-guy'

FactoryGuy.define('tenant', {
  default: {
  },

  tenant_all_attrs: {
    createdTime: () => faker.date.past(),
    description: () => faker.lorem.sentences(),
    displayName: () => faker.commerce.productName(),
    isActive: () => faker.random.boolean(),
    isMaster: () => faker.random.boolean(),
    modifiedTime: () => faker.date.past(),
    name: () => faker.commerce.productName(),
    parent: null
  }
})
```

## The users.hbs template

```handlebars
// Users Route

<ul>
  {{#each users as |record|}}
    <li data-test={{hook 'userRecord'}}>
      Username: {{record.username}} |
      First Name: {{record.firstName}} |
      Last Name: {{record.lastName}} |
      Email: {{record.email}} |
      Id: {{record.id}} |
      Description: {{record.description}}
    </li>
    <br>
  {{else}}
    No records to show
  {{/each}}
</ul>

{{frost-button
  hook='createButton'
  design='info-bar'
  text='Create user'
  onClick=(action 'createUser')
}}
```

## The Acceptance Test for Users Route

```javascript
import {expect} from 'chai'
import {mockCreate, mockFindAll, mockSetup, mockTeardown} from 'ember-data-factory-guy'
import {describe, it, beforeEach, afterEach} from 'mocha'
import destroyApp from '../helpers/destroy-app'
import startApp from '../helpers/start-app'
import page from '../pages/users'

describe('Acceptance: Users', function () {
  let application

  beforeEach(function () {
    application = startApp()
    // Adding FactoryGuy mockSetup call
    mockSetup()
  })

  afterEach(function () {
    destroyApp(application)
    // Adding FactoryGuy mockTeardown call
    mockTeardown()
  })

  describe('Page shows users', function () {
    beforeEach(function () {
      mockFindAll('user', 3)
      mockFindAll('role', 5)
      mockFindAll('tenant', 5)

      page.visit()
    })

    it('has 3 users', function () {
      return andThen(() => {
        expect(page.userCount).to.equal(3)
      })
    })
  })

  describe('Can create a new user', function () {
    beforeEach(function () {
      mockCreate('user')
      mockFindAll('user')
      mockFindAll('role', 5)
      mockFindAll('tenant', 5)

      page.visit()
        .submit()
    })

    it('creates a user', function () {
      return andThen(() => {
        expect(page.userCount).to.equal(1)
      })
    })
  })
})
```