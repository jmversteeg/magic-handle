# worq
[![Build Status][travis-image]][travis-url]
[![dependency Status][david-image]][david-url]
[![devDependency Status][david-dev-image]][david-dev-url]
[![Code Coverage][coveralls-image]][coveralls-url]
[![NPM Version][npm-image]][npm-url]

Promise-based job queue that opens on demand and closes after a specified interval of inactivity

## Install

```bash
$ npm install --save worq
```

## Example usage

```js
// This simple example demonstrates basic usage of the Worqer API.
// Real-life use cases would often be dealing with a remote connection such as SSH

var Worqer = require('worq'),
    Q      = require('q');

// Some helper code to clarify the output log
var start              = new Date().getTime(),
    logWithElapsedTime = function (msg) {
        var now = new Date();
        now.setTime(now.getTime() - start);
        console.log('[' + now.toLocaleTimeString().substring(3, 7) + '] ' + msg);
    };

// Declare the Worqer
var handle = new Worqer(
    function () {
        logWithElapsedTime('opening');
        return Q.delay(1000).then(function () {
            logWithElapsedTime('open');
        });
    },
    function (data, threadNo) {
        logWithElapsedTime('Performing lengthy operations on ' + data + 
        ' (thread ' + threadNo + ')');
        return Q.delay(1000).then(function () {
            return data.toUpperCase();
        });
    },
    function () {
        logWithElapsedTime('closing');
        return Q.delay(1000).then(function () {
            logWithElapsedTime('closed');
        });
    }, {
        concurrency: 2,
        timeout:     3000
    });

//Queue some jobs
['foobar', 'foobaz', 'barbaz'].forEach(function (sample) {
    handle.process(sample).then(function (result) {
        logWithElapsedTime('The result of ' + sample + ': ' + result);
    });
});

handle.close(true);
// Gracefully closes the handle (waits for the job queue to finish processing, then closes)
// The handle would have automatically closed after the predefined timeout of three seconds
// if the close method had not been invoked manually.
```

The output of above code:

```
[0:00] opening
[0:01] open
[0:01] Performing lengthy operations on foobar (thread 0)
[0:01] Performing lengthy operations on foobaz (thread 1)
[0:02] The result of foobar: FOOBAR
[0:02] Performing lengthy operations on barbaz (thread 0)
[0:02] The result of foobaz: FOOBAZ
[0:03] The result of barbaz: BARBAZ
[0:03] closing
[0:04] closed
```

## License

MIT © JM Versteeg

[travis-image]: https://img.shields.io/travis/jmversteeg/worq.svg?style=flat-square
[travis-url]: https://travis-ci.org/jmversteeg/worq

[david-image]: https://img.shields.io/david/jmversteeg/worq.svg?style=flat-square
[david-url]: https://david-dm.org/jmversteeg/worq

[david-dev-image]: https://img.shields.io/david/dev/jmversteeg/worq.svg?style=flat-square
[david-dev-url]: https://david-dm.org/jmversteeg/worq#info=devDependencies

[coveralls-image]: https://img.shields.io/coveralls/jmversteeg/worq.svg?style=flat-square
[coveralls-url]: https://coveralls.io/r/jmversteeg/worq

[npm-image]: https://img.shields.io/npm/v/worq.svg?style=flat-square
[npm-url]: https://www.npmjs.com/package/worq