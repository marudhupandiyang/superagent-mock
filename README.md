<p align="center">
<b><a href="#installation">Installation</a></b>
|
<b><a href="#usage">Usage</a></b>
|
<b><a href="#supported-methods">Supported Methods</a></b>
|
<b><a href="#tests">Tests</a></b>
|
<b><a href="#credits">Credits</a></b>
|
<b><a href="#license">License</a></b>
</p>

# superagent-mockify

[superagent](https://github.com/visionmedia/superagent) plugin allowing to simulate HTTP calls by returning data fixtures based on the requested URL with an optional delay(simulating network call).

**Note**: this plugin is developed for `superagent: ^v1.1.0`.

See [this post](http://tech.m6web.fr/how-did-we-mock-the-backend-developers.html) to know why we use superagent-mock at M6Web.

## Credits
Inspired from [superagent-mock](https://github.com/M6Web/superagent-mock) by [M6Web](https://github.com/M6Web)

## Installation

Install with [npm](http://npmjs.org/):

```sh
$ npm install superagent-mockify
```

## Usage

First, you have to define the URLs to mock in a configuration file:

```js
// ./superagent-mockify-config.js file
module.exports = [
  {
    /**
     * regular expression of URL
     */
    pattern: 'https://domain.example(.*)',

    /**
     * The amount of time that should be delayed to simulate the dealy in network call.
     * Time is in milliseconds
     */
    delayResponse: 1000,

    /**
     * returns the data
     *
     * @param match array Result of the resolution of the regular expression
     * @param params object sent by 'send' function
     * @param headers object set by 'set' function
     */
    fixtures: function (match, params, headers) {
      /**
       * Returning error codes example:
       *   request.get('https://domain.example/404').end(function(err, res){
       *     console.log(err); // 404
       *     console.log(res.notFound); // true
       *   })
       */
      if (match[1] === '/404') {
        throw new Error(404);
      }

      /**
       * Checking on parameters example:
       *   request.get('https://domain.example/hero').send({superhero: "superman"}).end(function(err, res){
       *     console.log(res.body); // "Your hero: superman"
       *   })
       */

      if (match[1] === '/hero') {
        if(params['superhero']) {
          return 'Your hero:' + params['superhero'];
        } else {
          return 'You didnt chose a hero';
        }
      }


      /**
       * Checking on headers example:
       *   request.get('https://domain.example/authorized_endpoint').set({Authorization: "9382hfih1834h"}).end(function(err, res){
       *     console.log(res.body); // "Authenticated!"
       *   })
       */

      if (match[1] === '/authorized_endpoint') {
        if(headers['Authorization']) {
          return 'Authenticated!';
        } else {
          throw new Error(401); // Unauthorized
        }
      }

    },

    /**
     * returns the result of the GET request
     *
     * @param match array Result of the resolution of the regular expression
     * @param data  mixed Data returns by `fixtures` attribute
     */
    get: function (match, data) {
      return {
        body: data
      };
    },

    /**
     * returns the result of the POST request
     *
     * @param match array Result of the resolution of the regular expression
     * @param data  mixed Data returns by `fixtures` attribute
     */
    post: function (match, data) {
      return {
        code: 201
      };
    }
  },
  ...
];
```

Then use the plugin:

```js
// ./server.js file
var request = require('superagent');
var config = require('./superagent-mockify-config');

// Before tests
var superagentMockify = require('superagent-mockify')(request, config);

...

// After tests
superagentMockify.unset();
```

## Supported methods

All request methods are supported (get, put, post, etc.).

Each request method mock have to be declared in the config file. Otherwise, the `callback` method is used.

## Logging

You can monitor each call, that has been intercepted by superagent-mockify or not, by passing a callback function at initialization.

``` js
// ./server.js file
var request = require('superagent');
var config = require('./superagent-mockify-config');

var logger = function(log)  {
  console.log('superagent call', log);
};

// Before tests
var superagentMock = require('superagent-mockify')(request, config, logger);

...

// After tests
superagentMock.unset();
```

The callback function will be called with an object containing the following informations
 - data : data used with `superagent.send` function
 - headers : array of headers given by `superagent.set` function
 - matcher : regex matching the current url which is defined in the provided config
 - url : url which superagent was called
 - method : HTTP method used for the call
 - timestamp : timestamp of the superagent call
 - mocked : true if the call was mocked by superagent mock, false if it used superagent real methods

## Tests

To run units tests: `npm test`.

To check code style: `npm run lint`.


## Credits

Developped by the [Cytron Team](http://cytron.fr/) of [M6 Web](http://tech.m6web.fr/).
Tested with [nodeunit](https://github.com/caolan/nodeunit).

## License

superagent-mockify is licensed under the [MIT license](LICENSE).
