
# WhatsApp notification channel for Laravel

[![Latest Version on Packagist](https://img.shields.io/packagist/v/emanueletoffolon/laravel-notification-whatsapp.svg?style=flat-square)](https://packagist.org/packages/emanueletoffolon/laravel-notification-whatsapp/)
[![Software License](https://img.shields.io/badge/license-MIT-brightgreen.svg?style=flat-square)](LICENSE.md)
[![Build Status](https://img.shields.io/github/workflow/status/emanueletoffolon/laravel-notification-whatsapp/Tests.svg?style=flat-square)](https://github.com/emanueletoffolon/laravel-notification-whatsapp/actions)
[![StyleCI](https://github.styleci.io/repos/576005059/shield)](hhttps://github.styleci.io/repos/576005059)
[![Quality Score](https://img.shields.io/scrutinizer/g/emanueletoffolon/laravel-notification-whatsapp.svg?style=flat-square)](https://scrutinizer-ci.com/g/emanueletoffolon/laravel-notification-whatsapp)
[![Code Coverage](https://img.shields.io/scrutinizer/coverage/g/emanueletoffolon/laravel-notification-whatsapp/main.svg?style=flat-square)](https://scrutinizer-ci.com/g/emanueletoffolon/laravel-notification-whatsapp?branch=main)
[![Total Downloads](https://img.shields.io/packagist/dt/emanueletoffolon/laravel-notification-whatsapp.svg?style=flat-square)](https://packagist.org/packages/emanueletoffolon/laravel-notification-whatsapp/)

This package makes it easy to send notifications using [WhatsApp Cloud API](https://developers.facebook.com/docs/whatsapp/cloud-api/) with Laravel.

This package uses the [whatsapp-cloud-api](https://github.com/netflie/whatsapp-cloud-api) library that will allow you to send messages via the WhatsApp Cloud API from any type of project and framework written in PHP.

## Contents

- [Installation](#installation)
	- [Setting up the WhatsApp service](#setting-up-the-WhatsApp-service)
- [Usage](#usage)
	- [Available Message methods](#available-message-methods)
- [Changelog](#changelog)
- [Testing](#testing)
- [Security](#security)
- [Contributing](#contributing)
- [Credits](#credits)
- [License](#license)


## Installation

You can install the package via composer:
```
composer require emanueletoffolon/laravel-notification-whatsapp
```
### Setting up the WhatsApp Cloud API

Create a new Meta application and get your Whatsapp `application token` and `phone number id` following the ["Get Started"](https://developers.facebook.com/docs/whatsapp/cloud-api/get-started?locale=en_US#set-up-developer-assets) guide. Place them inside your `.env` file. To load them, add this to your `config/services.php` file:
```php
<?php

return [

    /*
    |--------------------------------------------------------------------------
    | Third Party Services
    |--------------------------------------------------------------------------
    |
    | This file is for storing the credentials for third party services such
    | as Mailgun, Postmark, AWS and more. This file provides the de facto
    | location for this type of information, allowing packages to have
    | a conventional file to locate the various service credentials.
    |
    */

    // Other third-party config...

    'whatsapp' => [
        'from-phone-number-id' => env('WHATSAPP_FROM_PHONE_NUMBER_ID'),
        'token' => env('WHATSAPP_TOKEN'),
    ],

];
```

## Usage

The Whatsapp API only allows you to start conversations if you send a template message. This means that you will only be able to send template notifications from this package.

Whatsapp forces you to configure your templates before using them. You can learn how to configure your templates by following Meta's official guide on ["How to create templates"](https://developers.facebook.com/docs/whatsapp/cloud-api/guides/send-message-templates).

### WhatsApp templates sections

A template is divided into 4 sections: header, body, footer and buttons. The header, body and buttons accept variables. The footer doesn't accept variables. You can only send variables from this package for the header and body.

### Components

You have available several components that can be used to add context (variables) to your templates. The different components can be created with the component factory:

```php
<?php

use NotificationChannels\WhatsApp\Component;

Component::currency($amount, $code = 'EUR');
Component::dateTime($dateTime, $format = 'Y-m-d H:i:s');
Component::document($link);
Component::image($link);
Component::video($link);
Component::text($text);
Component::urlButton($array_of_urls);
Component::quickReplyButton($array_of_payloads);
Component::flowButton($flow_token, $array_of_data);
```
Components supported by Whatsapp template sections:

 - Header: image, video, document and text (the text accepts currency, datetime and text variables)
 - Body: currency, datetime and text
 - Buttons: url and quick reply, 

### Send a notification from a template

To use this package, you need to create a notification class, like `MovieTicketPaid` from the example below, in your Laravel application. Make sure to check out [Laravel's documentation](https://laravel.com/docs/master/notifications) for this process.

```php
<?php

namespace App\Notifications;

use Illuminate\Notifications\Notification;
use NotificationChannels\WhatsApp\Component;
use NotificationChannels\WhatsApp\WhatsAppChannel;
use NotificationChannels\WhatsApp\WhatsAppTemplate;

class MovieTicketPaid extends Notification
{
    /**
     * Get the notification's delivery channels.
     *
     * @param  mixed  $notifiable
     * @return array
     */
    public function via($notifiable)
    {
        return [WhatsAppChannel::class];
    }

    public function toWhatsapp($notifiable)
    {
        return WhatsAppTemplate::create()
            ->name('sample_movie_ticket_confirmation') // Name of your configured template
            ->header(Component::image('https://lumiere-a.akamaihd.net/v1/images/image_c671e2ee.jpeg'))
            ->body(Component::text('Star Wars'))
            ->body(Component::dateTime(new \DateTimeImmutable))
            ->body(Component::text('Star Wars'))
            ->body(Component::text('5'))
            ->buttons(Component::quickReplyButton(['Thanks for your reply!']))
            ->buttons(Component::urlButton(['reply/01234'])) // List of url suffixes
            ->to('34676010101');
    }
}
```

### Send a text message

You can only send a text message after you've send a template and the user responded.

```php
<?php

namespace App\Notifications;

use Illuminate\Notifications\Notification;
use NotificationChannels\WhatsApp\Component;
use NotificationChannels\WhatsApp\WhatsAppChannel;
use NotificationChannels\WhatsApp\WhatsAppTextMessage;

class MovieTicketPaid extends Notification
{
    /**
     * Get the notification's delivery channels.
     *
     * @param  mixed  $notifiable
     * @return array
     */
    public function via($notifiable)
    {
        return [WhatsAppChannel::class];
    }

    public function toWhatsapp($notifiable)
    {
        return WhatsAppTextMessage::create()
            ->message('Hello, this is a test message')
            ->to('34676010101');
    }
}
```

## Changelog

Please see [CHANGELOG](CHANGELOG.md) for more information what has changed recently.

## Testing

``` bash
$ composer test
```

## Security

If you discover any security related issues, please email hola@netflie.es instead of using the issue tracker.

## Contributing

Please see [CONTRIBUTING](CONTRIBUTING.md) for details.

## Credits

- [Álex Albarca](https://github.com/netflie)
- [All Contributors](../../contributors)

## License

The MIT License (MIT). Please see [License File](LICENSE.md) for more information.
