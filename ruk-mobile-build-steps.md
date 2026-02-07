# Ruk Mobile: Build Steps

## Part 1: Austin's Steps (One-Time)

### Step 1: Create the Repo

```bash
cd ~/Code
mkdir ruk-mobile && cd ruk-mobile
git init
gh repo create FractalLabsDev/ruk-mobile --private --source=. --remote=origin
```

### Step 2: Initialize React Native

```bash
npx @react-native-community/cli init RukMobile --package-name=dev.fractallabs.rukmobile --pm npm
```

This creates the default RN structure. We'll reorganize.

### Step 3: Move Generated Files

```bash
# Move generated content up to repo root
mv RukMobile/* .
mv RukMobile/.* . 2>/dev/null
rmdir RukMobile
```

### Step 4: Install Core Dependencies

```bash
npm install zustand @tanstack/react-query axios
npm install @react-navigation/native @react-navigation/native-stack
npm install react-native-screens react-native-safe-area-context
npm install @react-native-async-storage/async-storage
npm install react-native-audio-recorder-player
npm install react-native-fs
npm install @rneui/themed @rneui/base
npm install react-native-vector-icons
```

### Step 5: Install Dev Dependencies

```bash
npm install -D @types/react @types/react-native
```

### Step 6: iOS Setup

```bash
cd ios
pod install
cd ..
```

### Step 7: Initial Commit

```bash
git add .
git commit -m "Initial RN 0.77 setup with core dependencies"
git push -u origin main
```

### Step 8: Add to RUK additionalDirectories

Edit `~/Code/RUK/.claude/settings.json` and add to `additionalDirectories`:
```json
"../ruk-mobile"
```

### Step 9: Verify Build

```bash
npm run ios
# Should open simulator with default RN app
```

---

## Part 2: Mac Mini Setup (For Ruk Autonomy)

Run these on ruk-macmini after Austin completes Part 1:

### 1. Clone the Repo

```bash
cd ~/Code
git clone git@github.com:FractalLabsDev/ruk-mobile.git
```

### 2. Install Dependencies

```bash
cd ~/Code/ruk-mobile
npm install
```

### 3. Add to Claude Settings

Edit `~/Code/RUK/.claude/settings.json` and add to `additionalDirectories`:
```json
"../ruk-mobile"
```

### 4. iOS Build Dependencies (if not already installed)

```bash
# Install Xcode Command Line Tools (if needed)
xcode-select --install

# Install CocoaPods (if needed)
sudo gem install cocoapods

# Install iOS pods
cd ~/Code/ruk-mobile/ios
pod install
```

**Note:** Full iOS builds require Xcode. The Mac Mini may need Xcode installed for simulator testing. For dev work without simulator:
- I can write/edit code
- I can run tests
- I can run linting
- I cannot build to simulator (requires Xcode GUI or xcodebuild)

---

## What Ruk Can Do Autonomously

Once the repo is cloned to Mac Mini with `additionalDirectories` configured:

| Task | Capability |
|------|------------|
| Write/edit TypeScript | ✅ Full |
| Create components | ✅ Full |
| Run `npm test` | ✅ Full |
| Run `npm run lint` | ✅ Full |
| Install npm packages | ✅ Full |
| Commit and push | ✅ Full |
| Run iOS build | ⚠️ Requires Xcode |
| Run Android build | ⚠️ Requires Android Studio |

**Recommended workflow:** I write code, push to GitHub, Austin pulls and tests on device.

---

## Quick Reference: Start Developing

After setup, from either machine:

```bash
cd ~/Code/ruk-mobile
npm start          # Metro bundler
npm run ios        # iOS simulator (requires Xcode)
npm run android    # Android emulator (requires Android Studio)
npm test           # Jest tests
npm run lint       # ESLint
```

---

## Project Structure After Init

```
ruk-mobile/
├── App.tsx
├── package.json
├── tsconfig.json
├── babel.config.js
├── metro.config.js
├── index.js
├── ios/
│   └── Podfile
├── android/
└── __tests__/
```

We'll add:
```
src/
├── components/
├── hooks/
├── services/
├── store/
├── screens/
├── navigation/
└── types/
```

---

*Ready to build. Step 1 is yours.*
