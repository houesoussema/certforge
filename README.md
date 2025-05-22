# CertForge

> CertForge is a self-serve, mobile-first learning platform that transforms structured AI-generated content into a personalized certification mastery system — seamlessly integrated with Google Drive, Obsidian, and your daily workflow.

## Overview

Built for serious professionals, CertForge eliminates passive content consumption and replaces it with structured microlearning, adaptive feedback loops, and user-owned data. Whether you're preparing for PMP, ISO standards, IT security, or safety certifications, CertForge becomes your certification operating system.

## Core Value Propositions

1. **User-Controlled, AI-Generated Content**  
   Professionals generate flashcards using Gemini, ChatGPT, or similar tools — following a public YAML/JSON schema. CertForge reads this content directly from Drive or Obsidian-compatible folders, turning structure into actionable quizzes.

2. **No Vendor Lock-In**  
   You own your content. CertForge reads your data — it doesn't trap it. You organize, back it up, and modify it freely with any text editor.

3. **Structured Knowledge Organization**  
   Your YAML/JSON flashcards are parsed, validated, and auto-sorted into categories (e.g., PMP > Risk Management > Qualitative Analysis). CertForge mirrors your folder taxonomy and tags for intuitive navigation.

4. **Adaptive Learning Engine**  
   Every quiz feeds into CertForge's analytics core:
   - Tracks right/wrong by category
   - Identifies weak zones
   - Adapts repetition using spaced learning or Leitner systems
   - Measures retention decay to prompt strategic reviews

5. **Mobile-First Execution**  
   Built natively for Android, CertForge is optimized for speed, offline mode, and bite-sized sessions. It's not just portable—it's frictionless. Perfect for 3-minute waits, 10-minute commutes, or 30-minute deep dives.

## Project Structure

This repository contains the Android application for CertForge, built with Kotlin and Jetpack Compose. The project follows MVVM architecture with Clean Architecture principles.

### Documentation

- [Architecture Overview](docs/architecture_overview.md)
- [UI Design System](docs/ui_design_system.md)
- [Development Plan](docs/development_plan.md)
- [Development Environment](docs/development_environment.md)
- [Prototype Specification](docs/prototype_specification.md)

## Getting Started

### Prerequisites

- Android Studio Arctic Fox (2021.3.1) or higher
- JDK 11 or higher
- Google Cloud Console account with Drive API enabled

### Setup

1. Clone this repository
2. Open the project in Android Studio
3. Create a `local.properties` file with your Google API credentials
4. Sync the project with Gradle files
5. Run the app on an emulator or physical device

## Development

Please refer to the [Development Environment](docs/development_environment.md) document for detailed instructions on setting up your development environment, coding conventions, and workflow.

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## Acknowledgments

- The CertForge team
- All contributors to this project
