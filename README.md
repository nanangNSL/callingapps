# Table of Contents
- [Description](#description)
- [Installation](#installation)
- [Usage](#usage)
- [Contributing](#contributing)
- [License](#license)

## Description

### Overview
Ini adalah dokumentasi untuk kode yang mengimplementasikan server menggunakan Express, Socket.IO, dan modul 'http' dari Node.js. Server ini berfungsi untuk menyediakan layanan web, mengatur direktori statis, dan mengaktifkan komunikasi real-time melalui Socket.IO.

1. **index.js (server)**

   ```bash
   // Import library yang diperlukan
   const path = require('path');
   const { createServer } = require('http');
   const express = require('express');
   const { getIO, initIO } = require('./socket');
   
   // Inisialisasi aplikasi Express
   const app = express();
   
   // Menggunakan middleware Express untuk mengatur direktori statis
   app.use('/', express.static(path.join(__dirname, 'static')));
   
   // Membuat server HTTP menggunakan modul 'http' dari Node.js
   const httpServer = createServer(app);
   
   // Mendefinisikan port yang akan digunakan oleh server
   let port = process.env.PORT || 3500;
   
   // Menginisialisasi Socket.IO pada server HTTP yang telah dibuat
   initIO(httpServer);
   
   // Mendengarkan server pada port yang telah ditentukan
   httpServer.listen(port);
   console.log("Server started on ", `http://127.0.0.1:${port}` );
   
   // Memanggil fungsi getIO() untuk mendapatkan instance Socket.IO
   getIO();
   
2. **index.js (server)**

   ```bash
   const { Server } = require("socket.io");
   let IO;
   // ### Inisialisasi Socket.IO
   // Fungsi `initIO` digunakan untuk menginisialisasi server Socket.IO pada server HTTP yang telah dibuat.
   module.exports.initIO = (httpServer) => {
   IO = new Server(httpServer);
   // Middleware untuk mengatur data pengguna pada koneksi.
   IO.use((socket, next) => {
    if (socket.handshake.query) {
      let callerId = socket.handshake.query.callerId;
      socket.user = callerId;
      next();
    }
   });
   // ### Koneksi
   // Kode di bawah ini menangani peristiwa ketika klien terhubung ke server.

   IO.on("connection", (socket) => {
    console.log(socket.user, "Connected");
    socket.join(socket.user);

    // ### Menangani Panggilan
    // Kode di bawah ini menangani peristiwa ketika klien memulai panggilan.

    socket.on("call", (data) => {
      let calleeId = data.calleeId;
      let rtcMessage = data.rtcMessage;

      // Mengirim pesan panggilan ke penerima panggilan.
      socket.to(calleeId).emit("newCall", {
        callerId: socket.user,
        rtcMessage: rtcMessage,
      });
    });

    // ### Menangani Jawaban Panggilan
    // Kode di bawah ini menangani peristiwa ketika klien menjawab panggilan.

    socket.on("answerCall", (data) => {
      let callerId = data.callerId;
      rtcMessage = data.rtcMessage;

      // Mengirim pesan bahwa panggilan telah dijawab.
      socket.to(callerId).emit("callAnswered", {
        callee: socket.user,
        rtcMessage: rtcMessage,
      });
    });

    // ### Menangani ICE Candidates
    // Kode di bawah ini menangani peristiwa saat kandidat ICE ditemukan.
   socket.on("ICEcandidate", (data) => {
      console.log("ICEcandidate data.calleeId", data.calleeId);
      let calleeId = data.calleeId;
      let rtcMessage = data.rtcMessage;
      console.log("socket.user emit", socket.user);

      // Mengirim kandidat ICE kepada penerima panggilan.
      socket.to(calleeId).emit("ICEcandidate", {
        sender: socket.user,
        rtcMessage: rtcMessage,
   });
   });
   });
   };

    // ### Mendapatkan Instance Socket.IO
   // Fungsi `getIO` digunakan untuk mendapatkan instance Socket.IO yang telah diinisialisasi.

   module.exports.getIO = () => {
     if (!IO) {
     throw Error("IO not initialized.");
   } else {
    return IO;
   }
   };

   
   
- [Features](#features)

## Installation
- [Prerequisites](#prerequisites)
- > Catatan: Pastikan Anda sudah memiliki React Native dan alat-alat yang diperlukan diinstal di komputer Anda sebelum memulai. Anda juga perlu menginstal Node.js dan npm (Node Package Manager).
- [Getting Started](#getting-started)

> **Clone Repositori**: Pertama, clone repositori "CallingApps" ke komputer Anda dengan menjalankan perintah berikut dalam terminal:

> ```bash
> git clone https://github.com/nanangNSL/callingapps.git
> ```

> **Pindah ke Direktori Proyek**: Masuk ke direktori proyek dengan menjalankan perintah:

> ```bash
> cd callingapps
> ```

> **Instal Dependencies**: Anda perlu menginstal semua dependensi proyek dengan menjalankan perintah npm install atau yarn install:

> ```bash
> npm install
> # atau jika Anda menggunakan Yarn
> yarn install
> ```

> **Konfigurasi WebRTC (jika diperlukan)**: Konfigurasi WebRTC bisa berbeda-beda tergantung pada proyek Anda dan berbagai faktor lainnya. Pastikan Anda telah mengikuti instruksi konfigurasi WebRTC yang mungkin diberikan dalam dokumentasi proyek atau README.

> **Menjalankan Aplikasi**: Setelah menginstal dependensi dan melakukan konfigurasi yang diperlukan, Anda dapat menjalankan aplikasi React Native dengan perintah:

> ```bash
> npx react-native run-android
> # atau untuk iOS
> npx react-native run-ios
> ```

> Pastikan emulator atau perangkat fisik Anda terhubung dan terdeteksi dengan baik.

> **Menjalankan Server Backend (jika diperlukan)**: Jika aplikasi Anda memerlukan server backend, pastikan Anda juga menjalankannya sesuai dengan instruksi yang diberikan dalam README server.
>  ```bash
> cd server
> # install depedency
> npm install
> # Jalankan
> npm start
> ```


## Usage
- [Configuration](#configuration)
- [Examples](#examples)

## Contributing
- [Issues](#issues)
- [Pull Requests](#pull-requests)
- [Code of Conduct](#code-of-conduct)

## License
- [License Information](#license-information)
