# Login with facebook

untuk dokumentasi login dengan facebook bisa di lihat di [https://developers.facebook.com/docs/facebook-login/web](https://developers.facebook.com/docs/facebook-login/web)

## Verifikasi token

```php
$curl = curl_init();
curl_setopt($curl, CURLOPT_URL,  'https://graph.facebook.com/me?fields=email&access_token=' . input token);
curl_setopt($curl, CURLOPT_RETURNTRANSFER, 1);
$output = json_decode(curl_exec($curl));
curl_close($curl);

```
