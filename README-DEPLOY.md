# 3D Ludo Royale — Online Edition

This project keeps the original 3D Ludo Royale board, visual theme, camera controls, dice, sound effects, CPU players, and the special **Ai** dice-outcome selector. It adds online rooms using Firebase Realtime Database.

## Project files

- `index.html` — complete game; edit the Firebase Web App config in the `firebaseConfig` object.
- `firebase-database.rules.json` — Realtime Database security rules to paste into Firebase Console.

## How online rooms work

1. A player creates a room and becomes the host (Red).
2. Other players enter the six-character room code. Seats are assigned in order: Green, Yellow, Blue.
3. The host can start once at least two human players have joined. Empty seats become CPU players.
4. The host's browser runs the authoritative game logic and publishes the shared game state. Guests send turn actions to the host through Firebase.
5. Only the player whose turn it is gets the dice/token controls on their device. If that player's name is exactly `Ai`, only that player's device shows the number selector.

Keep the host's browser open for the whole match. This is a no-backend, free-tier architecture; it does not include server-side anti-cheat or host migration.

---

## Part A — Firebase Console setup (Spark / free plan)

### 1. Create a Firebase project

1. Open [Firebase Console](https://console.firebase.google.com/).
2. Choose **Create a project** and give it a name, such as `ludo-royale-online`.
3. Google Analytics is not required for this game; you can leave it off.

### 2. Register a Web App

1. In the Firebase project overview, click the Web icon (`</>`).
2. Give the app a nickname such as `Ludo Royale Web`.
3. Register the app.
4. Firebase shows a configuration object. Copy its values; you will paste them into `index.html` in step 5.

### 3. Enable Anonymous Authentication

1. Open **Build → Authentication**.
2. Select **Get started** if prompted.
3. Open **Sign-in method**.
4. Enable **Anonymous** and save.

### 4. Create Realtime Database

1. Open **Build → Realtime Database**.
2. Click **Create Database**.
3. Choose a region close to your players (for example, an Asia region).
4. Choose **Locked mode**. Do not use open Test mode for a public game.
5. After it is created, copy the database URL shown on the Realtime Database page. It may end in `.firebasedatabase.app` or `.firebaseio.com`.

### 5. Add the Firebase config to the game

Open `index.html` in a code editor and find:

```js
const firebaseConfig = {
    apiKey: "PASTE_YOUR_FIREBASE_API_KEY",
    authDomain: "PASTE_YOUR_PROJECT_ID.firebaseapp.com",
    databaseURL: "https://PASTE_YOUR_DATABASE_NAME-default-rtdb.firebaseio.com",
    projectId: "PASTE_YOUR_PROJECT_ID",
    storageBucket: "PASTE_YOUR_PROJECT_ID.appspot.com",
    messagingSenderId: "PASTE_YOUR_MESSAGING_SENDER_ID",
    appId: "PASTE_YOUR_FIREBASE_APP_ID"
};
```

Replace each placeholder with the exact value from your Firebase Web App config. For `databaseURL`, use the exact URL shown in **Realtime Database**, not a guessed URL. Save the file.

The Firebase Web config is intended to be present in a browser app. It is not a server secret. Never put a service-account JSON file or private server key in this project.

### 6. Publish the database security rules

1. In Firebase Console, open **Realtime Database → Rules**.
2. Replace the rules with the complete contents of `firebase-database.rules.json`.
3. Click **Publish**.
4. Confirm Authentication → Anonymous is enabled.

These rules allow authenticated clients to read a room when they know its code, claim only an empty seat, and send commands as their own user. Only the room host can write the shared game state. Keep the room code private to the people you invite.

---

## Part B — Deploy the game on Vercel (free static hosting)

### Option 1: GitHub import (recommended)

1. Sign in to [GitHub](https://github.com/) and create a new repository, for example `ludo-royale-online`.
2. Upload `index.html` to the repository root. You can also upload `firebase-database.rules.json` and this README.
3. Open [Vercel](https://vercel.com/) and sign in.
4. Choose **Add New → Project**.
5. Import your `ludo-royale-online` GitHub repository.
6. In project configuration:
   - Framework Preset: **Other**
   - Build Command: leave blank
   - Output Directory: `.` (or leave the default if Vercel detects the static root)
   - Root Directory: repository root
7. Click **Deploy**.
8. When deployment finishes, open the generated `https://your-project.vercel.app` URL.

Every later push to the connected GitHub production branch will trigger a new deployment.

### 7. Add the Vercel domain to Firebase Authentication

1. In Firebase Console, open **Authentication → Settings → Authorized domains**.
2. Add your Vercel hostname, for example `your-project.vercel.app` (hostname only; no `https://` and no path).
3. Save.

### 8. Test with phones

1. Open the Vercel URL on the host phone.
2. Enter a player name and select the maximum room size (2, 3, or 4).
3. Tap **CREATE ROOM**.
4. Share the six-character room code with the other players.
5. Each guest opens the same Vercel URL, enters a name and the room code, then taps **JOIN ROOM**.
6. Once at least two humans have joined, the host taps **START ONLINE MATCH**.
7. Test that only the active player can roll and move, and that all phones see the same dice and token positions.

For the special dice option, name one seat exactly `Ai` before creating/joining that seat. The selector appears only on the device assigned to that seat; other players see the resulting roll, not the selector.

---

## Free-tier notes and limitations

- Firebase Realtime Database's Spark plan currently includes **1 GB stored data**, **10 GB/month downloads**, and a **100 simultaneous connection** limit. The game is intended for small private rooms, not large public traffic.
- Anonymous Authentication is available without phone/SMS verification.
- Vercel's free Hobby plan is intended for personal, non-commercial use and is subject to Vercel's current plan terms and usage limits.
- Monitor Firebase Console → Realtime Database → Usage. If the free quota is exceeded, Firebase may reject further database activity until the quota window resets or the plan changes.
- No Cloud Functions are used, so this project does not require Firebase's paid server-function tier.
- The room host must remain online. If the host closes the browser or loses connection for a long time, the match may stop syncing. There is no automatic host migration or server-side anti-cheat in this free client-authoritative version.

## Troubleshooting

**“Firebase setup needed”**
- The placeholders in `firebaseConfig` have not been replaced, or the page was deployed before saving the config.

**“auth/operation-not-allowed”**
- Enable Anonymous sign-in in Firebase Authentication → Sign-in method.

**“Permission denied”**
- Publish the supplied Realtime Database rules.
- Check that the Vercel hostname is in Authentication → Settings → Authorized domains.
- Check that the database URL is copied exactly from the Realtime Database page.

**Room code says not found**
- Make sure the host created the room and has not left it. Room codes are case-insensitive in the UI.

**Players see different boards or moves**
- Refresh all phones, create a new room, and test again. Keep the host tab open. Avoid opening the same room twice on one device.

**The local game**
- If you do not create or join an online room, the original local game remains available with the original player/CPU setup and **START 3D MATCH** button.
