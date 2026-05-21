# Calmify – Mental Wellness Android App

**CS 309 – Spring 2026 | Team 1_sb_4**

Calmify is a full-stack mental wellness application that connects users with licensed counsellors, supports daily habit tracking, and provides AI-assisted emotional support. The backend is a Spring Boot REST + WebSocket server; the frontend is a native Android application.

---

## Table of Contents

- [Project Overview](#project-overview)
- [Features](#features)
- [Architecture](#architecture)
- [Tech Stack](#tech-stack)
- [Repository Structure](#repository-structure)
- [Prerequisites](#prerequisites)
- [Getting Started](#getting-started)
  - [Backend Setup](#backend-setup)
  - [Frontend Setup](#frontend-setup)
- [API Reference](#api-reference)
- [CI/CD Pipeline](#cicd-pipeline)
- [Team Members](#team-members)

---

## Project Overview

Calmify helps users manage their mental wellbeing through a structured set of tools: daily mood check-ins, routine and sleep tracking, prescription reminders, worry journaling, and one-on-one chat with counsellors. Administrators can manage accounts and assign counsellors to users. An integrated Gemini AI chat provides 24/7 supportive conversation when a counsellor is unavailable.

---

## Features

| Role | Key Capabilities |
|---|---|
| **User** | Sign up / login, daily check-in, sleep & routine tracking, prescription reminders, worry notes, AI chat, real-time messaging with counsellor, appointment booking, task management |
| **Counsellor** | View assigned users, manage appointments, assign tasks & prescriptions, share notes, access counsellor profile |
| **Admin** | Full user management (create, edit, deactivate), assign counsellors to users |

Other highlights:

- Real-time bidirectional chat powered by WebSockets (STOMP over SockJS)
- Push-style notifications via WebSocket broadcasts
- Profile picture upload and management
- Swagger UI for interactive API exploration
- GitLab CI/CD with separate Maven and Android build/test pipelines

---

## Architecture

```
╔══════════════════════════════════════════════════════════════════╗
║                    FRONTEND  (Android App)                       ║
║                                                                  ║
║  ┌─────────────────────┐  ┌──────────────────┐                  ║
║  │       Views         │  │    App Logic      │                  ║
║  │                     │  │                  │                  ║
║  │ Login / Signup      │  │ Handles UI events│                  ║
║  │ Home / Landing      │  │ Validates input  │                  ║
║  │ Profile / Edit      │  │ Manages session  │                  ║
║  │ Counsellor Home     │  │ (SharedPrefs)    │                  ║
║  │ Admin Dashboard     │  │ Triggers API     │                  ║
║  │ Chat (Real-time)    │  │ calls            │                  ║
║  │ AI Chat             │  └────────┬─────────┘                  ║
║  │ Check-In            │           │                             ║
║  │ Routine Tracker     │  ┌────────▼──────────────────────────┐ ║
║  │ Sleep Tracker       │  │       Server Request Layer        │ ║
║  │ Prescriptions       │  │                                   │ ║
║  │ Tasks / Assign      │  │  Volley Request Queue (REST)      │ ║
║  │ Appointments        │  │  OkHttp (multipart file upload)   │ ║
║  │ Worry Notes         │  │  STOMP over WebSocket (chat)      │ ║
║  │ Shared Notes        │  │  JSON Object / Array parsing      │ ║
║  │ User List (Admin)   │  └────────────────────┬──────────────┘ ║
║  └─────────────────────┘                       │                ║
║                                                │                ║
║  ┌──────────────────────────────────────┐      │                ║
║  │           Local Model                │      │                ║
║  │  User · Notes · Profile · Settings  │      │                ║
║  │  CounsellorStatus · AvatarHelper    │      │                ║
║  └──────────────────────────────────────┘      │                ║
╚════════════════════════════════════════════════╪════════════════╝
                          REST / WebSocket (HTTP:8080 / WS:8080)
                 ┌──────────────── ◄──►────────────────┐
                 │           Request / Response          │
╔════════════════╪══════════════════════════════════════╪══════════╗
║                ▼    BACKEND  (Spring Boot :8080)                 ║
║                                                                  ║
║  ┌─────────────────────────────────────────────────────────┐    ║
║  │                      Controllers                         │    ║
║  │                                                         │    ║
║  │  UserController · UserAdminController                   │    ║
║  │  CounsellorProfileController · AssignmentController     │    ║
║  │  AppointmentController · TaskController                 │    ║
║  │  NoteController · AiChatController                      │    ║
║  │  ChatController (REST) · ChatServer (WebSocket/STOMP)   │    ║
║  │  NotificationController · NotificationWebSocket         │    ║
║  │  SleepController · RoutineController                    │    ║
║  │  PrescriptionController                                 │    ║
║  └──────────────────────────┬──────────────────────────────┘    ║
║                             │                                    ║
║  ┌──────────────────────────▼──────────────────────────────┐    ║
║  │                       Services                           │    ║
║  │                                                         │    ║
║  │  UserService · CounsellorProfileService                 │    ║
║  │  AppointmentService · TaskService · NoteService         │    ║
║  │  AiChatService (Gemini API) · ChatService               │    ║
║  │  NotificationService · RoutineService                   │    ║
║  │  PrescriptionService · SleepService                     │    ║
║  └──────────────────────────┬──────────────────────────────┘    ║
║                             │                                    ║
║  ┌──────────────────────────▼──────────────────────────────┐    ║
║  │                      Models (Entities)                   │    ║
║  │                                                         │    ║
║  │  User (roles: USER · COUNSELLOR · ADMIN)                │    ║
║  │  CounsellorProfile · UserCounsellorAssignment           │    ║
║  │  Appointment · Task · Note · ChatMessage                │    ║
║  │  Notification · SleepRecord · Routine · RoutineCheckIn  │    ║
║  │  Prescription · MedicationCheckIn · AiChatMessage       │    ║
║  │  DailyCheckIn                                           │    ║
║  └──────────────────────────┬──────────────────────────────┘    ║
║                             │                                    ║
║  ┌──────────────────────────▼──────────────────────────────┐    ║
║  │                     Repositories                         │    ║
║  │         (Spring Data JPA — all extend JpaRepository)    │    ║
║  │                                                         │    ║
║  │  UserRepository · CounsellorProfileRepository           │    ║
║  │  AssignmentRepository · AppointmentRepository           │    ║
║  │  TaskRepository · NoteRepository · ChatMessageRepository│    ║
║  │  NotificationRepository · SleepRepository               │    ║
║  │  RoutineRepository · RoutineCheckInRepository           │    ║
║  │  PrescriptionRepository · MedicationCheckInRepository   │    ║
║  │  AiChatMessageRepository · DailyCheckInRepository       │    ║
║  └──────────────────────────┬──────────────────────────────┘    ║
║                             │  JPA / Hibernate (MySQL dialect)  ║
╚═════════════════════════════╪════════════════════════════════════╝
                              │
              ┌───────────────▼────────────────┐
              │    MySQL Database (:3306)       │
              │                                │
              │  users · counsellor_profiles   │
              │  assignments · appointments    │
              │  tasks · notes · chat_messages │
              │  notifications · sleep_records │
              │  routines · routine_check_ins  │
              │  prescriptions · ai_chat_msgs  │
              │  daily_check_ins               │
              └───────────────┬────────────────┘
                              │
              ┌───────────────▼────────────────┐
              │    External: Gemini AI API      │
              │  (called by AiChatService)      │
              └────────────────────────────────┘

Legend:  ──►  Request   ◄──  Response   ══  System boundary
```

---

## Tech Stack

### Backend
| Technology | Version | Purpose |
|---|---|---|
| Java | 17 | Language |
| Spring Boot | 3.1.4 | Application framework |
| Spring Data JPA + Hibernate | — | ORM / database layer |
| Spring WebSocket (STOMP) | — | Real-time messaging |
| MySQL | 8.x | Persistent storage |
| SpringDoc / Swagger UI | 2.5.0 | API documentation |
| Lombok | — | Boilerplate reduction |
| Rest-Assured + JUnit 4 | — | Integration testing |
| Maven | — | Build tool |

### Frontend
| Technology | Version | Purpose |
|---|---|---|
| Android (Java) | compileSdk 34 / minSdk 24 | Native application |
| Volley | 1.2.1 | HTTP networking |
| OkHttp | 4.11.0 | HTTP client (multipart uploads) |
| STOMP Protocol Android | 1.6.6 | WebSocket / STOMP client |
| Java-WebSocket | 1.5.4 | Raw WebSocket support |
| RxJava 2 | 2.2.21 | Reactive streams for STOMP |
| Glide | 4.16.0 | Image loading and caching |
| Espresso | 3.5.1 | UI testing |
| Gradle | — | Build tool |

---

## Repository Structure

```
1_sb_4/
├── Backend/
│   └── Roundtrip 1/
│       └── springboot_example/
│           ├── pom.xml
│           └── src/main/java/onetoone/
│               ├── Main.java
│               ├── Users/            # User entity, auth, roles, admin
│               ├── Counsellors/      # Counsellor profiles
│               ├── Assignments/      # User-counsellor mapping
│               ├── Appointments/     # Booking system
│               ├── Tasks/            # Counsellor-assigned tasks
│               ├── Notes/            # Shared / private notes
│               ├── AiChat/           # Gemini AI integration
│               ├── realtime_chat/    # WebSocket chat (STOMP)
│               ├── Notification/     # Push-style notifications
│               ├── SleepTracker/     # Sleep logging
│               ├── RoutineTracker/   # Daily routine check-ins
│               └── Prescription/     # Medication management
├── Frontend/
│   └── AndroidExample/
│       └── app/src/main/java/com/example/androidexample/
│           ├── ApiConstants.java     # All API endpoint URLs
│           ├── LoginActivity.java
│           ├── SignUpActivity.java
│           ├── HomeActivity.java
│           ├── CounselorHomeActivity.java
│           ├── AdminDashboardActivity.java
│           ├── ChatActivity.java     # Real-time counsellor chat
│           ├── AIChatActivity.java   # Gemini AI chat
│           ├── CheckInActivity.java
│           ├── RoutineTrackerActivity.java
│           ├── PrescriptionsActivity.java
│           ├── WorryNotes.java
│           ├── TasksOverview.java
│           └── ...
├── Experiments/                      # Individual team member experiments
├── Documents/                        # Javadoc and design documents
├── .gitlab-ci.yml
└── README.md
```

---

## Prerequisites

### Backend
- Java 17+
- Maven 3.8+
- MySQL 8.x running and accessible
- (Optional) A Gemini API key from [Google AI Studio](https://aistudio.google.com/app/apikey)

### Frontend
- Android Studio Hedgehog (or later)
- Android SDK with API level 34 installed
- A physical device or emulator running Android 7.0+ (API 24+)

---

## Getting Started

### Backend Setup

1. **Clone the repository**

   ```bash
   git clone https://git.las.iastate.edu/cs309/2026spring/1_sb_4.git
   cd 1_sb_4
   ```

2. **Configure the database**

   Edit `Backend/Roundtrip 1/springboot_example/src/main/resources/application.properties`:

   ```properties
   spring.datasource.url=jdbc:mysql://<your-host>:3306/<your-db>?useSSL=false&allowPublicKeyRetrieval=true&serverTimezone=UTC
   spring.datasource.username=<your-username>
   spring.datasource.password=<your-password>
   ```

3. **(Optional) Set your Gemini API key**

   ```properties
   gemini.api.key=<your-gemini-api-key>
   ```

4. **Build and run**

   ```bash
   cd "Backend/Roundtrip 1/springboot_example"
   mvn package -DskipTests
   java -jar target/*.jar
   ```

   The server starts on `http://0.0.0.0:8080`.

5. **Explore the API**

   Open your browser at `http://localhost:8080/swagger-ui-custom.html` for the interactive Swagger UI.

   The raw OpenAPI spec is available at `http://localhost:8080/api-docs`.

6. **Run backend tests**

   ```bash
   mvn test
   ```

---

### Frontend Setup

1. **Set the backend URL**

   Open `Frontend/AndroidExample/app/src/main/java/com/example/androidexample/ApiConstants.java` and update:

   ```java
   public static final String BASE_URL    = "http://<your-server-ip>:8080";
   public static final String WS_BASE_URL = "ws://<your-server-ip>:8080";
   ```

2. **Open in Android Studio**

   - File → Open → select the `Frontend/AndroidExample` directory.
   - Let Gradle sync complete.

3. **Build and run**

   - Connect a device or start an emulator (API 24+).
   - Click **Run ▶** or use:

   ```bash
   cd Frontend/AndroidExample
   ./gradlew installDebug
   ```

4. **Run unit and instrumented tests**

   ```bash
   ./gradlew test              # JVM unit tests
   ./gradlew connectedCheck    # Instrumented tests (requires connected device)
   ```

---

## API Reference

Full interactive documentation is served by the running backend at `/swagger-ui-custom.html`.

Key endpoint groups:

| Prefix | Description |
|---|---|
| `/users/**` | Registration, login, profile management |
| `/api/admin/**` | Admin user management |
| `/api/counsellors/**` | Counsellor profiles and search |
| `/api/assignments/**` | User–counsellor assignments |
| `/api/appointments/**` | Appointment booking and management |
| `/api/chat/**` | Chat history (REST); real-time via WebSocket `/ws/chat/{senderId}/{receiverId}` |
| `/api/users/{id}/notes` | Notes CRUD |
| `/prescriptions/**` | Prescription and medication check-ins |
| `/sleep/**` | Sleep record logging |
| `/routines/**` | Routine tracking and check-ins |
| `/tasks/**` | Task assignment and status |
| `/ai-chat/**` | Gemini AI conversation |
| `/notifications/**` | Notification management; push via WebSocket |

---

## CI/CD Pipeline

The `.gitlab-ci.yml` defines five stages that run automatically on pushes to `main`:

| Stage | Job | Trigger condition |
|---|---|---|
| `mavenbuild` | Build Spring Boot JAR (`mvn package`) | Any change under `Backend/` |
| `maventest` | Run backend tests (`mvn test`) | Any change under `Backend/` |
| `mavendeploy` | Deploy JAR to production server via `systemctl` | Any change under `Backend/` |
| `androidbuild` | Build Android app (`./gradlew build`) | Any change under `Frontend/` |
| `androidtest` | Run Android unit tests (`./gradlew test`) | Any change under `Frontend/` |

Runners required: a `springboot_tag` runner (Java 17, Maven) and an `android_tag` runner (Docker image `afirefly/android-ci:java17`).

---

## Team Members

| Name | GitLab Username | Primary Role |
|---|---|---|
| Boudhayan | Boudhayan | Backend / Spring Boot |
| Nakshatra | nakshatra | Backend / WebSocket |
| Shrey | shreyp | Backend / Spring Boot |
| Ayrneto | ayrneto | Android Frontend |
