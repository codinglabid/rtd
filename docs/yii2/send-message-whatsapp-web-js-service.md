## Send Message Wahatsapp Web Js Service

untuk melakukan pengiriman pesan melalui Wahatsapp web js Service pastikan service sudah berjalan

## Installation

Install my-project with npm

```bash
  composer require codinglab/yii2-whatsapp-web
```

## Usage/Examples

```php
$url = 'http://localhost:8000/send-message';
$phoneNumber = '081277758656';
$message = 'halo mas mas';
$WhatsApp = new WhatsAppWeb($url, $phoneNumber, $message);
$WhatsApp->sendMessage();
```

## Reference

- [https://github.com/codinglabid/whatsappweb](https://github.com/codinglabid/whatsappweb)
