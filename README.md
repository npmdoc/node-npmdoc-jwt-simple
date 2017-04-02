# api documentation for  [jwt-simple (v0.5.1)](https://github.com/hokaccha/node-jwt-simple#readme)  [![npm package](https://img.shields.io/npm/v/npmdoc-jwt-simple.svg?style=flat-square)](https://www.npmjs.org/package/npmdoc-jwt-simple) [![travis-ci.org build-status](https://api.travis-ci.org/npmdoc/node-npmdoc-jwt-simple.svg)](https://travis-ci.org/npmdoc/node-npmdoc-jwt-simple)
#### JWT(JSON Web Token) encode and decode module

[![NPM](https://nodei.co/npm/jwt-simple.png?downloads=true)](https://www.npmjs.com/package/jwt-simple)

[![apidoc](https://npmdoc.github.io/node-npmdoc-jwt-simple/build/screen-capture.buildNpmdoc.browser._2Fhome_2Ftravis_2Fbuild_2Fnpmdoc_2Fnode-npmdoc-jwt-simple_2Ftmp_2Fbuild_2Fapidoc.html.png)](https://npmdoc.github.io/node-npmdoc-jwt-simple/build..beta..travis-ci.org/apidoc.html)

![package-listing](https://npmdoc.github.io/node-npmdoc-jwt-simple/build/screen-capture.npmPackageListing.svg)



# package.json

```json

{
    "author": {
        "name": "Kazuhito Hokamura",
        "email": "k.hokamura@gmail.com"
    },
    "bugs": {
        "url": "https://github.com/hokaccha/node-jwt-simple/issues"
    },
    "dependencies": {},
    "description": "JWT(JSON Web Token) encode and decode module",
    "devDependencies": {
        "expect.js": "^0.3.1",
        "istanbul": "^0.4.2",
        "mocha": "^2.3.4"
    },
    "directories": {},
    "dist": {
        "shasum": "79ea01891b61de6b68e13e67c0b4b5bda937b294",
        "tarball": "https://registry.npmjs.org/jwt-simple/-/jwt-simple-0.5.1.tgz"
    },
    "engines": {
        "node": ">= 0.4.0"
    },
    "gitHead": "c988eb232cb5ba3fd3055032cdc4f39b727f0a0e",
    "homepage": "https://github.com/hokaccha/node-jwt-simple#readme",
    "keywords": [
        "jwt",
        "encode",
        "decode"
    ],
    "license": "MIT",
    "main": "./index",
    "maintainers": [
        {
            "name": "hokaccha",
            "email": "k.hokamura@gmail.com"
        }
    ],
    "name": "jwt-simple",
    "optionalDependencies": {},
    "readme": "ERROR: No README data found!",
    "repository": {
        "type": "git",
        "url": "git+ssh://git@github.com/hokaccha/node-jwt-simple.git"
    },
    "scripts": {
        "test": "istanbul cover _mocha test/*.js"
    },
    "version": "0.5.1"
}
```



# <a name="apidoc.tableOfContents"></a>[table of contents](#apidoc.tableOfContents)

#### [module jwt-simple](#apidoc.module.jwt-simple)
1.  [function <span class="apidocSignatureSpan">jwt-simple.</span>decode (token, key, noVerify, algorithm)](#apidoc.element.jwt-simple.decode)
1.  [function <span class="apidocSignatureSpan">jwt-simple.</span>encode (payload, key, algorithm, options)](#apidoc.element.jwt-simple.encode)
1.  string <span class="apidocSignatureSpan">jwt-simple.</span>version



# <a name="apidoc.module.jwt-simple"></a>[module jwt-simple](#apidoc.module.jwt-simple)

#### <a name="apidoc.element.jwt-simple.decode"></a>[function <span class="apidocSignatureSpan">jwt-simple.</span>decode (token, key, noVerify, algorithm)](#apidoc.element.jwt-simple.decode)
- description and source-code
```javascript
function jwt_decode(token, key, noVerify, algorithm) {
  // check token
  if (!token) {
    throw new Error('No token supplied');
  }
  // check segments
  var segments = token.split('.');
  if (segments.length !== 3) {
    throw new Error('Not enough or too many segments');
  }

  // All segment should be base64
  var headerSeg = segments[0];
  var payloadSeg = segments[1];
  var signatureSeg = segments[2];

  // base64 decode and parse JSON
  var header = JSON.parse(base64urlDecode(headerSeg));
  var payload = JSON.parse(base64urlDecode(payloadSeg));

  if (!noVerify) {
    var signingMethod = algorithmMap[algorithm || header.alg];
    var signingType = typeMap[algorithm || header.alg];
    if (!signingMethod || !signingType) {
      throw new Error('Algorithm not supported');
    }

    // verify signature. 'sign' will return base64 string.
    var signingInput = [headerSeg, payloadSeg].join('.');
    if (!verify(signingInput, key, signingMethod, signingType, signatureSeg)) {
      throw new Error('Signature verification failed');
    }

    // Support for nbf and exp claims.
    // According to the RFC, they should be in seconds.
    if (payload.nbf && Date.now() < payload.nbf*1000) {
      throw new Error('Token not yet active');
    }

    if (payload.exp && Date.now() > payload.exp*1000) {
      throw new Error('Token expired');
    }
  }

  return payload;
}
```
- example usage
```shell
...
// HS256 secrets are typically 128-bit random strings, for example hex-encoded:
// var secret = Buffer.from('fe1a1915a379f3be5394b64d14794932', 'hex)

// encode
var token = jwt.encode(payload, secret);

// decode
var decoded = jwt.decode(token, secret);
console.log(decoded); //=> { foo: 'bar' }
'''

### decode params

'''javascript
/*
...
```

#### <a name="apidoc.element.jwt-simple.encode"></a>[function <span class="apidocSignatureSpan">jwt-simple.</span>encode (payload, key, algorithm, options)](#apidoc.element.jwt-simple.encode)
- description and source-code
```javascript
function jwt_encode(payload, key, algorithm, options) {
  // Check key
  if (!key) {
    throw new Error('Require key');
  }

  // Check algorithm, default is HS256
  if (!algorithm) {
    algorithm = 'HS256';
  }

  var signingMethod = algorithmMap[algorithm];
  var signingType = typeMap[algorithm];
  if (!signingMethod || !signingType) {
    throw new Error('Algorithm not supported');
  }

  // header, typ is fixed value.
  var header = { typ: 'JWT', alg: algorithm };
  if (options && options.header) {
    assignProperties(header, options.header);
  }

  // create segments, all segments should be base64 string
  var segments = [];
  segments.push(base64urlEncode(JSON.stringify(header)));
  segments.push(base64urlEncode(JSON.stringify(payload)));
  segments.push(sign(segments.join('.'), key, signingMethod, signingType));

  return segments.join('.');
}
```
- example usage
```shell
...
var payload = { foo: 'bar' };
var secret = 'xxx';

// HS256 secrets are typically 128-bit random strings, for example hex-encoded:
// var secret = Buffer.from('fe1a1915a379f3be5394b64d14794932', 'hex)

// encode
var token = jwt.encode(payload, secret);

// decode
var decoded = jwt.decode(token, secret);
console.log(decoded); //=> { foo: 'bar' }
'''

### decode params
...
```



# misc
- this document was created with [utility2](https://github.com/kaizhu256/node-utility2)
