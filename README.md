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
   

### Komponen Utama: `App`

- `import React, { useEffect, useState, useRef } from 'react';`: Mengimpor pustaka React Native dan hooks yang diperlukan.

- `import { ... } from 'react-native';`: Mengimpor komponen-komponen React Native seperti `Platform`, `KeyboardAvoidingView`, dan lainnya.

- `import SocketIOClient from 'socket.io-client';`: Mengimpor klien Socket.IO untuk berkomunikasi dengan server.

- `import { ... } from 'react-native-webrtc';`: Mengimpor komponen-komponen React Native WebRTC untuk mengelola panggilan video.

- `import InCallManager from 'react-native-incall-manager';`: Mengimpor modul InCall Manager untuk mengatur panggilan masuk.

- `export default function App({}) { ... }`: Komponen utama aplikasi yang mengendalikan logika panggilan video dan tampilan layar.

### State Variables

- `localStream`: State yang menyimpan stream video lokal dari perangkat pengguna.

- `remoteStream`: State yang menyimpan stream video dari pengguna lain.

- `type`: State yang menentukan tipe tampilan layar (Join, Outgoing Call, Incoming Call, WebRTC Room).

- `callerId`: State yang menyimpan ID panggilan pengguna saat ini.

- `otherUserId`: Ref yang digunakan untuk menyimpan ID pengguna lain yang akan dihubungi.

- `socket`: Objek Socket.IO yang digunakan untuk berkomunikasi dengan server.

- `localMicOn` dan `localWebcamOn`: State yang mengontrol status mikrofon dan kamera lokal.

- `peerConnection`: Ref yang menyimpan instance RTCPeerConnection.

- `remoteRTCMessage`: Ref yang menyimpan pesan RTC dari pengguna lain.

### Koneksi Socket.IO

- `socket`: Membuat koneksi Socket.IO ke server dengan `callerId` sebagai query parameter.

### Inisialisasi Media Devices

- Menggunakan `mediaDevices` untuk mengakses perangkat media (kamera dan mikrofon).

- Mendapatkan stream video lokal dari perangkat pengguna.

### RTCPeerConnection

- Membuat instance `RTCPeerConnection` untuk mengelola koneksi WebRTC.

- Menggunakan `peerConnection.onaddstream` untuk mengatur stream video dari pengguna lain.

- Menggunakan `peerConnection.onicecandidate` untuk mengirim kandidat ICE.

### Event Listeners Socket.IO

- Mendengarkan peristiwa seperti 'newCall', 'callAnswered', dan 'ICEcandidate' dari server.

### Layar Komponen

- Terdapat komponen-komponen layar seperti 'JoinScreen', 'OutgoingCallScreen', 'IncomingCallScreen', dan 'WebrtcRoomScreen' yang ditampilkan berdasarkan nilai `type`.

### Fungsi-fungsi Tambahan

- `sendICEcandidate()`: Mengirim kandidat ICE ke pengguna lain melalui Socket.IO.

- `processCall()`: Membuat dan mengirim penawaran panggilan kepada pengguna lain.

- `processAccept()`: Menerima panggilan dan mengirim jawaban kepada pengguna lain.

- `answerCall()`: Mengirim pesan bahwa panggilan telah dijawab.

- `sendCall()`: Mengirim permintaan panggilan kepada pengguna lain.

### Switch Statement

- Menentukan tampilan layar berdasarkan nilai `type`.

# Dokumentasi Aplikasi Video Call React Native

Aplikasi video call ini memanfaatkan teknologi WebRTC (Web Real-Time Communication) dan React Native untuk memungkinkan pengguna melakukan panggilan video real-time. Dokumentasi ini akan membahas cara kerja WebRTC dalam konteks aplikasi ini.

## Pengenalan

Aplikasi ini mengintegrasikan WebRTC ke dalam React Native menggunakan modul React Native WebRTC dan memungkinkan pengguna untuk:

- Bergabung dengan panggilan video.
- Membuat panggilan video keluar.
- Menerima panggilan video masuk.
- Berkomunikasi secara real-time melalui video dan audio.

