# Priya — AI-Powered Navigation Assistant

An AI-powered Android application concept designed to support blind and visually impaired users with environmental awareness, voice interaction, and reading text aloud.

## 🌟 Overview

Priya uses the smartphone camera and Google's Gemini 2.5 Flash AI model to analyze images of the user's surroundings and provide spoken descriptions. The project is designed with accessibility and hands-free interaction in mind.

## ✨ Key Features

- **Navigation Assistance:** Spoken descriptions of the surrounding environment.
- **AI Assistant:** Ask questions about what the camera sees using voice or text.
- **Text Reading:** Read text from signs, books, documents, and other images aloud.
- **Voice Output:** Android Text-to-Speech (TTS) provides audio responses.
- **Voice Input:** Speech recognition enables hands-free interaction.
- **Gesture Controls:** Touch gestures support accessible interaction.

## 🛠️ Technology Stack

| Technology | Purpose |
|---|---|
| Kotlin | Android application development |
| Jetpack Compose | User interface |
| CameraX | Camera access and image capture |
| Google Gemini 2.5 Flash | Image understanding and AI responses |
| Android Text-to-Speech | Spoken output |
| Android SpeechRecognizer | Voice input |
| Gradle | Build system |

## 🏗️ Architecture

1. The smartphone camera captures an image.
2. The image is prepared for AI analysis.
3. The Gemini API processes the image and the user's question or task.
4. The response is converted into speech using Android Text-to-Speech.

## 📋 Project Status

The project report is available in this repository. The Android application source code and build instructions will be added when they are ready.

## 🔐 API Key Security

The Gemini API requires an API key. Do not commit API keys, passwords, signing keys, or other secrets to this public repository.

## ⚠️ Safety Notice

This project is intended as an assistive technology project. AI-generated descriptions may be incorrect, incomplete, or delayed. Do not rely on the application as the sole means of avoiding obstacles or navigating traffic. It is not a replacement for a white cane, guide dog, or other established mobility support.

## 📄 Project Documentation

See [PROJECT REPORT](report(2).md) for the technical project report.

## 👩‍💻 Author

**Priyanka Meenkeri**

---

*Built to explore how AI and mobile accessibility technology can support greater independence.*
