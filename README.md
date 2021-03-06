[![Stories in Ready](https://badge.waffle.io/PHPAuth/PHPAuth.png?label=ready&title=Ready)](https://waffle.io/PHPAuth/PHPAuth)
[![Build Status](https://api.travis-ci.org/PHPAuth/PHPAuth.png)](https://api.travis-ci.org/PHPAuth/PHPAuth) 
[![Minimum PHP Version](https://img.shields.io/badge/php-%3E%3D%205.6-8892BF.svg?style=flat-circle)](https://php.net/)

PHPAuth
=======

What is it
---------------

PHPAuth is a secure user authentication class for PHP websites, using a powerful password hashing system and attack blocking to keep your website and users secure.

PHPAuth is work in progress, and not meant for people that doesn't know how to program, its meant for people that know what they are doing.. We cannot help everyone because they dont understand this class.. 

IT'S NOT FOR BEGINNERS!

Features
---------------
* Authentication by email and password combination
* Uses [bcrypt](http://en.wikipedia.org/wiki/Bcrypt) to hash passwords, a secure algorithm that uses an expensive key setup phase
* Uses an individual 128 bit salt for each user, pulled from /dev/urandom, making rainbow tables useless
* Uses PHP's [PDO](http://php.net/manual/en/book.pdo.php) database interface and uses prepared statements meaning an efficient system, resilient against SQL injection
* Blocks (or verifies) attackers by IP for any defined time after any amount of failed actions on the portal
* No plain text passwords are sent or stored by the system
* Integrates easily into most existing websites, and can be a great starting point for new projects
* Easy configuration of multiple system parameters
* Allows sending emails via SMTP or sendmail
* Blocks disposable email addresses from registration

User actions
---------------
* Login
* Register
* Activate account
* Resend activation email
* Reset password
* Change password
* Change email address
* Delete account
* Logout

Requirements
---------------
* PHP 5.6 or newer.
* MySQL / MariaDB database or PostGreSQL database

Composer Support
---------------
PHPAuth can now be installed with the following command:

`composer require phpauth/phpauth:dev-master`

Then: `require 'vendor/autoload.php';`

Configuration
---------------

The database table `config` contains multiple parameters allowing you to configure certain functions of the class.

* `site_name`   : the name of the website to display in the activation and password reset emails
* `site_url`    : the URL of the Auth root, where you installed the system, without the trailing slash, used for emails.
* `site_email`  : the email address from which to send activation and password reset emails
* `site_key`    : a random string that you should modify used to validate cookies to ensure they are not tampered with
* `site_timezone` : the timezone for correct datetime values
* `site_activation_page` : the activation page name appended to the `site_url` in the activation email
* `site_password_reset_page` : the password reset page name appended to the `site_url` in the password reset email
* `cookie_name` : the name of the cookie that contains session information, do not change unless necessary
* `cookie_path` : the path of the session cookie, do not change unless necessary
* `cookie_domain` : the domain of the session cookie, do not change unless necessary
* `cookie_secure` : the HTTPS only setting of the session cookie, do not change unless necessary
* `cookie_http` : the HTTP only protocol setting of the session cookie, do not change unless necessary
* `cookie_remember` : the time that a user will remain logged in for when ticking "remember me" on login. Must respect PHP's [strtotime](http://php.net/manual/en/function.strtotime.php) format.
* `cookie_forget` : the time a user will remain logged in when not ticking "remember me" on login.  Must respect PHP's [strtotime](http://php.net/manual/en/function.strtotime.php) format.
* `cookie_renew` : the maximum time difference between session expiration and last page load before allowing the session to be renewed. Must respect PHP`s [strtotime](http://php.net/manual/en/function.strtotime.php) format.
* `allow_concurrent_sessions` : Allow a user to have multiple active sessions (boolean). If false (default), logging in will end any existing sessions.
* `bcrypt_cost` : the algorithmic cost of the bcrypt hashing function, can be changed based on hardware capabilities
* `smtp` : `0` to use sendmail for emails, `1` to use SMTP
* `smtp_debug` : `0` to disable SMTP debugging, `1` to enable SMTP debugging, useful when you are having email/smtp issues
* `smtp_host` : hostname of the SMTP server
* `smtp_auth` : `0` if the SMTP server doesn't require authentication, `1` if authentication is required
* `smtp_username` : the username for the SMTP server
* `smtp_password` : the password for the SMTP server
* `smtp_port` : the port for the SMTP server
* `smtp_security` : `NULL` for no encryption, `tls` for TLS encryption, `ssl` for SSL encryption
* `verify_password_min_length` : minimum password length, default is `3`  
* `verify_email_min_length` : minimum EMail length, default is `5`
* `verify_email_max_length` : maximum EMail length, default is `100`
* `verify_email_use_banlist` : use banlist while checking allowed EMails (see `/files/domains.json`), default is `1` (`true`)
* `attack_mitigation_time` : time used for rolling attempts timeout, default is `+30 minutes`. Must respect PHP's [strtotime](http://php.net/manual/en/function.strtotime.php) format.
* `attempts_before_verify` : maximum amount of attempts to be made within `attack_mitigation_time` before requiring captcha. Default is `5`
* `attempt_before_ban` : maximum amount of attempts to be made within `attack_mitigation_time` before temporally blocking the IP address. Default is `30`
* `password_min_score` : the minimum score given by [zxcvbn](https://github.com/bjeavons/zxcvbn-php) that is allowed. Default is `3`
* `translation_source`: source of translation, possible values: 'sql' (data from <table_translations> will be used), 'php' (default, translations will be loaded from languages/*.php), 'ini' (will be used languages/*.ini files)
* `table_translations` : name of table with translation for all messages
* `table_attempts` : name of table with all attempts (default is 'phpauth_attempts')
* `table_requests` : name of table with all requests (default is 'phpauth_requests')
* `table_sessions` : name of table with all sessions (default is 'phpauth_sessions')
* `table_users` : name of table with all users (default is 'phpauth_users')
* `table_emails_banned` : name of table with all banned email domains (default is 'phpauth_emails_banned')
* `recaptcha_enabled`: 1 for Google reCaptcha enabled, 0 - disabled (default)
* `recaptcha_site_key`: string, contains public reCaptcha key (for javascripts)
* `recaptcha_secret_key`: string, contains secret reCaptcha key

The rest of the parameters generally do not need changing.

CAPTCHA Implementation
---------------

If `isBlocked()` returns `verify`, then a CAPTCHA code should be displayed.
The method `checkCaptcha($captcha)` is called to verify a CAPTCHA code. By default this method returns `true`, but should be overridden to verify a CAPTCHA.

For example, if you are using Google's ReCaptcha NoCaptcha, use the following code:

```php
    private function checkCaptcha($captcha)
    {
 try {

        $url = 'https://www.google.com/recaptcha/api/siteverify';
        $data = ['secret'   => 'your_secret_here',
            'response' => $captcha,
            'remoteip' => $this->getIp()];

        $options = [
            'http' => [
                'header'  => "Content-type: application/x-www-form-urlencoded\r\n",
                'method'  => 'POST',
                'content' => http_build_query($data)
            ]
        ];

        $context  = stream_context_create($options);
        $result = file_get_contents($url, false, $context);
        return json_decode($result)->success;
    }
    catch (\Exception $e) {
        return false;
    }
}
```

If a CAPTCHA is not to be used, please ensure to set `attempt_before_block` to the same value as `attempts_before_verify`.

Also, `Auth::checkReCaptcha()` method can be called.

How to secure a page
---------------

Making a page accessible only to authenticated users is quick and easy, requiring only a few lines of code at the top of the page:

```php
<?php

include("Config.php");
include("Auth.php");

$dbh = new PDO("mysql:host=localhost;dbname=phpauth", "username", "password");

$config = new PHPAuth\Config($dbh);
$auth   = new PHPAuth\Auth($dbh, $config);

if (!$auth->isLogged()) {
    header('HTTP/1.0 403 Forbidden');
    echo "Forbidden";

    exit();
}

?>
```

or

```php
<?php

require_once 'vendor/autoload.php';

use PHPAuth\Config as PHPAuthConfig;
use PHPAuth\Auth as PHPAuth;

$dbh = new PDO("mysql:host=localhost;dbname=phpauth", "username", "password");

$config = new PHPAuthConfig($dbh);
$auth = new PHPAuth($dbh, $config);

if (!$auth->isLogged()) {
    header('HTTP/1.0 403 Forbidden');
    echo "Forbidden";

    exit();
}

?>
```
**NB:** required package installed via composer: `composer require phpauth/phpauth:dev-master`!!!


Custom config sources
---------------------

By default, config defined at `phpauth_config` data table.

It is possible to define custom config from other sources: ini-file, other SQL-table or php-array:

```
Config($dbh, $config_type, $config_source, $config_language)
```
* `config_type`:
  * 'sql' (or empty value) - load config from database,
  * 'ini' - config must be declared in INI file (sections can be used for better readability, but will not be parsed)
  * 'array' - config will be loaded from $config_source (type of array)
* `config_source` -
  * for 'sql': name of custom table with configuration
  * for 'ini': path and name of INI file (for example: '$/config/config.ini', '$' means application root)
  * for 'array': it is a array with configuration
* `config_language` - custom language for site as locale value (default is 'en_GB')

Examples:

```
new Config($dbh); // load config from SQL table 'phpauth_config', language is 'en_GB'

new Config($dbh, '', 'my_config'); // load config from SQL table 'my_config', language is 'en_GB'

new Config($dbh, 'ini', '$/config/phpauth.ini'); // configuration will be loaded from INI file, '$' means Application basedir

new Config($dbh, 'array', $CONFIG_ARRAY); // configuration must be defined in $CONFIG_ARRAY value

new Config($dbh, '', '', 'ru_RU'); // load configuration from default SQL table and use ru_RU locale
```



Message languages
---------------------

The language for error and success messages returned by PHPAuth can be configured by passing in one of
the available languages as the third parameter to the Auth constructor. If no language parameter is provided
then the default `en_GB`language is used.

Example: `$auth   = new PHPAuth\Auth($dbh, $config, "fr_FR");`

Available languages:

* `ar-TN`
* `cs_CZ`
* `da_DK`
* `de_DE`
* `en_GB` (Default)
* `es_MX`
* `fa_IR`
* `fr_FR`
* `gr_GR`
* `hu_HU`
* `id_ID`
* `it_IT`
* `nl_BE`
* `nl_NL`
* `no_NB`
* `pl_PL`
* `ps_AF`
* `pt_BR`
* `ru_RU`
* `se_SE`
* `sl_SI`
* `sr_RS`
* `tr_TR`
* `th_TH`
* `uk_UA`
* `vi_VN`

Documentation
---------------

All class methods are documented in [the Wiki](https://github.com/PHPAuth/PHPAuth/wiki/Class-Methods)  
System error codes are listed and explained [here](https://github.com/PHPAuth/PHPAuth/wiki/System-error-codes)


Contributing
---------------

Anyone can contribute to improve or fix PHPAuth, to do so you can either report an issue (a bug, an idea...) or fork the repository, perform modifications to your fork then request a merge.

Credits
---------------

* [password_compat](https://github.com/ircmaxell/password_compat) - @ircmaxell
* [disposable](https://github.com/lavab/disposable) - @lavab
* [PHPMailer](https://github.com/PHPMailer/PHPMailer) - @PHPMailer
* [zxcvbn-php](https://github.com/bjeavons/zxcvbn-php) - @bjeavons
