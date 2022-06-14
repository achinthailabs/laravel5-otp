# laravel5-otp

[![Latest Version on Packagist][ico-version]][link-packagist]
[![Software License][ico-license]](LICENSE.md)
[![Build Status][ico-travis]][link-travis]
[![Coverage Status][ico-scrutinizer]][link-scrutinizer]
[![Quality Score][ico-code-quality]][link-code-quality]
[![Total Downloads][ico-downloads]][link-downloads]

The code was taken directly from [https://github.com/teckwei1993/laravel-otp](https://github.com/teckwei1993/laravel-otp) and tweaked to our need. Please use original repo for your work.

A package for Laravel One Time Password (OTP) generator and validation without Eloquent Model, since it done by *Cache*.
The cache connection same as your laravel cache config and it supported: "apc", "array", "database", "file", "memcached", "redis"

## Install

Via Composer

``` bash
$ composer require achinthailabs/laravel5-otp
```

## Add Service Provider & Facade

**For Laravel 5.5+**

Once the package is added, the service provider and facade will be auto discovered.

**For Laravel 5.2 / 5.3 / 5.4**

Add the ServiceProvider to the providers array in `config/app.php`:

```php
Achinthailabs\Otp\OtpServiceProvider::class
```

Add the Facade to the aliases array in `config/app.php`:

```php
'Otp' => Achinthailabs\Otp\OtpFacade::clas
```

## Configuration

Publish config and language file

```bash
php artisan vendor:publish --provider="Achinthailabs\Otp\OtpServiceProvider"
```

This package publishes an `otp.php` file inside your applications's config folder which contains the settings for this package. Most of the variables are bound to environment variables, you may add Key-Value pair to the `.env` file in the Laravel application.

```
OTP_FORMAT=numeric
OTP_LENGTH=6
OTP_SENSITIVE=false
OTP_EXPIRES_TIME=15
OTP_ATTEMPT_TIMES=5
OTP_REPEATED=true
OTP_DEMO=false
```

## Usage

### Generate OTP

```php
Otp::generate(string $identifier)
```

* `$identifier`: The identity that will be tied to the OTP.

#### Sample

```php
use OTP;

// in controller

$password = Otp::generate('reg:name@domain.com');
```

This will generate a OTP that will be valid for 15 minutes.

### Validate OTP

```php
Otp::validate(string $identifier, string $password)
```

* `$identifier`: The identity that is tied to the OTP.
* `$password`: The password tied to the identity.

#### Sample

```php
use OTP;

// in controller

$result = Otp::validate('reg:name@domain.com', '123456');
```

#### Responses

**On Success**

```object
{
  "status": true
}
```

**Invalid OTP**

```object
{
  "status": false,
  "error": "invalid"
}
```

**Expired**

```object
{
  "status": false,
  "error": "expired"
}
```

**Max attempt**

```object
{
  "status": false,
  "error": "max_attempt"
}
```

* Reached the maximum allowed attempts, default 10 times with each identifier

### Validate OTP by Laravel Validation

```php
// in a `FormRequest`

use Achinthailabs\Otp\Rules\OtpValidate;

public function rules()
{
    return [
        'code' => ['required', new OtpValidate('change-email:name@domain.com')]
    ];
}

// in a controller

$request->validate([
    'code' => ['required', new OtpValidate('change-email:name@domain.com')]
]);
```

### Validate OTP by session id

```php
// Otp class

$result = Otp::validate('123456');

// in a `FormRequest`

use Achinthailabs\Otp\Rules\OtpValidate;

public function rules()
{
    return [
        'code' => ['required', new OtpValidate()]
    ];
}

// in a controller

$request->validate([
    'code' => ['required', new OtpValidate()]
]);
```

* The setting without identifier will automatically use the session ID as the default, and the OTP generation and verification will be completed in same session (browser's cookies).

## Advanced Usage

### Generate OTP with options

```php
$password = Otp::setLength(8)->setFormat('string')->setExpires(60)->setRepeated(false)->generate('identifier-key-here');

// or array option

$password = Otp::generate('identifier-key-here', [
    'length' => 8,
    'format' => 'string',
    'expires' => 60,
    'repeated' => false
]);
```

* `setLength($length)`: The length of the password. Default: 6
* `setFormat($format)`: The format option allows you to decide which generator implementation to be used when generating new passwords. Options: 'string','numeric','numeric-no-zero','customize'. Default: "numeric"
* `setExpires($minutes)`: The expiry time of the password in minutes. Default: 15
* `setRepeated($boolean)`: The repeated of the password. The previous password is valid when new password generated until either one password used or itself expired. Default: true

### Generate OTP with customize password

```php
$password = Otp::setCustomize('12345678ABC@#$')->generate('identifier-key-here');
```

* `setCustomize($string)`: Random letter from the customize string

### Validate OTP with specific attempt times

```php
$password = Otp::setAttempts(3)->validate('identifier-key-here', 'password-here');
```

* `setAttempts($times)`: The number of incorrect password attempts. Default: 5

### Validate OTP with case sensitive

```php
$password = Otp::setSensitive(true)->generate('identifier-key-here');

// validate

$result = Otp::setSensitive(true)->validate('identifier-key-here', 'password-here');

// in controller

use Achinthailabs\Otp\Rules\OtpValidate;

$request->validate([
    'code' => ['required', new OtpValidate('identifier-key-here', ['sensitive' => true])]
]);
```

* `setSensitive($boolean)`: Requiring correct input of uppercase and lowercase letters. Default: true

### Generate OTP with seperate password

```php
$password = Otp::setLength([4,3,4])->setSeparator(':')->generate('identifier-key-here');
```
**Sample password**

```
3526:126:3697
```

* `setLength($array)`: The length of the password, use array to separate each length.
* `setSeparator($string)`: The separator of the password. Default: "-"

### Validate OTP with extra data

```php
$password = Otp::setData(['user_id' => auth()->id()])->generate('login-confirmation');
```

* `setData($var)`: Allows you to get the extra data of OTP.

```php
// validate

$result = Otp::setDisposable(false)->validate('login-confirmation', 'password-here');

// in controller

use Achinthailabs\Otp\Rules\OtpValidate;

$request->validate([
    'code' => ['required', new OtpValidate('login-confirmation', ['disposable' => false])]
]);
```

* `setDisposable($boolean)`: The disposable of the Otp identifier, the different password is not valid when same identifier password used. Default: true

**On Success Response**

```object
{
  "status": true,
  "data": [
    "user_id": 10
  ]
}
```

* When you set disposable to `false`, you are able support different password with different extra data for different user in the same identifier key of the OTP.

### Validate OTP with skip using

```php
// validate

$result = Otp::setSkip(true)->validate('identifier-key-here', 'password-here');

// in controller

use Achinthailabs\Otp\Rules\OtpValidate;

$request->validate([
    'code' => ['required', new OtpValidate('identifier-key-here', ['skip' => true])]
]);
```

* `setSkip($boolean)`: Skip using the password when validate, which means you can reuse the password again. Default: false
* When there is an error response to the form request, it will skip using the password, but remember to `OTP::validate(...)`  in controller.

### Delete OTP

```php
Otp::forget('identifier-key-here');
```

* Delete all password with this specific identifier

### Delete specific password

```php
Otp::forget('identifier-key-here', 'password-here');
```

### Reset attempt times

```php
Otp::resetAttempt('identifier-key-here');
```

### Demo password

Add the following Key-Value pair to the `.env` file in the Laravel application.

```
OTP_DEMO=true
```

* Demo mode for development purposes, no need to use real password to validate.
* Default demo password: "1234", "123456", "12345678"
* 
## Change log

Please see [CHANGELOG](CHANGELOG.md) for more information on what has changed recently.

## Contributing

Please see [CONTRIBUTING](CONTRIBUTING.md) and [CODE_OF_CONDUCT](CODE_OF_CONDUCT.md) for details.

## Security

If you discover any security related issues, please email achintha@incubatelabs.com instead of using the issue tracker.

## Credits

- [Achintha Rodrigo][link-author]
- [All Contributors][link-contributors]

## License

The MIT License (MIT). Please see [License File](LICENSE.md) for more information.

[ico-version]: https://img.shields.io/packagist/v/achinthailabs/laravel5-otp.svg?style=flat-square
[ico-license]: https://img.shields.io/badge/license-MIT-brightgreen.svg?style=flat-square
[ico-travis]: https://img.shields.io/travis/achinthailabs/laravel5-otp/master.svg?style=flat-square
[ico-scrutinizer]: https://img.shields.io/scrutinizer/coverage/g/achinthailabs/laravel5-otp.svg?style=flat-square
[ico-code-quality]: https://img.shields.io/scrutinizer/g/achinthailabs/laravel5-otp.svg?style=flat-square
[ico-downloads]: https://img.shields.io/packagist/dt/achinthailabs/laravel5-otp.svg?style=flat-square

[link-packagist]: https://packagist.org/packages/achinthailabs/laravel5-otp
[link-travis]: https://travis-ci.org/achinthailabs/laravel5-otp
[link-scrutinizer]: https://scrutinizer-ci.com/g/achinthailabs/laravel5-otp/code-structure
[link-code-quality]: https://scrutinizer-ci.com/g/achinthailabs/laravel5-otp
[link-downloads]: https://packagist.org/packages/achinthailabs/laravel5-otp
[link-author]: https://github.com/achinthailabs
[link-contributors]: ../../contributors
