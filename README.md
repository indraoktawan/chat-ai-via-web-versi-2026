# chat-ai-via-web-versi-2026
CETAK BIRU SISTEM & ARSITEKTUR (SYSTEM BLUEPRINT)
AuraHealth - Sistem Konsultasi Klinis AI & Diagnosis Citra Medis
1. RINGKASAN EKSEKUTIF (EXECUTIVE SUMMARY)
AuraHealth adalah platform konsultasi medis berbasis kecerdasan buatan (Clinical AI Assistant) yang dirancang untuk menyediakan triase klinis multi-turn, pemeriksaan diagnostik visual citra medis (dermatologi, oftalmologi, luka), serta sintesis audio interaktif (Text-to-Speech).
Platform ini dibangun dengan arsitektur Google Cloud & Firebase, memisahkan logika antarmuka pengguna (Frontend SPA) dengan lapisan komputasi kecerdasan buatan di backend (Cloud Functions & Server-Side Gemini API) demi menjamin keamanan kunci API (Zero-Leakage Architecture via Google Cloud Secret Manager).
2. ARSITEKTUR TINGKAT TINGGI (HIGH-LEVEL ARCHITECTURE)
code
Code
+-------------------------------------------------------------------------------+
|                             CLIENT / USER LAYER                               |
|  +-------------------------------------------------------------------------+  |
|  | Modern Web Client (React 19 + Tailwind CSS + Lucide Icons + Vite)       |  |
|  | - Multi-Turn Clinical Chat Thread (Dr. Elena, Dr. Marcus, Dr. Maya, Rx) |  |
|  | - Clinical Vision Diagnosis Modal (Drag & Drop / High-Res Upload)       |  |
|  | - Patient Clinical Context Manager (Age, Sex, Rx, Allergies, History)  |  |
|  | - Voice Dictation (SpeechRecognition) & Spoken Playback (WebAudio/TTS) |  |
|  +-------------------------------------------------------------------------+  |
+-------------------------------------------------------------------------------+
                                      |
                     HTTPS / JSON Payload (Port 3000 / Cloud Run)
                                      v
+-------------------------------------------------------------------------------+
|                           BACKEND & PROXY RUNTIME                             |
|  +-------------------------------------------------------------------------+  |
|  | Express Proxy & Middleware Runtime (server.ts / Cloud Run / App Engine) |  |
|  | - Payload Body Parsing (50MB limit for high-res base64 medical scans)   |  |
|  | - CORS Security Headers & Health Check Monitor (/api/health)           |  |
|  | - Static Asset Delivery (/dist) + SPA Catch-all Routing                 |  |
|  +-------------------------------------------------------------------------+  |
+-------------------------------------------------------------------------------+
                                      |
                   Direct Invocation / Route Dispatching
                                      v
+-------------------------------------------------------------------------------+
|                    SERVERLESS CLOUD FUNCTIONS (functions/)                    |
|  +-------------------------------------------------------------------------+  |
|  | 1. aiConsultation (/aiConsultation, /api/aiConsultation)                |  |
|  |    -> File: functions/src/flows/ai-consultation.ts                      |  |
|  |    -> Preserves multi-turn chat history, role personas, triage levels   |  |
|  +-------------------------------------------------------------------------+  |
|  | 2. diagnoseImage (/diagnoseImage, /api/diagnoseImage)                   |  |
|  |    -> File: functions/src/flows/diagnose-image-.ts                      |  |
|  |    -> Multimodal Vision analysis, differential diagnoses & urgency      |  |
|  +-------------------------------------------------------------------------+  |
|  | 3. textToSpeech (/textToSpeech, /api/tts)                               |  |
|  |    -> File: functions/src/services/text-to-speech.ts                     |  |
|  |    -> Clinical voice synthesis using gemini-3.8-flash-lite-tts          |  |
|  +-------------------------------------------------------------------------+  |
+-------------------------------------------------------------------------------+
             |                                              |
      Secret Resolution                              AI Inference
             v                                              v
+-----------------------------+               +-------------------------------+
| GOOGLE SECRET MANAGER       |               | GOOGLE GEMINI MODELS          |
| Secret: Ai_API_KEY          |               | - gemini-3.5-flash (General)  |
| (firebase functions:secrets)|               | - gemini-3.1-pro-preview      |
| Injected securely at runtime|               | - gemini-3.1-flash-lite (Fast)|
+-----------------------------+               | - gemini-3.8-flash-lite-tts   |
                                              +-------------------------------+