## Komponen Utama: `App`

`App` adalah komponen utama aplikasi yang mengendalikan logika panggilan video dan tampilan layar.

```javascript
// Import library yang diperlukan
import React, { useEffect, useState, useRef } from 'react';
import { ... } from 'react-native';
import SocketIOClient from 'socket.io-client';
import { ... } from 'react-native-webrtc';
import InCallManager from 'react-native-incall-manager';

export default function App({}) {
  // State Variables dan Pengaturan Awal
  // ...

  // Koneksi Socket.IO
  // ...

  // Inisialisasi Media Devices
  // ...

  // RTCPeerConnection
  // ...

  // Event Listeners Socket.IO
  // ...

  // Layar Komponen
  // ...

  // Fungsi-fungsi Tambahan
  // ...
}
```

Cara Kerja WebRTC
-----------------

### 1\. Membuat Peer Connection

Pertama, kita membuat instance RTCPeerConnection untuk mengelola koneksi WebRTC. Konfigurasi dan server STUN ditentukan pada tahap ini.

javascriptCopy code

`const peerConnection = useRef(
  new RTCPeerConnection({
    iceServers: [
      {
        urls: 'stun:stun.l.google.com:19302',
      },
      // ...
    ],
  }),
);`

### 2\. Mendapatkan Media

Aplikasi menggunakan `mediaDevices` untuk mengakses kamera dan mikrofon pengguna. Ini memungkinkan aplikasi untuk mengambil aliran video dan audio lokal dari perangkat pengguna.

javascriptCopy code

`mediaDevices
  .getUserMedia({
    audio: true,
    video: {
      mandatory: {
        minWidth: 500,
        minHeight: 300,
        minFrameRate: 30,
      },
      // ...
    },
  })
  .then(stream => {
    // Mengatur aliran video lokal
    peerConnection.current.addStream(stream);
  })
  .catch(error => {
    // Handle error
  });`

### 3\. Menangani Peristiwa

Pada tahap ini, event listener digunakan untuk menangani peristiwa seperti saat ada aliran video tambahan (penerima panggilan) dan ketika kandidat ICE tersedia.

javascriptCopy code

`peerConnection.current.onaddstream = event => {
  // Menampilkan aliran video dari pengguna lain
};

peerConnection.current.onicecandidate = event => {
  if (event.candidate) {
    // Mengirim kandidat ICE kepada pengguna lain
  } else {
    console.log('End of candidates.');
  }
};`

### 4\. Mengirim dan Menerima Pesan

Pesan yang mengandung informasi panggilan, jawaban, dan kandidat ICE dikirim dan diterima melalui Socket.IO.

-   `processCall()`: Membuat penawaran panggilan dan mengirimnya ke pengguna lain.
-   `processAccept()`: Menerima panggilan dan mengirim jawaban kepada pengguna lain.
-   `sendICEcandidate()`: Mengirim kandidat ICE kepada pengguna lain.

### 5\. Menampilkan Video

Aliran video lokal dan video dari pengguna lain ditampilkan menggunakan komponen RTCView dari React Native WebRTC.

javascriptCopy code

`{localStream ? (
  <RTCView
    objectFit={'cover'}
    style={{ flex: 1, backgroundColor: '#050A0E' }}
    streamURL={localStream.toURL()}
  />
) : null}
{remoteStream ? (
  <RTCView
    objectFit={'cover'}
    style={{
      flex: 1,
      backgroundColor: '#050A0E',
      marginTop: 8,
    }}
    streamURL={remoteStream.toURL()}
  />
) : null}`

Dengan integrasi WebRTC dalam kode aplikasi ini, pengguna dapat melakukan panggilan video dan audio real-time dengan mudah. Peer Connection mengelola koneksi, Socket.IO digunakan untuk komunikasi real-time, dan RTCView digunakan untuk menampilkan aliran video. Semua ini bersama-sama menciptakan pengalaman panggilan video yang interaktif dalam aplikasi Anda.


   
   
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
