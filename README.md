# MindWatch Setup and Installation Guide

This guide explains how to run MindWatch on a computer for development or marking, how to build the Android app, and how the live version is deployed. The commands are for Windows using Git Bash, and they work the same way on macOS and Linux.

## 1. What you need

The following software must be installed before you start:

1. **Git**, to download the code.
2. **Python 3.12**, for the backend.
3. **Node.js 22** and **pnpm 10**, for the website. npm, which comes with Node.js, also works.
4. **Android Studio**, only if you want to build the Android app.

## 2. Download the code

Clone the repository and move into its folder:

```bash
git clone https://github.com/JacobSong1/MindWatch-IE08.git
cd MindWatch-IE08
```

## 3. Run the backend

The backend needs a database. For local use, a SQLite file is enough, because the app creates all of its tables automatically when it starts.

1. Create a Python virtual environment, which keeps the project's packages separate from the rest of the computer, and activate it:

   ```bash
   python -m venv .venv
   source .venv/Scripts/activate
   ```

   On macOS or Linux, the second command is `source .venv/bin/activate`.

2. Install the backend packages:

   ```bash
   pip install -r backend/requirements-dev.txt
   ```

3. Set the two settings the backend needs. `DATABASE_URL` tells it where the database is, and `INVITE_CODES` lists the codes that allow someone to sign up. Choose any code you like for local testing:

   ```bash
   export DATABASE_URL=sqlite:///./mindwatch-local.db
   export INVITE_CODES=local-test-code
   ```

4. Start the backend from the repository's top folder:

   ```bash
   uvicorn backend.main:app --reload --port 8000
   ```

5. Open http://localhost:8000/health in a browser. The reply should include `"status": "online"` and `"database": "ready"`. FastAPI also provides an interactive list of every API address at http://localhost:8000/docs. If `database` says `not_configured`, the `DATABASE_URL` setting was not picked up, and if it says `unavailable`, the database address is wrong or the database cannot be reached.

## 4. Run the website

1. Open a second terminal, move into the `frontend` folder and install the packages:

   ```bash
   cd frontend
   pnpm install
   ```

2. Create a file called `.env.local` in the `frontend` folder with this line, which switches on the account screens:

   ```
   VITE_ACCOUNTS_ENABLED=true
   ```

   The website looks for the backend at http://localhost:8000 by default. To point it at a different backend, add a second line such as `VITE_API_BASE_URL=https://mindwatch-ie08.onrender.com`.

3. Start the website:

   ```bash
   pnpm dev
   ```

4. Open the address that Vite prints, usually http://localhost:5173, and create an account using the invite code from Section D.3.

## 5. Load the sample account (optional)

The sample account lets visitors try the app with 30 real check-ins from the SSAQS study. With the backend's virtual environment active and the same `DATABASE_URL` set, run this from the repository's top folder:

```bash
python -m backend.scripts.seed_demo_user
```

The Welcome screen's sample option then gives each visitor their own private copy, which is deleted after one day. Running the command again refreshes the sample so that its last check-in is today.

## 6. Build and install the Android app

The Android app is the website packaged with Capacitor, so the website must be built first.

1. In `frontend/.env.local`, set `VITE_API_BASE_URL` to a backend the phone can reach. A real phone cannot reach `localhost` on your computer, so use either the deployed backend or your computer's local network address, for example `http://192.168.1.20:8000`. The Android emulator can reach your computer at `http://10.0.2.2:8000`, which the app uses automatically if no address is set.

2. Build the website and copy it into the Android project:

   ```bash
   pnpm build
   npx cap sync android
   ```

3. Open the Android project in Android Studio:

   ```bash
   npx cap open android
   ```

4. Connect an Android phone with USB debugging turned on, or start an emulator, and press **Run** in Android Studio. The app installs as **MindWatch**.

5. To use watch data, install the Fitbit app and Health Connect on the phone, pair the watch, and allow the Fitbit app to share steps, sleep and heart rate with Health Connect. MindWatch then asks for its own permission the first time the watch card is opened.

The website service runs `pnpm install` and `pnpm build`, and serves the `dist` folder. It is built with `VITE_ACCOUNTS_ENABLED=true` and with `VITE_API_BASE_URL` set to the backend's address, https://mindwatch-ie08.onrender.com.
