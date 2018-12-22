# Homey Mocks

Mock objects that can be used during the development of custom pairing/settings templates for Homey apps.

Instead of a tedious modify-upload-test cycle, you can create mock event handlers/listeners that can be configured to react like your driver/app. That means that you can develop the pairing/settings templates mostly from the comfort of your desktop or laptop computer.

## How to use

Include the script at the top of your template:

```html
<!-- for pairing templates -->
<script src='https://cdn.staticaly.com/gh/robertklep/homey-mocks/v0.0.2/homey-pairing-mock.js'></script>

<!-- for settings templates -->
<script src='/homey.js'></script>
<script src='https://cdn.staticaly.com/gh/robertklep/homey-mocks/v0.0.2/homey-settings-mock.js'></script>
```

(or download the files and add them to your app, they're not big)

Then, use `Homey.isMock` to run code when the mock object is active:

```javascript
if (Homey.isMock) {
}
```

That's all.

You don't necessarily need to remove the includes from production code, because the scripts will not do anything if they're running in a production environment.

## Pairing API

All original `Homey` methods are supported.

However, some react differently then when run on Homey itself:
* `Homey.nextView/prevView()`: require that you called `setNextView/setPrevView` first;
* `Homey.showView(view)`: will try to open the file `${ view }.html` in your browser;
* `Homey.addDevices()`: will log the device data to console;
* `Homey.getOptions()`: the `viewId` options is not optional, options should have been set using `Homey.setOptions()` first;
* `Homey.setNavigationClose()`: does the same as `Homey.done()`;
* `Homey.done()`: calls `window.alert()`;
* `Homey.__()`: no-op;

### Mock API methods

* `Homey.isMock`

  Equals `true` when the `Homey` object is mocked.

* `Homey.registerEmitHandler(event, fn)`

  Register a function that will be called when `Homey.emit(event)` is called:

  ```javascript
  // Register a mock handler for the `start` event.
  Homey.registerEmitHandler('start', (event, data, cb) => {
    cb(null, 'Started!');
  });

  // The handler above is called when using the following code:
  Homey.emit('start', { 'foo': 'bar' }, function( err, result ) {
    console.log(result); // result is Started!
  });
  ```

* `Homey.registerOnHandler(event, fn)`

  Register a function that will be called when `Homey.on(event)` is called:

  ```javascript
  // Register a mock listener for the `hello` event:
  Homey.registerOnHandler('hello', (event, cb) => {
    cb('Hello to you!', (err, result) => {
      console.log(result); // Hi!
    });
  });

  // The listener above will be called when using the following code:
  Homey.on('hello', function(message, callback) {
    Homey.alert( message ); // Hello to you!
    callback( null, 'Hi!' ) // send something back, (err, result) style
  });
  ```

* `Homey.setZone(zoneId)`

  Set the current zoneId (for use with  `Homey.getZone()`)

* `Homey.setNextView(view)`

  Set the next view (for use with `Homey.nextView()`)

* `Homey.setPrevView(view))`

  Set the previous view (for use with `Homey.prevView()`)

* `Homey.setOptions(viewId, opts)`

  Set view options (for use with `Homey.getOptions()`)

## Settings API

All original `Homey` methods are supported.

However, some react differently then when run on Homey itself:
* `Homey.ready()`: no-op;
* `Homey.api()`: no actual requests are made, see `Homey.addRoutes()` below;
* `Homey.__()`: no-op;

### Mock API methods

* `Homey.isMock`

  Equals `true` when the `Homey` object is mocked.

* `Homey.setSettings(settings)`

  Assign application settings (to be retrieved using `Homey.get()`).

* `Homey.addRoutes(routes)`

  Set API routes to match the routes in your `api.js` files. The structure of the `routes` array is basically the same as for `api.js`, where `fn` should implement the (fake) API response.

  For example, this is how you could mock a CRUD store:

  ```javascript
  let store = {};
  Homey.addRoutes([
    {
      method: 'GET',
      path:   '/',
      fn:     function(args, callback) {
        return callback(null, Object.assign({}, store));
      }
    },
    {
      method: 'GET',
      path:   '/:id',
      fn:     function(args, callback) {
        const item = store[args.params.id];
        if (! item) {
          return callback(Error('NOT_FOUND'));
        }
        return callback(null, item);
      }
    },
    {
      method: 'POST',
      path:   '/',
      fn:     function(args, callback) {
        const id   = ('00000000' + 100000000 * Math.random()).slice(-8);
        const item = store[id] = args.body;
        item.id    = id;
        return callback(null, item);
      }
    },
    {
      method: 'PUT',
      path:   '/:id',
      fn:     function(args, callback) {
        const item = store[args.params.id];
        if (! item) {
          return callback(Error('NOT_FOUND'));
        }
        Object.assign(item, args.body);
        item.id = args.params.id;
        return callback(null, item);
      }
    },
    {
      method: 'DELETE',
      path:   '/:id',
      fn:     function(args, callback) {
        const item = store[args.params.id];
        if (! item) {
          return callback(Error('NOT_FOUND'));
        }
        delete store[args.params.id];
        return callback();
      }
    },
  ]);
  ```
