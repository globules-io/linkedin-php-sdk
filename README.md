## LinkedIn PHP SDK

## Installation

You will need at least PHP 7.4. Compatible with PHP 8.x

Use [composer](https://getcomposer.org/) package manager to install the lastest version of the package:

```bash
composer require globules-io/linkedin-php-sdk
```

Or add this package as dependency to `composer.json`.



## Get Started

Before you will get started, play visit to [LinkedIn API Documentation](https://docs.microsoft.com/en-us/linkedin/marketing/getting-started).
This will save you a lot of time and prevent some silly questions. To start working with LinkedIn API, you will need to get application client id and secret.

Go to [LinkedIn Developers portal](https://www.linkedin.com/developers/)
and create new application in section My Apps. Once your app has been approved, you will get a ClientId and ClientSecret, that you will use later.

#### LinkedIn Restrictions

The scopes or permissions aren't available right away when you register an application with LinkedIn. 
When you first create your app in the developer portal, you only have access to the `w_member_social` scope.
You can start developing but the token you obtain at this stage will have no refresh information.
Then you need to request access to the `Marketing Developer Platform` which will give you 2 additionals scopes, `r_emailaddress`, `r_liteprofile`
Once your request has been approved, you will get access to all the remaining scopes and your token will have refresh information.


#### Bootstrapping autoloader and instantiating a client


```php
// ... please, add composer autoloader first
include_once __DIR__ . DIRECTORY_SEPARATOR . 'vendor' . DIRECTORY_SEPARATOR . 'autoload.php';

// import client class
use LinkedIn\Client;

// instantiate the Linkedin client
$client = new Client(
    'YOUR_LINKEDIN_APP_CLIENT_ID',
    'YOUR_LINKEDIN_APP_CLIENT_SECRET'
);
```

#### Getting local redirect URL

To start linking process you have to setup redirect url.
You can set your own or use current one.
SDK provides you a `getRedirectUrl()` helper for your convenience:

```php
$redirectUrl = $client->getRedirectUrl();
```

We recommend you to have it stored during the linking session
because you will need to use it when you will be getting access token.

#### Setting local redirect URL

Set a custom redirect url use:

```php
$client->setRedirectUrl('http://your.domain.tld/path/to/script/');
```

#### Getting LinkedIn redirect URL

In order of performing OAUTH 2.0 flow, you should get LinkedIn login URL.
During this procedure you have to define scope of requested permissions.

You can read more about Linkedin Api scopes [here](https://docs.microsoft.com/en-us/linkedin/shared/references/migrations/default-scopes-migration).

Use `Scope` enum class to get scope names.
To get redirect url to LinkedIn, use the following approach:

```php
use LinkedIn\Scope;

// define scope
$scopes = [
  Scope::READ_LITE_PROFILE,
  Scope::READ_EMAIL_ADDRESS,
  Scope::SHARE_AS_USER,
  Scope::SHARE_AS_ORGANIZATION,
];
$loginUrl = $client->getLoginUrl($scopes); // get url on LinkedIn to start linking
```

Now you can take user to LinkedIn. You can use link or rely on Location HTTP header.

#### Getting Access Token

To get access token use (don't forget to set redirect url)

```php
$accessToken = $client->getAccessToken($_GET['code']);
```
This method returns object of `LinkedIn\AccessToken` class.
You can store this token in the file like this:
```php
file_put_contents('token.json', json_encode($accessToken));
```
This way of storing tokens is not recommended due to security concerns and used for demonstration purpose.
Please, ensure that tokens are stored securely.

#### Setting Access Token

You can use method `setAccessToken()` for the `LinkedIn\Client` class to set token stored as string. You have to pass
instance of `LinkedIn\AccessToken` to this method.

```php
use LinkedIn\AccessToken;
use LinkedIn\Client;

// instantiate the Linkedin client
$client = new Client(
    'LINKEDIN_APP_CLIENT_ID',
    'LINKEDIN_APP_CLIENT_SECRET'
);

// load token from the file
$tokenString = file_get_contents('token.json');
$tokenData = json_decode($tokenString, true);
// instantiate access token object from stored data
$accessToken = new AccessToken($tokenData['token'], $tokenData['expiresAt']);

// set token for client
$client->setAccessToken($accessToken);
```

#### Renewing Tokens
LinkedIn tokens expire after 60 days but you can renew your access token by using the refresh token. 
Tokens can be then refreshed for 365 days, after which the end-user must re-auth.
```php
$client->renewTokenFromRefreshToken($refreshToken);
```

#### Performing API calls

All API calls can be called through simple method:

```php
$profile = $client->api(
    'ENDPOINT',
    ['parameter name' => 'its value here'],
    'HTTP method like GET for example'
);
```

There are 3 helper methods:

```php
// get method
$client->get('ENDPOINT', ['param' => 'value']);

//post
$client->post('ENDPOINT', ['param' => 'value']);

// delete
$client->delete('ENDPOINT');
```

#### Examples

##### Perform api call to get profile information

```php
$profile = $client->get(
    'me',
    ['fields' => 'id,firstName,lastName']
);
print_r($profile);
```

##### List companies where you are an admin

```php
$profile = $client->get(
    'organizationalEntityAcls',
    ['q' => 'roleAssignee']
);
print_r($profile);
```



##### Get Company page profile

```php
$companyId = '123'; // use id of the company where you are an admin
$companyInfo = $client->get('organizations/' . $companyId);
print_r($companyInfo);
```



##### Setup custom API request headers

Change different headers sent to LinkedIn API.

```php
$client->setApiHeaders([
  'Content-Type' => 'application/json',
  'x-li-format' => 'json',
  'X-Restli-Protocol-Version' => '2.0.0', // use protocol v2
  'x-li-src' => 'msdk' // set a src header to "msdk" to mimic a mobile SDK
]);
```

##### Change default API root

Some private API access there.

```php
$client->setApiRoot('https://api.linkedin.com/v2/');
```
