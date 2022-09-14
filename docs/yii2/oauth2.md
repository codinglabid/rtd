# Yii2 Oauth2

## Installation

Install with composer

```bash
    composer require --prefer-dist filsh/yii2-oauth2-server "*"
```

Untuk menggunakan ekstensi ini, cukup tambahkan kode berikut pada main.php pada module yang ingin di gunakan:

```php
    'bootstrap' => ['oauth2'],
    'modules' => [
        'oauth2' => [
            'class' => 'filsh\yii2\oauth2server\Module',
            'tokenParamName' => 'accessToken',
            'tokenAccessLifetime' => 3600 * 24,
            'storageMap' => [
                'user_credentials' => 'common\models\User',
            ],
            'grantTypes' => [
                'user_credentials' => [
                    'class' => 'OAuth2\GrantType\UserCredentials',
                ],
                'refresh_token' => [
                    'class' => 'OAuth2\GrantType\RefreshToken',
                    'always_issue_new_refresh_token' => true
                ]
            ]
        ]
    ],
    'components' => [
        ...,
        'urlManager' => [
            ...,
            'rules' => [
                'POST oauth2/<action:\w+>' => 'oauth2/rest/<action>',
            ]
        ],
    ]
```

pada model user di common models tambahkan implementasi dari:
`use OAuth2\Storage\UserCredentialsInterface;` menjadi seperti ini `class User extends ActiveRecord implements IdentityInterface, UserCredentialsInterface {...}`
lalu pada tambahkan method ini pada model user

```php
    public function checkUserCredentials($username, $password)
    {
        $model = $this->findByUsername($username);
        return $model;
    }

    public function getUserDetails($username)
    {
        $model = $this->findByUsername($username);
        return [
            'user_id' => $model->id
        ];
    }
```

langkah selanjutnya pindahkan migrate yang berada di vendor `@vendor/filsh/yii2-oauth2-server/src/migrations` ke `console\migrations`

selanjutnya buka postman atau sejenis untuk melakukan request token

## API Reference

#### Request Token

```http
  POST /oauth2/token
```

| body            | Type     | Description                      | Default    |
| :-------------- | :------- | :------------------------------- | :--------- |
| `grant_type`    | `string` | **Required**. Your grant type    | password   |
| `username`      | `string` | **Required**. Your username      | null       |
| `password`      | `string` | **Required**. Your password      | null       |
| `client_id`     | `string` | **Required**. Your client id     | testclient |
| `client_secret` | `string` | **Required**. Your client secret | testpass   |

## Usage/Examples

```php
<?php

namespace frontend\controllers;

use yii\helpers\ArrayHelper;
use yii\filters\auth\HttpBearerAuth;
use yii\filters\auth\QueryParamAuth;
use filsh\yii2\oauth2server\filters\ErrorToExceptionFilter;
use filsh\yii2\oauth2server\filters\auth\CompositeAuth;

class HaloController extends \yii\rest\Controller
{
    /**
     * @inheritdoc
     */
    // start validasi token
    public function behaviors()
    {
        return ArrayHelper::merge(parent::behaviors(), [
            'authenticator' => [
                'class' => CompositeAuth::class,
                'authMethods' => [
                    ['class' => HttpBearerAuth::class],
                    ['class' => QueryParamAuth::class, 'tokenParam' => 'accessToken'],
                ]
            ],
            'exceptionFilter' => [
                'class' => ErrorToExceptionFilter::class
            ],
        ]);
    }
    // end validasi token

    public function actionIndex()
    {
        return 'Halo';
    }
}

```

silahkan request token lalu copykan access_token yang di dapat dan akses endpoint /halo dengan memasukkan authorization nya dari token yang tadi

## Generate Access token with username

langkah pertama manfaatkan method login user, kemudian generate token melalui method `createAccessToken` pada class server filsh. pada kasus yang digunakan untuk login with google, facebook, karna kita tidak tau password yang digunakan setiap user. maka kita generate token hanya dengan user login menggunakan username tanpa validasi password

contoh penggunaan

```php
Yii::$app->user->login($user, $this->rememberMe ? 3600 * 24 * 30 : 0);
$module = Yii::$app->getModule('oauth2');
$acces_token = $module->getServer()->createAccessToken('testclient', Yii::$app->getUser()->getId());
```

## Reference

- [https://github.com/filsh/yii2-oauth2-server](https://github.com/filsh/yii2-oauth2-server)
- [https://github.com/bshaffer/oauth2-server-php](https://github.com/bshaffer/oauth2-server-php)
