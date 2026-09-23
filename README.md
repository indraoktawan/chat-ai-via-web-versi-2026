# chat-ai-via-web-versi-2026

```html
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Enterprise AI Chat - Terminal Presentation</title>
    <style>
        /* Desain ala Terminal Developer / Hacker */
        body {
            background-color: #0d1117; /* Warna gelap khas GitHub/Terminal */
            color: #58a6ff; /* Warna teks biru terang */
            font-family: 'Courier New', Courier, monospace;
            padding: 40px;
            margin: 0;
            overflow-x: hidden;
            display: flex;
            justify-content: center;
        }

        .terminal-window {
            width: 100%;
            max-width: 800px;
            background: #161b22;
            border: 1px solid #30363d;
            border-radius: 8px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.5);
            padding: 20px;
            min-height: 80vh;
        }

        /* Area tempat teks akan muncul */
        #typing-area {
            white-space: pre-wrap; /* Menjaga spasi dan baris baru (enter) */
            line-height: 1.6;
            font-size: 15px;
            color: #c9d1d9; /* Warna teks abu-abu terang */
        }

        /* Animasi kursor berkedip di akhir teks */
        .cursor {
            display: inline-block;
            width: 10px;
            height: 18px;
            background-color: #00ff00; /* Kursor hijau */
            vertical-align: middle;
            animation: blink 1s step-end infinite;
        }

        @keyframes blink {
            0%, 100% { opacity: 1; }
            50% { opacity: 0; }
        }
    </style>
</head>
<body>

    <div class="terminal-window">
        <!-- Teks akan diinjeksi ke dalam span ini -->
        <span id="typing-area"></span><span class="cursor"></span>
    </div>

    <script>
        // Teks dokumentasi Markdown milikmu
        const markdownText = `# 🌐 Enterprise AI Chat & Complaint Kit

> Production-ready Firebase Genkit boilerplate designed for global enterprise web applications. Powered by Google Gemini and Firebase.

[![Built with Firebase Genkit](https://img.shields.io/badge/Genkit-Firebase-orange?style=flat-square&logo=firebase)](https://firebase.google.com/)
[![Powered by Gemini](https://img.shields.io/badge/AI-Gemini%203.5%20Flash-blue?style=flat-square&logo=google)](https://ai.google.dev/)
[![License: Commercial EULA](https://img.shields.io/badge/License-Commercial%20EULA-green?style=flat-square)](LICENSE)

---

## 🚀 Overview

Enterprise AI Chat & Complaint Kit is a robust, modular developer toolkit built to help organizations rapidly deploy intelligent customer interaction and automated ticketing systems. 

Developed with clean architecture standards by PT BERTANI TEKNOLOGI INTELEGENSI 
this kit eliminates weeks of boilerplate setup by providing secure backend flows, dynamic prompt configurations, and automated Firestore database integration out of the box.

---

## ✨ Key Features

- 🧠 Google Genkit & Gemini 3.5 Flash Integration: High-performance, low-latency AI response generation optimized for corporate environments.
- ⚙️ Centralized Prompt Management: Easily adjust AI personas, system instructions, and classification rules directly from a single configuration file (\`prompts.ts\`).
- 🗄️ Automated Ticket Storage: Seamlessly analyzes customer complaints using AI and automatically logs structured tickets (category, priority, suggested actions) into Google Cloud Firestore.
- 💬 Clean Web Frontend: Modern, responsive chat UI featuring sleek glassmorphism styling and native Web Speech (Text-to-Speech) capabilities.
- 🔒 Enterprise Security: Built-in secret management using Google Cloud Secret Manager to protect sensitive API keys.

---

## 📂 Project Structure

\`\`\`text
├── functions/
│   ├── src/
│   │   ├── config/prompts.ts       # Centralized AI system prompts
│   │   ├── flows/
│   │   │   ├── consultationFlow.ts # Core AI chat consultation logic
│   │   │   └── complaintFlow.ts    # AI analysis + automated Firestore logging
│   │   └── index.ts                # Cloud Functions entry point
│   ├── package.json
│   └── tsconfig.json
├── web/
│   ├── index.html                  # Responsive chat & dashboard UI
│   └── assets/
│       ├── css/style.css           # Modern aesthetic styles
│       └── js/app.js               # Frontend fetch logic & TTS handler
├── INTEGRATION.md                  # Detailed deployment guide
└── LICENSE                         # Commercial End User License Agreement
\`\`\`
`;

        // Pengaturan Animasi
        let index = 0;
        const typingSpeed = 15; // Kecepatan mengetik (dalam milidetik). Semakin kecil, semakin cepat.
        const typingArea = document.getElementById("typing-area");

        // Fungsi utama untuk menjalankan animasi mengetik
        function typeWriter() {
            if (index < markdownText.length) {
                // Menambahkan satu karakter setiap kali fungsi dipanggil
                typingArea.textContent += markdownText.charAt(index);
                index++;
                
                // Membuat layar otomatis bergerak/menggulir ke bawah (Auto-scroll) mengikuti teks
                window.scrollTo(0, document.body.scrollHeight);
                
                // Mengulang fungsi ini dengan jeda waktu yang ditentukan
                setTimeout(typeWriter, typingSpeed);
            }
        }

        // Memulai animasi saat halaman dimuat
        window.onload = typeWriter;
    </script>
</body>
</html>
```
