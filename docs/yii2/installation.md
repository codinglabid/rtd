# Installation

install yii2 advanced

```bash
  composer create-project --prefer-dist yiisoft/yii2-app-advanced nama_project
```

## Setup vhost

buka file vhost, untuk mac ada di `/usr/local/etc/httpd/extra/httpd-vhost.conf`
lalu tambahkan

```bash
<VirtualHost *:80>
  DocumentRoot "/Users/muhammadansorinasution/Sites/nama_project"
  ServerName nama_project.test
</VirtualHost>
```

untuk `/Users/muhammadansorinasution/Sites/` sesuaikan dengan path masing masing.
langkah selanjutnya buka file `/private/etc/hosts` pada mac lalu tambahkan `127.0.0.1 nama_project.test`
kemudian restart apache.

## Init Project

buka project menggunakan editor seperti vscode, pada contoh ini kita menggunakan vscode.
buka terminal lalu jalankan cli berikut:

- php init
- pilih mode develop atau production

## htaccess

buat file di root project `.htaccess`
tambahkan ini pada file tersebut

```htaccess
Options +FollowSymlinks
RewriteEngine On

# ===================== admin =====================
RewriteCond %{REQUEST_URI} ^/(admin)
RewriteRule ^admin/assets/(.*)$ backend/web/assets/$1 [L]
RewriteRule ^admin/css/(.*)$ backend/web/css/$1 [L]
RewriteRule ^admin/images/(.*)$ backend/web/images/$1 [L]
RewriteRule ^admin/js/(.*)$ backend/web/js/$1 [L]
RewriteRule ^admin/fonts/(.*)$ frontend/web/fonts/$1 [L]
RewriteRule ^admin/icon/(.*)$ frontend/web/icon/$1 [L]

RewriteCond %{REQUEST_URI} !^/backend/web/(assets|css|images|js|fonts|icon)/
RewriteCond %{REQUEST_URI} ^/(admin)
RewriteRule ^.*$ backend/web/index.php [L]
# ===================== end admin =====================

# ===================== frontend =====================
RewriteCond %{REQUEST_URI} ^/(assets|css)
RewriteRule ^assets/(.*)$ frontend/web/assets/$1 [L]
RewriteRule ^css/(.*)$ frontend/web/css/$1 [L]
RewriteRule ^js/(.*)$ frontend/web/js/$1 [L]
RewriteRule ^images/(.*)$ frontend/web/images/$1 [L]
RewriteRule ^fonts/(.*)$ frontend/web/fonts/$1 [L]
RewriteRule ^icon/(.*)$ frontend/web/icon/$1 [L]

RewriteCond %{REQUEST_URI} !^/(frontend|backend)/web/(assets|css|images|js|fonts|icon)/
RewriteCond %{REQUEST_URI} !index.php
RewriteCond %{REQUEST_FILENAME} !-f [OR]
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^.*$ frontend/web/index.php

# ===================== end frontend =====================
```

Note: setiap penambahan modul ataupun app pada project maka lakukan penambahan pada .htaccess sesuai dengan modul yang dibuat

## Pretty Url

buka file `nama_app/config/main.php` lalu cari urlManager dan uncomment.
langka selanjutnya kita akan membuat file `Request.php` pada folder `commmon/components`
lalu tambahkan kode berikut:

```php
<?php

namespace common\components;

class Request extends \yii\web\Request {
    public $web;
    public $adminUrl;

    public function getBaseUrl(){
        return str_replace($this->web, "", parent::getBaseUrl()) . $this->adminUrl;
    }
    public function resolvePathInfo(){
        if($this->getUrl() === $this->adminUrl){
            return "";
        }else{
            return parent::resolvePathInfo();
        }
    }
}
```

buka kembali file `nama_app/config/main.php` modifikasi request yang ada

```php
[
  ...
  'components' => [
    ...
     'request' => [
            'csrfParam' => '_csrf-frontend',
            'class' => 'common\components\Request',
            'web'=> '/frontend/web',
            // 'adminUrl' => '/admin' // untuk adminUrl harap di perhatikan biasanya ini di aktifkan selain daripada app landing
      ],
    ...
  ]
]
```

## kebutuhan library dasar

- [https://github.com/mdmsoft/yii2-admin](https://github.com/mdmsoft/yii2-admin)
- [https://github.com/johnitvn/yii2-ajaxcrud](https://github.com/johnitvn/yii2-ajaxcrud)
- [https://github.com/mdmsoft/yii2-upload-file](https://github.com/mdmsoft/yii2-upload-file)
- [https://demos.krajee.com/widget-details/datepicker](https://demos.krajee.com/widget-details/datepicker)
- [https://demos.krajee.com/number](https://demos.krajee.com/number)
- [https://demos.krajee.com/widget-details/select2](https://demos.krajee.com/widget-details/select2)
