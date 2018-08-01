
## Testing in Nodejs
An example and discussion about unit and integration testing in node.

---

## Testing options

Option 1 | Option 2
--- | ---
Mocha | Jest
Chai | 
Sinon |
Mockery |
Istanbul (nyc) |

---

![jest logo](https://raw.githubusercontent.com/matthew-wasbrough-m3/jest-presentation/master/img/jest-logo.png)

## Jest

https://jest.io

```javascript
npm install --save-dev jest
```

```javascript
yarn add -D jest
```

---

## Community

https://github.com/jest-community

* jest-extended
* jest-chain
* jest-junit

---

## Previous Testing

```javascript
it('creates a new targeting group', function(done) {
  var newTargetingGroupId = 4;
  var contentId = 2;
  dbMock.deleteContentTargeting.returns(Promise.resolve());
  dbMock.findExistingTargetingGroup.returns(Promise.resolve(null));
  dbMock.createTargetingGroup.returns(Promise.resolve(4));
  dbMock.linkProfileAttributesToTargetingGroup.returns(Promise.resolve());

  testObj.addContentTargeting(contentId, [{profileAttributeIds: [1,2,3]}])
    .then(function() {
      assert.ok(dbMock.createTargetingGroup.called);
      done();
    });
});
```

+++

@title[Code being tested]

```javascript
const addContentTargeting = (contentId, targetingGroups) => {
  log.debug(`Add content targeting: contentId=${contentId} ` +
    `targetingGroups=${JSON.stringify(targetingGroups)}`);
  return Promise.map(targetingGroups, group =>
    db.transaction(client =>
      contentDb
        .findExistingTargetingGroup(client, group.profileAttributeIds)
        .then((id) => {
          if (!id) {
            return contentDb
              .createTargetingGroup(client)
              .then(tgId =>
                contentDb
                  .linkProfileAttributesToTargetingGroup(client, tgId, group.profileAttributeIds)
                .then(() => tgId));
          }
          return id;
        }), { concurrency: 20 }))
      .then((targetingGroupIds) => {
        log.debug('TARGETING GROUP IDS', targetingGroupIds);
        return db.transaction(client =>
          contentDb
            .deleteContentTargeting(client, contentId)
            .then(() =>
              contentDb
                .linkTargetingGroupsToContent(client, contentId, targetingGroupIds)));
      }).catch((err) => {
        log.error('Error during addContentTargeting', err);
        throw err;
      });
};
```

Note:
- deleteContentTargeting.returns(Promise.resolve());
- findExistingTargetingGroup.returns(Promise.resolve(null));
- createTargetingGroup.returns(Promise.resolve(4));
- linkProfileAttributesToTargetingGroup.returns(Promise.resolve()); 

---

## Unit Testing

* Unit testing functions is good.
* Jest is as good, if not better, than Mocha.

Note:
- not going to talk about it here
- Jest is better than mocha because it parallises the tests for speed.
- Jest can do as much as Mocha, Chai, Sinon, Mockery, Istanbul put together.

---

## Integration testing a service

m3.intl.eddie-service

```javascript
test('resquest for 0 bookmarks returns all bookmarks', () =>
  request(app)
    .get('/recommendation/community1/user1/0')
    .then((res) => {
      expect(res.status).toBe(200);
      expect(res.body).toBeArrayOfSize(6);
    }));
```

Note:
- using supertest
- toBeArrayOfSize() - jest-extended
- setup test data so I know I should be expecting an array of 6
- request(app) vs running server (see slide below)

+++

@title[Old style test]

from m3.intl.deep-thought

```javascript
request = request('http://localhost:3009');
it('should return 200', function(done){
  request
    .get('/healthcheck')
    .expect(200, done);
})
```

The server must be started in order to perform the test.

Note:
- calling done is an artefact of callbacks instead of promises

---

## A simple express server

```javascript
// server.js
import express from 'express';
import healthcheck from './routes/healthcheck';
const app = express();
app.use('/healthcheck');
app.listen(3000, () => log.debug(`server started on port 3000`));
```

---

## Refactored

If we refactor this into two files:

```javascript
//app.js
import express from 'express';
import healthcheck from './routes/healthcheck';
const app = express();
app.use('/healthcheck');
module.exports = app;
```

```javascript
//server.js
import app from './app';
app.listen(3000, () => log.debug(`server started on port 3000`));
```

Note:
- don't need to start up a server (or shut it down)
- supertest can handle it.

---

## setup and teardown the db

---

## snapshot testing

also using jest-chain

```javascript
test('request of all recommended bookmarks returns an array of objects', () =>
  request(app)
    .get('/recommendation/community1/user1/')
    .then((res) => {
      const isObj = el => Object.prototype.toString.call(el) === '[object Object]';
      const containsKeys = el => Object.keys(el).length === 3 && el.title && el.url && el.id;
      expect(res.status).toBe(200);
      expect(res.body)
        .toBeArray()
        .toSatisfyAll(isObj)
        .toSatisfyAll(containsKeys)
        .toMatchSnapshot();
    }));
```

---

## package.json

config can be moved to an external file which we could then import...
yarn test

---

## test coverage

---

## circle integration

---

## personal-data-controller

---

## react-testing-library