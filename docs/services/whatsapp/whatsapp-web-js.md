# Whatsapp Web js

Whatsapp web js ialah suatu library yang memungkinkan kita untuk membuat sebuah bot Whatsapp. Whatsapp web ini berjalan menggunakan [puppeter](https://pptr.dev/) yang memungkinkan kita merunning browser lewat syntax dan kita bisa mengontrolnya lewat syntax, defaultnya menggunakan chromium. hingga dokumentasi ini ditulis library ini running menggunakan node js.

## Preinstall

pastikan node js sudah terinstall pada komputer yang di gunakan, jika belum silahkan download pada [https://nodejs.org/en/download/](https://nodejs.org/en/download/) pilih sesuai os yang di gunakan, download dan lakukan installasi

## Installation

langkah pertama silhkan buat folder whatsapp, masuk ke dalam folder whatsapp kemudian jalankan perintah `npm init` enter sampai proses selesai. sekarang kita install dependencies nya dengan menjalankan perintah `npm install express express-validator nodemon qrcode socket.io whatsapp-web.js`

## Syntax

#### app.js

```javascript
const { Client, LocalAuth } = require("whatsapp-web.js");
const express = require("express");
const { body, validationResult } = require("express-validator");

const qrcode = require("qrcode");
const http = require("http");
const { phoneNumberFormatter } = require("./helpers/formatter");

const app = express();
const { Server } = require("socket.io");
const server = http.createServer(app);
const io = new Server(server);

const port = process.env.PORT || 8000;

app.use(express.json());
app.use(
  express.urlencoded({
    extended: true,
  })
);

app.get("/", (req, res) => {
  res.sendFile("index.html", {
    root: __dirname,
  });
});

const client = new Client({
  restartOnAuthFail: true,
  puppeteer: {
    headless: true,
    args: [
      "--no-sandbox",
      "--disable-setuid-sandbox",
      "--disable-dev-shm-usage",
      "--disable-accelerated-2d-canvas",
      "--no-first-run",
      "--no-zygote",
      "--single-process", // <- this one doesn't works in Windows
      "--disable-gpu",
    ],
  },
  authStrategy: new LocalAuth(),
});

client.initialize();

io.on("connection", function (socket) {
  console.log("connection");
  socket.emit("message", "Connecting...");

  client.on("qr", (qr) => {
    console.log("QR RECEIVED", qr);
    qrcode.toDataURL(qr, (err, url) => {
      socket.emit("qr", url);
      socket.emit("message", "QR Code received, scan please!");
    });
  });

  client.on("ready", () => {
    console.log("ready");
    socket.emit("ready", "Whatsapp is ready!");
    socket.emit("message", "Whatsapp is ready!");
  });

  client.on("authenticated", () => {
    socket.emit("authenticated", "Whatsapp is authenticated!");
    socket.emit("message", "Whatsapp is authenticated!");
    console.log("AUTHENTICATED");
  });

  client.on("auth_failure", function (session) {
    console.log("auth_failure");
    socket.emit("message", "Auth failure, restarting...");
  });

  client.on("disconnected", (reason) => {
    console.log("disconnected");
    socket.emit("message", "Whatsapp is disconnected!");
    client.destroy();
    client.initialize();
  });
});

const checkRegisteredNumber = async function (number) {
  const isRegistered = await client.isRegisteredUser(number);
  return isRegistered;
};

// Send message
app.post(
  "/send-message",
  [body("number").notEmpty(), body("message").notEmpty()],
  async (req, res) => {
    const errors = validationResult(req).formatWith(({ msg }) => {
      return msg;
    });

    if (!errors.isEmpty()) {
      return res.status(422).json({
        status: false,
        message: errors.mapped(),
      });
    }

    const number = phoneNumberFormatter(req.body.number);
    const message = req.body.message;

    const isRegisteredNumber = await checkRegisteredNumber(number);

    if (!isRegisteredNumber) {
      return res.status(422).json({
        status: false,
        message: "The number is not registered",
      });
    }

    client
      .sendMessage(number, message)
      .then((response) => {
        res.status(200).json({
          status: true,
          response: response,
        });
      })
      .catch((err) => {
        res.status(500).json({
          status: false,
          response: err,
        });
      });
  }
);

server.listen(port, function () {
  console.log("App running on *: " + port);
});
```

#### index.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>

    <style>
      .hide {
        display: none;
      }
    </style>
  </head>
  <body>
    <div id="app">
      <img src="" alt="QR Code" id="qrcode" class="hide" />
      <h3>Logs:</h3>
      <ul class="logs"></ul>
    </div>

    <script
      src="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.5.1/jquery.min.js"
      crossorigin="anonymous"
    ></script>
    <script src="/socket.io/socket.io.js" crossorigin="anonymous"></script>

    <script>
      $(document).ready(function () {
        var socket = io();

        socket.on("message", function (msg) {
          $(".logs").append($("<li>").text(msg));
        });

        socket.on("qr", function (src) {
          $("#qrcode").attr("src", src);
          $("#qrcode").show();
        });

        socket.on("ready", function (data) {
          $("#qrcode").hide();
        });

        socket.on("authenticated", function (data) {
          $("#qrcode").hide();
        });
      });
    </script>
  </body>
</html>
```

## Penggunaan

untuk menjalankannya silahkan jalankan perintah `nodemon app.js` kemudian buka `http://localhost:8000` pada browser

tunggu beberapa saat hingga barcode muncul, setelah barcode muncul scan barcode tersebut dengan akun Whatsapp, mausk dari tautkan perangkat. ketika scan berhasil silahkan tunggu hingga status ready, dan Api pun siap di gunakan

## API Reference

#### Send Message

```http
  POST /send-message
```

| Body      | Type     | Description                |
| :-------- | :------- | :------------------------- |
| `number`  | `number` | **Required**. Your number  |
| `message` | `string` | **Required**. Your message |

| Headers        | value              |
| :------------- | :----------------- |
| `Content-Type` | `application/json` |

## Reference

- [https://github.com/pedroslopez/whatsapp-web.js/](https://github.com/pedroslopez/whatsapp-web.js/)
- [https://github.com/ngekoding/whatsapp-api-tutorial](https://github.com/ngekoding/whatsapp-api-tutorial)
- [https://socket.io/](https://socket.io/)
- [https://expressjs.com/](https://expressjs.com/)

## Deployment VPS

untuk melakukan deploy node js silahkan install [https://pm2.keymetrics.io/](https://pm2.keymetrics.io/) pada pvs yang ada. setelah pm2 berhasil di install clone source yang telah di buat tadi pada github terkait dan masuk ke dalam folder hasil clone tadi. jalankan perintah `pm2 start app.js --name==wa --watch` dan aplikasipun siap di gunakan pada port 8000 sesuai dengan yang kita buat pada app.js dan pastikan port tersebut sudah di buka pada firewall