3. STRUKTUR POHON PROYEK (DIRECTORY BLUEPRINT)
code
Text
your-project/
├── functions/                              # Backend Cloud Functions
│   ├── src/
│   │   ├── flows/
│   │   │   ├── ai-consultation.ts          # Logika percakapan multi-turn & triase
│   │   │   ├── diagnose-image-.ts          # Logika analisis citra medis (Gemini Vision)
│   │   │   └── diagnose-image.ts           # Re-export bridge
│   │   ├── services/
│   │   │   └── text-to-speech.ts           # Sintesis suara dokter (Gemini TTS)
│   │   └── index.ts                        # Entry point Cloud Functions API
│   ├── package.json                        # Dependensi backend (@google/genai)
│   └── tsconfig.json                       # Konfigurasi TypeScript backend
├── src/                                    # Frontend Web Application (React)
│   ├── components/
│   │   ├── ArchitectureModal.tsx           # Panduan arsitektur & Secret Manager
│   │   ├── BlueprintModal.tsx              # Modal cetak biru sistem interaktif
│   │   ├── ChatInterface.tsx               # Antarmuka chat konsultasi klinis
│   │   ├── Header.tsx                      # Navigasi & banner keamanan medis
│   │   ├── ModelSelector.tsx               # Pemilih model Gemini
│   │   ├── PatientProfileModal.tsx         # Manajemen konteks pasien
│   │   ├── SpecialistSelector.tsx          # Pemilih spesialis medis
│   │   └── VisionDiagnosisModal.tsx        # Modal diagnostik citra medis
│   ├── data/
│   │   └── sampleCases.ts                  # Kasus klinis bawaan (dermatitis, mata, luka)
│   ├── types/
│   │   └── index.ts                        # Definisi tipe TypeScript
│   ├── App.tsx                             # Komponen root aplikasi
│   ├── index.css                           # Desain Tailwind CSS & tema klinis
│   └── main.tsx                            # Titik masuk aplikasi React
├── firebase.json                           # Konfigurasi routing Firebase Hosting & Functions
├── server.ts                               # Server full-stack Express + Vite dev proxy
├── package.json                            # Dependensi aplikasi utama
├── BLUEPRINT.md                            # Cetak biru dokumentasi teknis
└── metadata.json                           # Metadata aplikasi AI Studio
4. SPESIFIKASI MODUL BACKEND & ALUR KERJA (BACKEND FLOWS)
Modul A: aiConsultation (functions/src/flows/ai-consultation.ts)
Tujuan: Menangani konsultasi medis percakapan berulang (multi-turn) dengan persona dokter spesifik dan triase klinis otomatis.
Peran Spesialis (System Instruction Personas):
clinical_triage (Dr. Elena Vance): Dokter Umum & Spesialis Triase Klinis. Menganalisis gejala akut, onset, durasi, skala nyeri (1-10), dan mengidentifikasi red flags.
dermatology (Dr. Marcus Chen): Dokter Spesialis Kulit & Kelamin. Menganalisis morfologi lesi (makula, papula, vesikel), kriteria ABCDE, dan diferensial dermatologi.
mental_health (Dr. Maya Patel): Psikolog Klinis & Spesialis Kesehatan Mental. Bahasa empatik, teknik grounding somatik (4-7-8, 5-4-3-2-1), serta deteksi krisis (988).
rapid_rx (RapidRx Guide): Penapisan interaksi obat cepat, indikasi, kontraindikasi, dan efek samping farmakologis.
Seleksi Model Dinamis:
gemini-3.5-flash: Standar untuk percakapan umum berimbang.
gemini-3.1-pro-preview: Analisis mendalam ketika riwayat medis kompleks atau prompt > 500 karakter.
gemini-3.1-flash-lite: Mode cepat (rapid response / RapidRx).
Modul B: diagnoseImage (functions/src/flows/diagnose-image-.ts)
Tujuan: Menganalisis foto klinis (ruam, luka, nodul, mata merah) menggunakan Gemini Vision Multimodal dengan output terstruktur strictly typed JSON Schema.
Skema Output JSON:
visualObservations: Array observasi objektif (warna, batas lesi, tekstur, distribusi).
anatomicRegion: Estimasi lokasi anatomis tubuh.
primaryImpression: Kesimpulan dugaan diagnosis utama.
differentialDiagnoses: Daftar diagnosis pembanding disertai tingkat kemungkinan (high, moderate, low) dan rasional klinis.
urgencyLevel: Tingkat kegawatan (EMERGENCY, URGENT, ROUTINE, SELF_CARE).
urgencyExplanation: Penjelasan medis mengapa tingkat kegawatan tersebut dipilih.
recommendedActions: Langkah-langkah tindak lanjut untuk pasien.
questionsForPhysician: Pertanyaan terarah untuk dibawa ke dokter fisik.
redFlags: Tanda bahaya yang mewajibkan ke IGD terdekat segera.
homeCareGuidance: Panduan perawatan rumahan mandiri yang aman.
clinicalDisclaimer: Penafian hukum medis resmi.
Modul C: textToSpeech (functions/src/services/text-to-speech.ts)
Tujuan: Mengubah respons teks dokter menjadi audio bersuara natural.
Model: gemini-3.8-flash-lite-tts dengan suara prebuilt (Kore, Puck) dan modalitas audio output base64.
Fallback: Otomatis dialihkan ke Web Speech API browser jika terjadi kendala jaringan backend.
5. SKEMA KONTRAK API (API CONTRACT SPECIFICATIONS)
1. Endpoint: POST /aiConsultation
Request Body (JSON):
code
JSON
{
  "message": "Saya mengalami sakit kepala berdenyut sebelah kanan sejak 2 hari lalu disertai mual.",
  "conversationHistory": [
    { "role": "user", "content": "Halo dok" },
    { "role": "model", "content": "Halo! Saya Dr. Elena Vance. Ada gejala apa yang Anda rasakan?" }
  ],
  "role": "clinical_triage",
  "model": "gemini-3.5-flash",
  "patientContext": {
    "age": "32",
    "gender": "female",
    "knownConditions": ["Migraine history"],
    "medications": ["Paracetamol 500mg prn"]
  }
}
Response Body (JSON 200 OK):
code
JSON
{
  "reply": "Terima kasih telah menceritakan gejalanya. Sakit kepala berdenyut unilateral disertai mual mengarah pada episode migrain...",
  "role": "clinical_triage",
  "modelUsed": "gemini-3.5-flash",
  "triageLevel": "routine",
  "followUpQuestions": [
    "Apakah Anda mengalami sensitivitas terhadap cahaya atau suara?",
    "Apakah terdapat gangguan penglihatan (aura) sebelum sakit kepala muncul?"
  ],
  "timestamp": "2026-09-23T20:05:00.000Z"
}
2. Endpoint: POST /diagnoseImage
Request Body (JSON):
code
JSON
{
  "imageBase64": "data:image/jpeg;base64,/9j/4AAQSkZJRg...",
  "mimeType": "image/jpeg",
  "bodyPartOrContext": "Lengan kanan bagian dalam",
  "symptomsPrompt": "Gatal hebat dan muncul bintil berair setelah berkebun kemarin.",
  "duration": "24 jam",
  "model": "gemini-3.5-flash"
}
Response Body (JSON 200 OK):
code
JSON
{
  "visualObservations": [
    "Plak eritematosa batas tegas dengan vesikel mikro multipel",
    "Pola lesi linier yang mengindikasikan kontak dengan tanaman",
    "Tanda ekskoriasi ringan tanpa nanah aktif"
  ],
  "anatomicRegion": "Volar surface of right forearm",
  "primaryImpression": "Dermatitis Kontak Alergi (Phytodermatitis)",
  "differentialDiagnoses": [
    {
      "condition": "Dermatitis Kontak Alergi",
      "likelihood": "high",
      "clinicalRationale": "Pola linier dengan vesikel berair pasca paparan tanaman sangat khas.",
      "typicalPresentation": "Eritema, vesikel, rasa gatal hebat."
    },
    {
      "condition": "Herpes Zoster",
      "likelihood": "low",
      "clinicalRationale": "Distribusi tidak mengikuti dermatom unilateral saraf sensorik tunggal.",
      "typicalPresentation": "Nyeri radikuler mendahului lesi vesikular berkelompok."
    }
  ],
  "urgencyLevel": "ROUTINE",
  "urgencyExplanation": "Kondisi superfisial stabil tanpa tanda keterlibatan sistemik atau selulitis berat.",
  "recommendedActions": [
    "Cuci area dengan air mengalir dan sabun lembut",
    "Hindari memecahkan gelembung/vesikel",
    "Konsultasikan ke dokter jika lesi meluas atau timbul demam"
  ],
  "questionsForPhysician": [
    "Apakah diperlukan kortikosteroid topikal potensi sedang?",
    "Bagaimana pencegahan hiperpigmentasi pasca inflamasi?"
  ],
  "redFlags": [
    "Demam tinggi > 38.5 C",
    "Bercak merah meluas cepat disertai rasa sangat hangat dan nyeri hebat (indikasi selulitis)",
    "Nanah purulen kental berbau"
  ],
  "homeCareGuidance": [
    "Kompres dingin selama 10-15 menit 3 kali sehari",
    "Gunakan lotion kalamin untuk meredakan gatal"
  ],
  "clinicalDisclaimer": "Hasil analisis AI ini adalah alat bantu skrining edukatif dan bukan merupakan diagnosis medis definitif.",
  "modelUsed": "gemini-3.5-flash",
  "timestamp": "2026-09-23T20:06:00.000Z"
}
3. Endpoint: POST /textToSpeech
Request Body (JSON):
code
JSON
{
  "text": "Halo, saya sarankan Anda beristirahat dan mengompres dingin area lengan tersebut.",
  "voiceName": "Kore",
  "speakingStyle": "Empathetic, clear, and reassuring physician voice"
}
Response Body (JSON 200 OK):
code
JSON
{
  "audioBase64": "UklGRiQAAABXQVZFZm10IBAAAAABAAEA...",
  "mimeType": "audio/wav",
  "voiceUsed": "Kore"
}
6. BLUEPRINT DATABASE & PENYIMPANAN DATA (FIRESTORE DATA MODEL)
Jika penyimpanan data persisten multi-user diaktifkan, berikut adalah skema koleksi Cloud Firestore yang telah distandardisasi:
code
Code
[Firestore Root]
├── users/ (Koleksi Profil Pengguna)
│   └── {userId}/
│       ├── profile: { name, email, age, gender, createdAt }
│       ├── clinicalHistory: { conditions: [], medications: [], allergies: [] }
│       │
│       ├── consultations/ (Sub-koleksi Sesi Konsultasi)
│       │   └── {consultationId}/
│       │       ├── role: "clinical_triage" | "dermatology" | ...
│       │       ├── status: "active" | "archived" | "referred"
│       │       ├── initialTriageLevel: "routine"
│       │       ├── createdAt: Timestamp
│       │       ├── updatedAt: Timestamp
│       │       │
│       │       └── messages/ (Sub-koleksi Pesan Chat)
│       │           └── {messageId}/
│       │               ├── sender: "user" | "model"
│       │               ├── text: string
│       │               ├── attachedImageRef: string (opsional)
│       │               ├── modelUsed: string
│       │               └── timestamp: Timestamp
│       │
│       └── imageDiagnoses/ (Sub-koleksi Analisis Gambar)
│           └── {diagnosisId}/
│               ├── storagePath: "users/{userId}/images/{id}.jpg"
│               ├── primaryImpression: string
│               ├── urgencyLevel: "EMERGENCY" | "URGENT" | "ROUTINE" | "SELF_CARE"
│               ├── structuredResult: Map<String, Any>
│               └── timestamp: Timestamp
│
└── auditLogs/ (Koleksi Log Keamanan & Kepatuhan Medis)
    └── {logId}/
        ├── userId: string
        ├── action: "CONSULTATION_INITIATED" | "IMAGE_DIAGNOSED"
        ├── riskDetected: boolean
        └── timestamp: Timestamp
