# php-base - PHP Slim 4 Server library for IDaaS API

* [OpenAPI Generator](https://openapi-generator.tech)
* [Slim 4 Documentation](https://www.slimframework.com/docs/v4/)

This server has been generated with [Slim PSR-7](https://github.com/slimphp/Slim-Psr7) implementation.
[PHP-DI](https://php-di.org/doc/frameworks/slim.html) package used as dependency container.

## Requirements

* Web server with URL rewriting
* PHP 8.1 or newer

This package contains `.htaccess` for Apache configuration.
If you use another server(Nginx, HHVM, IIS, lighttpd) check out [Web Servers](https://www.slimframework.com/docs/v3/start/web-servers.html) doc.

## Installation via [Composer](https://getcomposer.org/)

Navigate into your project's root directory and execute the bash command shown below.
This command downloads the Slim Framework and its third-party dependencies into your project's `vendor/` directory.
```bash
$ composer install
```

## Add configs

[PHP-DI package](https://php-di.org/doc/getting-started.html) helps to decouple configuration from implementation. App loads configuration files in straight order(`$env` can be `prod` or `dev`):
1. `config/$env/default.inc.php` (contains safe values, can be committed to vcs)
2. `config/$env/config.inc.php` (user config, excluded from vcs, can contain sensitive values, passwords etc.)
3. `lib/App/RegisterDependencies.php`

## Start devserver

Run the following command in terminal to start localhost web server, assuming `./php-slim-server/public/` is public-accessible directory with `index.php` file:
```bash
$ php -S localhost:8888 -t php-slim-server/public
```
> **Warning** This web server was designed to aid application development.
> It may also be useful for testing purposes or for application demonstrations that are run in controlled environments.
> It is not intended to be a full-featured web server. It should not be used on a public network.

## Tests

### PHPUnit

This package uses PHPUnit 8 or 9(depends from your PHP version) for unit testing.
[Test folder](tests) contains templates which you can fill with real test assertions.
How to write tests read at [2. Writing Tests for PHPUnit - PHPUnit 8.5 Manual](https://phpunit.readthedocs.io/en/8.5/writing-tests-for-phpunit.html).

#### Run

Command | Target
---- | ----
`$ composer test` | All tests
`$ composer test-apis` | Apis tests
`$ composer test-models` | Models tests

#### Config

Package contains fully functional config `./phpunit.xml.dist` file. Create `./phpunit.xml` in root folder to override it.

Quote from [3. The Command-Line Test Runner — PHPUnit 8.5 Manual](https://phpunit.readthedocs.io/en/8.5/textui.html#command-line-options):

> If phpunit.xml or phpunit.xml.dist (in that order) exist in the current working directory and --configuration is not used, the configuration will be automatically read from that file.

### PHP CodeSniffer

[PHP CodeSniffer Documentation](https://github.com/squizlabs/PHP_CodeSniffer/wiki). This tool helps to follow coding style and avoid common PHP coding mistakes.

#### Run

```bash
$ composer phpcs
```

#### Config

Package contains fully functional config `./phpcs.xml.dist` file. It checks source code against PSR-1 and PSR-2 coding standards.
Create `./phpcs.xml` in root folder to override it. More info at [Using a Default Configuration File](https://github.com/squizlabs/PHP_CodeSniffer/wiki/Advanced-Usage#using-a-default-configuration-file)

### PHPLint

[PHPLint Documentation](https://github.com/overtrue/phplint). Checks PHP syntax only.

#### Run

```bash
$ composer phplint
```

## Show errors

Switch your app environment to development
- When using with some webserver => in `public/.htaccess` file:
```ini
## .htaccess
<IfModule mod_env.c>
    SetEnv APP_ENV 'development'
</IfModule>
```

- Or when using whatever else, set `APP_ENV` environment variable like this:
```bash
export APP_ENV=development
```
or simply
```bash
export APP_ENV=dev
```

## Mock Server
Since this feature should be used for development only, change environment to `development` and send additional HTTP header `X-Mydex\ApiIdaasSdk\Slim4-Mock: ping` with any request to get mocked response.
CURL example:
```console
curl --request GET \
    --url 'http://localhost:8888/v2/pet/findByStatus?status=available' \
    --header 'accept: application/json' \
    --header 'X-Mydex\ApiIdaasSdk\Slim4-Mock: ping'
[{"id":-8738629417578509312,"category":{"id":-4162503862215270400,"name":"Lorem ipsum dol"},"name":"Lorem ipsum dolor sit amet, consectetur adipiscing elit. Lorem i","photoUrls":["Lor"],"tags":[{"id":-3506202845849391104,"name":"Lorem ipsum dolor sit amet, consectetur adipiscing elit. Lorem ipsum dolor sit amet, consectet"}],"status":"pending"}]
```

Used packages:
* [Openapi Data Mocker](https://github.com/ybelenko/openapi-data-mocker) - first implementation of OAS3 fake data generator.
* [Openapi Data Mocker Server Middleware](https://github.com/ybelenko/openapi-data-mocker-server-middleware) - PSR-15 HTTP server middleware.
* [Openapi Data Mocker Interfaces](https://github.com/ybelenko/openapi-data-mocker-interfaces) - package with mocking interfaces.

## Logging

Build contains pre-configured [`monolog/monolog`](https://github.com/Seldaek/monolog) package. Make sure that `logs` folder is writable.
Add required log handlers/processors/formatters in `lib/App/RegisterDependencies.php`.

## API Endpoints

All URIs are relative to *https://idp.mydexid.org*

> Important! Do not modify abstract API controllers directly! Instead extend them by implementation classes like:

```php
// src/Api/PetApi.php

namespace Mydex\ApiIdaasSdk\Slim4\Api;

use Mydex\ApiIdaasSdk\Slim4\Api\AbstractPetApi;
use Psr\Http\Message\ServerRequestInterface;
use Psr\Http\Message\ResponseInterface;

class PetApi extends AbstractPetApi
{
    public function addPet(
        ServerRequestInterface $request,
        ResponseInterface $response
    ): ResponseInterface {
        // your implementation of addPet method here
    }
}
```

When you need to inject dependencies into API controller check [PHP-DI - Controllers as services](https://github.com/PHP-DI/Slim-Bridge#controllers-as-services) guide.

Place all your implementation classes in `./src` folder accordingly.
For instance, when abstract class located at `./lib/Api/AbstractPetApi.php` you need to create implementation class at `./src/Api/PetApi.php`.

Class | Method | HTTP request | Description
------------ | ------------- | ------------- | -------------
*AbstractConnectionsApi* | **postFtcSetup** | **POST** /ftc/setup | Set up a First Time Connection URL.
*AbstractConnectionsApi* | **postIdentify** | **POST** /identify | Set up an identification URL.
*AbstractIdentifiersApi* | **patchNotifyStatus** | **PATCH** /members/notify/status | Update the status of a notification.
*AbstractIdentifiersApi* | **postInviteStatus** | **POST** /members/invite/status | Check the status of an invite.
*AbstractIdentifiersApi* | **postNotifyMemberViaIdentifier** | **POST** /members/notify | Send a notification to a member via its identifier.
*AbstractIdentifiersApi* | **postStoreAndSendIdentifierInvite** | **POST** /members/invite | Store an invitation code and send notification.
*AbstractIdentifiersApi* | **getIdentifierInvite** | **GET** /members/invite/{invite} | Fetch an invitation code and identifier auth data.
*AbstractMyAccountApi* | **getMydexIdFromMyAccountSvt** | **GET** /members/myaccount/{svt} | Check if a MyAccount SVT has an associated MydexID.
*AbstractScotAccountApi* | **getMydexIdFromScotAccountSub** | **GET** /members/scotaccount/{sub} | Check if a ScotAccount sub GUID has an associated MydexID.


## Models

* Mydex\ApiIdaasSdk\Slim4\Model\AuthErrorResponse
* Mydex\ApiIdaasSdk\Slim4\Model\AuthErrorResponseError
* Mydex\ApiIdaasSdk\Slim4\Model\CreateIdentifierResponse
* Mydex\ApiIdaasSdk\Slim4\Model\ErrorResponse
* Mydex\ApiIdaasSdk\Slim4\Model\FtcIdentifyUrlResponse
* Mydex\ApiIdaasSdk\Slim4\Model\FtcSetupRequestBody
* Mydex\ApiIdaasSdk\Slim4\Model\FtcUrlResponse
* Mydex\ApiIdaasSdk\Slim4\Model\GetIdentifiersResponse
* Mydex\ApiIdaasSdk\Slim4\Model\GetInviteResponse
* Mydex\ApiIdaasSdk\Slim4\Model\IdentifierAuthObject
* Mydex\ApiIdaasSdk\Slim4\Model\IdentifierObject
* Mydex\ApiIdaasSdk\Slim4\Model\IdentifyRequestBody
* Mydex\ApiIdaasSdk\Slim4\Model\IdentifyUrlResponse
* Mydex\ApiIdaasSdk\Slim4\Model\InviteRequestBody
* Mydex\ApiIdaasSdk\Slim4\Model\InviteStatusRequestBody
* Mydex\ApiIdaasSdk\Slim4\Model\InviteStatusResponse
* Mydex\ApiIdaasSdk\Slim4\Model\NotifyMemberRequestBody
* Mydex\ApiIdaasSdk\Slim4\Model\UpdateNotificationStatusRequestBody


## Authentication

### Security schema `oauth2`
> Important! To make OAuth authentication work you need to extend [\Mydex\ApiIdaasSdk\Slim4\Auth\AbstractAuthenticator](./lib/Auth/AbstractAuthenticator.php) class by [\Mydex\ApiIdaasSdk\Slim4\Auth\OAuthAuthenticator](./src/Auth/OAuthAuthenticator.php) class.

Scope list:
* `mydex:identifiers` - Access to identifiers endpoints
* `mydex:scotaccount` - Access to ScotAccount-to-MydexID mapping
* `mydex:myaccount` - Access to MyAccount-to-MydexID mapping
* `mydex:pdx` - Global access scope for connections endpoints

### Advanced middleware configuration
Ref to used Slim Token Middleware [dyorg/slim-token-authentication](https://github.com/dyorg/slim-token-authentication/tree/1.x#readme)
