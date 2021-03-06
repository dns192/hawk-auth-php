Hawk Authentication PHP
=======================

![Version Number](https://img.shields.io/packagist/v/shawm11/hawk-auth.svg)
![PHP Version](https://img.shields.io/packagist/php-v/shawm11/hawk-auth.svg)
[![License](https://img.shields.io/github/license/shawm11/hawk-auth-php.svg)](LICENSE.md)

A PHP implementation of the 7.x version of the [**Hawk**](https://github.com/hapijs/hawk)
HTTP authentication scheme.

Table of Contents
-----------------

<!--lint disable list-item-spacing-->

- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Installation](#installation)
- [Usage Examples](#usage-examples)
  - [Server](#server)
  - [Client](#client)
- [API References](#api-references)
- [Security Considerations](#security-considerations)
- [Contributing/Development](#contributingdevelopment)
- [Versioning](#versioning)
- [License](#license)

<!--lint enable list-item-spacing-->

Getting Started
---------------

### Prerequisites

- Git 2.9+
- PHP 5.5.0+
- OpenSSL PHP Extension
- JSON PHP Extension
- [Composer](https://getcomposer.org/)

### Installation

Download and install using [Composer](https://getcomposer.org/):

```shell
composer require shawm11/hawk-auth
```

Usage Examples
--------------

The examples in this section do not work without modification. However, these
examples should be enough to demonstrate how to use this package.

### Server Example

Because PHP is a language most commonly used for server logic, the "Server"
usage is more common than the "Client" usage.

```php
<?php

use Shawm11\Hawk\Server\Server as HawkServer;
use Shawm11\Hawk\Server\ServerException as HawkServerException;
use Shawm11\Hawk\Server\BadRequestException as HawkBadRequestException;
use Shawm11\Hawk\Server\UnauthorizedException as HawkUnauthorizedException;

// A fictional function that handles an incoming request
function handleRequest() {
    $hawkServer = new HawkServer;
    $result = [];
	// Pretend to get request data from a client
	$requestData = [
		'method' => 'GET',
		'url' => '/resource/4?a=1&b=2',
		'host' => 'example.com',
		'port' => 8080,
        // Authorization header
		'authorization' => 'Hawk id="dh37fgj492je", ts="1353832234", nonce="j4h3g2", ext="some-app-ext-data", mac="6R4rV5iE+NPoym+WwjeHzjAGXUtLNIxmo1vpMofpLAE="'
	];
    // Function for retrieving credentials
    $credentialsFunc = function ($id) {
        // Pretend to retrieve the credentials (maybe from database) using the given ID ($id)
        $credentials = [
            'id' => '123456',
            'key' => 'werxhqb98rpaxn39848xrunpaw3489ruxnpa98w4rxn',
            'algorithm' => 'sha256',
            'user' => 'Steve'
        ];

        return $credentials;
    };

    try {
        $result = $hawkServer->authenticate($requestData, $credentialsFunc);
    } catch (HawkBadRequestException $e) {
        $httpStatusCode = $e->getCode();

        // Send HTTP status 400 (Bad Request) response...

        return;
    } catch (HawkUnauthorizedException $e) {
        $httpStatusCode = $e->getCode();
        // Run a fictional function that sets the header
    	setHeaderSomehow('WWW-Authenticate', $e->getWwwAuthenticateHeader());

        // Send HTTP status 401 (Unauthorized) response...

        return;
    } catch (HawkServerException $e) {
        echo 'ERROR: ' . $e->getMessage();
        return;
    }

    $credentials = $result['credentials']; // an array
    $artifacts = $result['artifacts']; // an array

    // Do some more stuff

    // Then send an authenticated response (See `sendResponse` function below)
    sendResponse($hawkServer, $credentials, $artifacts);
}

function sendResponse($hawkServer, $credentials, $artifacts) {
	$header = '';

    try {
        $header = $hawkServer->header($credentials, $artifact); // Output is a string
    } catch (HawkServerException $e) {
        echo 'ERROR: ' . $e->getMessage();
        return;
	}

	// Run a fictional function that sets the header
	setHeaderSomehow('Server-Authorization', $header);

	// Now do some other stuff to send the response
}
```

### Client Example

```php
<?php

use Shawm11\Hawk\Client\Client as HawkClient;
use Shawm11\Hawk\Client\ClientException as HawkClientException;

// A fictional function that makes an authenticated request to the server
function makeRequest($requestData) {
    $hawkClient = new HawkClient;
    $result = [];
	$uri = 'http://example.com/resource?a=b';
	$options = [
        // This is required
		'credentials' => [
			'id' => 'dh37fgj492je',
            'key' => 'aoijedoaijsdlaksjdl',
            'algorithm' => 'sha256'
		]
	];

    try {
        $result = $hawkClient->header($uri, 'POST', $options);
    } catch (HawkClientException $e) {
        echo 'ERROR: ' . $e->getMessage();
        return;
    }

    $header = $result['header']; // a string
    $artifacts = $result['artifacts']; // an array

	// Run a fictional function that sets the header
	setHeaderSomehow('Authorization', $header);

    // Do some more stuff before sending request

	// Now send the request
	sendRequestSomehow(); // Not a real function

	// Wait for response from server...

    // Now do some stuff after receiving response (See the `responseCallback` function below)
    responseCallback($hawkClient, $options['credentials'], $artifacts);
}

function responseCallback($hawkClient, $credentials, $artifacts) {
    // Somehow get the headers used in the response
	$responseHeaders = [
        // Only need these 3 headers
		'Server-Authorization' => 'some stuff',
		'WWW-Authentication' => 'some more stuff',
		'Content-Type' => 'application/json' // A different content type can be used
	];

    // Validate the server's response
    try {
        // If the server's response is valid, the parsed response headers are
        // returned as an array
        $parsedHeaders = $hawkClient->authenticate($responseHeaders, $credentials, $artifacts);
    } catch (HawkClientException $e) {
        // If the server's response is invalid, an error is thrown
        echo 'ERROR: ' . $e->getMessage();
        return;
	}

	// Now do some other stuff with the response
}
```

API References
--------------

<!--lint disable list-item-spacing-->

- [Server API](docs/api-reference/server-api.md) — API reference for the classes
  in the `Shawm11\Hawk\Server` namespace
- [Client API](docs/api-reference/server-api.md) — API reference for the classes
  in the `Shawm11\Hawk\Client` namespace
- [Utils API](docs/api-reference/utils-api.md) — API reference for the classes
  in the `Shawm11\Hawk\Utils` namespace
- [Crypto API](docs/api-reference/crypto-api.md) — API reference for the classes
  in the `Shawm11\Hawk\Crypto` namespace

<!--lint enable list-item-spacing-->

Security Considerations
-----------------------

See the [Security Considerations](https://github.com/hapijs/hawk#security-considerations)
section of Hawk's README.

Contributing/Development
------------------------

Please read [CONTRIBUTING.md](CONTRIBUTING.md) for details on coding style, Git
commit message guidelines, and other development information.

Versioning
----------

This project uses [SemVer](http://semver.org/) for versioning. For the versions
available, see the tags on this repository.

License
-------

This project is open-sourced software licensed under the [MIT license](https://opensource.org/licenses/MIT).