7. TATA KELOLA KEAMANAN & PRIVASI (SECURITY & PRIVACY BLUEPRINT)
Zero Client Leakage of API Keys:
Seluruh panggilan ke Google Gemini SDK (@google/genai) dieksekusi di backend Node.js (server.ts & functions/src/).
Browser client tidak pernah memegang atau menginspeksi GEMINI_API_KEY.
Google Cloud Secret Manager:
Penyimpanan kunci rahasia dilakukan via CLI resmi:
code
Bash
firebase functions:secrets:set Ai_API_KEY
Cloud Functions otomatis menginjeksikan kunci pada runtime sebagai environment variable terenkripsi.
Pemberitahuan Keselamatan Klinis & Disclaimer:
Banner keselamatan aktif di setiap sesi.
Peringatan red-flag otomatis jika terdeteksi indikasi gawat darurat (911 / IGD).
Sanitasi Input & Kontrol Ukuran File:
Payload Express dibatasi maksimum 50MB untuk mencegah crash memori saat memproses gambar beresolusi tinggi.
8. PANDUAN PENGUJIAN & DEPLOYMENT (CI/CD DEPLOYMENT PIPELINE)
A. Pengujian Lokal dengan Firebase Emulator
code
Bash
# 1. Navigasi ke direktori functions & build
cd functions
npm install
npm run build

# 2. Jalankan Firebase Local Emulator
firebase emulators:start
B. Menjalankan Server Pengembangan Full-Stack
code
Bash
# Dari root project
npm run dev
# Server aktif di http://localhost:3000 dengan Vite middleware & Cloud Functions proxy
C. Build & Deploy ke Produksi
code
Bash
# 1. Build aplikasi web frontend
npm run build

# 2. Deploy Cloud Functions dan Firebase Hosting
firebase deploy --only functions,hosting
Dokumen ini merupakan cetak biru resmi arsitektur AuraHealth v1.0.0.
