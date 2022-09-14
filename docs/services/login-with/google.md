# Login With Google

berikut link penggunaan login dengan google di web [https://developers.google.com/identity/gsi/web/guides/overview](https://developers.google.com/identity/gsi/web/guides/overview)

catatan: harap pastikan telah membuat oauth client id pada [https://console.cloud.google.com/](https://console.cloud.google.com/) dan mengisi oauth consent screen

## Verifikasi token

untuk verifikasi token gunakan Google API Client Library `composer require google/apiclient`

```php
$post = Yii::$app->request->post();
$jwt = new JWT();
$jwt::$leeway = 10;

$client = new GoogleClient(['client_id' => your client id, 'jwt' => $jwt]);
$payload = $client->verifyIdToken(input token);

```
