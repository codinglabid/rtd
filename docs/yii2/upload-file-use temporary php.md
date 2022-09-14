# Upload File use temporary php

Pada jaman dahulu ketika kita memiliki 2 buah server dimana server a yang berfungsi untuk menghandle upload dari user, sementara server b adalah tempat penyimpanan file sebenarnya. pada contoh dulu kita akan membuat temporary sendiri pada project di common `temp` lalu kita akan melakukan file save as ke dalam folder tersebut, lalu ketika file berhasil di upload atau gagak upload ke server b kita juga akan melakukan unlink secara manual. untuk sekarang kita bisa memnfaatkan langsung temporary yang sudah otomatis di buat oleh method getInstance.

## Usage/Examples

server A

```php
$file = UploadedFile::getInstance($model, 'file');
$client = new Client();
$token = Yii::$app->api->token();
$baseUrl = Yii::$app->params['api']['baseUrl'];
$response = $client->createRequest()
    ->setMethod('POST')
    ->setUrl($baseUrl)
    ->setData([
        'name' => $file->name,
        'extension' => $file->extension
    ])
    ->addFile('imageFile', $file->tempName)//lokaso temporary yang di buat secara otomatis oleh getInstance
    ->addHeaders([
        'Authorization' => "Bearer $token"
    ])
    ->send();
```

server B
untuk handling upload file disini kita menggunakan model dari mdm

```php
$model = new Test();
$post = Yii::$app->request->post();
$file = UploadedFile::getInstanceByName('imageFile');
$file->name = $post['name'];
$FileModel = FileModel::saveAs($file,['uploadPath' => '@common/upload/']);

```
