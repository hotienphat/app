# HackinOS Builder: Kế Hoạch Triển Khai & Code Snippets

## Yêu cầu cần có

Trước khi bắt đầu triển khai dự án, hãy đảm bảo máy tính phát triển đáp ứng các điều kiện sau:

- **Hệ điều hành**:
  - Windows 10 (64‑bit) trở lên
  - macOS Mojave (10.14) trở lên
  - Linux (Ubuntu 18.04 LTS hoặc tương đương)
- **Node.js & Trình quản lý gói**:
  - Node.js phiên bản 16.x LTS hoặc mới hơn
  - npm (đi kèm Node.js) hoặc Yarn
- **Quản lý mã nguồn**:
  - Cài đặt Git CLI và cấu hình sẵn sàng
- **Ngữ cảnh chạy Tuỳ chọn**:
  - Python 3.8+ (cho các script phụ trợ)
  - Java JDK 11+ (chỉ khi cần tích hợp công cụ Java)
- **Công cụ toàn cục**:
  - Electron Forge CLI (cài qua `npm install -g @electron-forge/cli`)
  - electron-builder (cài qua `npm install -g electron-builder`, nếu cần)
- **Công cụ phát triển**:
  - VS Code hoặc trình soạn thảo bạn ưa thích
  - Các extension gợi ý: ESLint, Prettier, TypeScript
- **Khác**:
  - Kết nối Internet ổn định
  - Tối thiểu 5 GB dung lượng trống trên ổ đĩa

## Giai đoạn 1: Khởi tạo & Thiết lập (Tuần 0) Khởi tạo & Thiết lập (Tuần 0)

### 1.1 Khởi tạo Repo & Cấu trúc thư mục

```bash
mkdir hackinos-builder && cd hackinos-builder
git init
git checkout -b dev
mkdir public src/main src/renderer src/engine scripts tests
```

### 1.2 Cài đặt & Cấu hình dự án

```bash
npm init -y
npm install -D typescript @types/node electron electron-forge eslint prettier eslint-config-prettier eslint-plugin-react @typescript-eslint/parser @typescript-eslint/eslint-plugin husky lint-staged jest @types/jest ts-jest
npx tsc --init
npx jest --init
```

**package.json** (trích):

```json
{
  "scripts": {
    "start": "electron-forge start",
    "package": "electron-forge package",
    "lint": "eslint './src/**/*.{ts,tsx,js}'",
    "format": "prettier --write './**/*.{ts,tsx,js,json,md}'",
    "test": "jest",
    "prepare": "husky install"
  },
  "lint-staged": {
    "src/**/*.{ts,tsx,js}": ["eslint --fix", "prettier --write"]
  }
}
```

**tsconfig.json** (trích):

```json
{
  "compilerOptions": {"target":"ES2020","module":"CommonJS","outDir":"lib","strict":true,"esModuleInterop":true,"jsx":"react","skipLibCheck":true},
  "include":["src"]
}
```

---

## Giai đoạn 2: UI Wizard & Theming (Tuần 1–2)

### 2.1 Cài đặt & Cấu hình Tailwind

```bash
npm install react react-dom zustand
npm install -D tailwindcss postcss autoprefixer\ npx tailwindcss init -p
```

**tailwind.config.js**:

```js
module.exports = {
  content: ['./public/index.html','./src/**/*.{js,jsx,ts,tsx}'],
  darkMode: 'class',
  theme: { extend: {} },
  plugins: [],
};
```

### 2.2 Wizard Store & Component

```ts
// src/renderer/stores/wizardStore.ts
import create from 'zustand';
export const useWizardStore = create(set => ({ current: 0, data: {}, next: () => set(s => ({ current: Math.min(s.current+1,3) })), back: () => set(s => ({ current: Math.max(s.current-1,0) })) }));
```

```tsx
// src/renderer/components/Wizard.tsx
import React from 'react'; import { useWizardStore } from '../stores/wizardStore';
const steps=['Welcome','Pre-check','Path','Summary'];
export default function Wizard(){ const {current,next,back}=useWizardStore();
  return (<div className="p-4">...</div>);
}
```

### 2.3 App Entry & Theme Toggle

```tsx
// src/renderer/App.tsx
import React,{useState,useEffect}from'react';import Wizard from'./components/Wizard';
export default function App(){ const[dark,setDark]=useState(false);
  useEffect(()=>{document.documentElement.classList.toggle('dark',dark)},[dark]);
  return(<div className="min-h-screen bg-white dark:bg-gray-900 text-gray-900 dark:text-gray-100"><header>...<button onClick={()=>setDark(!dark)}>{dark?'Light':'Dark'} Mode</button></header><main><Wizard/></main></div>);
}
```

---

## Giai đoạn 3: Engine Core (Tuần 3–4)

### 3.1 manifest.json

```json
{
  "opencore": {
    "url": "https://github.com/acidanthera/OpenCorePkg/releases/download/1.0.5/OpenCore-1.0.5-RELEASE.zip",
    "checksum": "sha256:2608743afe81001114a85b6fba6555edbc43813b82538e1f70220aeb0662d8be"
  },
  "kexts": [
    {
      "name": "Lilu.kext",
      "url": "https://github.com/acidanthera/Lilu/releases/download/1.7.1/Lilu-1.7.1-RELEASE.zip",
      "checksum": "sha256:a2c8cf8db59bcbb725b24782b7b75a5f3779fc4eb514d5cec21803b7e14902c3"
    },
    {
      "name": "WhateverGreen.kext",
      "url": "https://github.com/acidanthera/WhateverGreen/releases/download/1.7.0/WhateverGreen-1.7.0-RELEASE.zip",
      "checksum": "sha256:6d6ffe8334ad60f784a662794e67b2560b79d757d506841dc8ca9994ab39979b"
    }
  ]
}
```

### 3.2 Scan Hardware

```ts
// src/engine/detect/hardware.ts
import si from'systeminformation';import fs from'fs-extra';
export async function scanHardware(){ const[ cpu,gpu,mem,storage,net ]=await Promise.all([si.cpu(),si.graphics(),si.mem(),si.blockDevices(),si.networkInterfaces()]);
  const info={cpu,gpu,memory:mem,storage,network:net};
  await fs.outputJson('cache/hardware.json',info,{spaces:2});
  return info;
}
```

### 3.3 Download & Checksum

```ts
// src/engine/utils/downloader.ts
import https from'https';import fs from'fs-extra';import crypto from'crypto';import path from'path';
export async function downloadFile(u,d){ await fs.ensureDir(path.dirname(d)); return new Promise((res,rej)=>{https.get(u,r=>{const f=fs.createWriteStream(d);r.pipe(f);f.on('finish',()=>f.close(res));}).on('error',rej);});}
export function verifyChecksum(f,e){const sum=crypto.createHash('sha256').update(fs.readFileSync(f)).digest('hex');if(sum!==e)throw new Error('Checksum mismatch');}
```

### 3.4 Generate Config

```ts
// src/engine/config/generator.ts
import fs from'fs-extra';import path from'path';import{HardwareInfo}from'../detect/hardware';
export function generateConfig(hw:HardwareInfo){ const tpl=fs.readFileSync(path.resolve(__dirname,'template.plist'),'utf-8'); const model=hw.cpu.brand.includes('Intel')?'MacBookPro15,1':'iMac19,1'; const out=tpl.replace(/{{systemProductName}}/,model); fs.ensureDirSync('cache/components/EFI/OC'); fs.writeFileSync('cache/components/EFI/OC/config.plist',out); }
```

---

## Giai đoạn 4: Bundle EFI & Packaging (Tuần 5–6)

### 4.1 bundleEFI & Snapshot

```ts
// src/engine/build/bundle.ts
import fs from'fs-extra'; export async function bundleEFI(){ await fs.remove('dist/EFI'); await fs.copy('cache/components/EFI/OC','dist/EFI'); return 'dist/EFI'; }
```

```ts
// src/engine/utils/snapshot.ts
import fs from'fs-extra'; export function snapshotEFI(){ const dest=`backups/EFI-${Date.now()}`; fs.copySync('dist/EFI',dest); return dest; } export function restoreEFI(s:string){ fs.removeSync('dist/EFI'); fs.copySync(s,'dist/EFI'); }
```

### 4.2 IPC & BuildStep UI

```ts
// src/main/index.ts
ipcMain.handle('bundle-efi',()=>bundleEFI());ipcMain.handle('snapshot-efi',()=>snapshotEFI());ipcMain.handle('restore-efi',(_,dir)=>restoreEFI(dir));
```

```tsx
// src/renderer/components/BuildStep.tsx
import React,{useEffect}from'react';const{ipcRenderer}=window.require('electron');export default()=>{useEffect(()=>ipcRenderer.invoke('bundle-efi').then(p=>console.log(p)),[]);return<div>Buttons+ListSnapshots</div>;};
```

### 4.3 Installer Config & Scripts

**electron-builder.config.js**:

```js
module.exports={appId:'com.hackinOS.builder',directories:{output:'release'},win:{target:'nsis'},mac:{target:'pkg'},linux:{target:'AppImage'}};
```

**package.json scripts**:

```json
{"build:win":"electron-builder --win","build:mac":"electron-builder --mac","build:linux":"electron-builder --linux","build:all":"electron-builder --publish=never"}
```

---

## Giai đoạn 5: Test & Auto-update (Tuần 7–8)

### 5.1 Auto-update

```ts
// src/main/updater.ts
import{autoUpdater}from'electron-updater';export function initAutoUpdate(win){autoUpdater.setFeedURL({provider:'github',owner:'you',repo:'hackinos-builder'});autoUpdater.checkForUpdates();autoUpdater.on('update-available',i=>win.webContents.send('update-available',i));autoUpdater.on('update-downloaded',()=>autoUpdater.quitAndInstall());}
```

```tsx
// src/renderer/components/Updater.tsx
import React,useEffect from'react';const{ipcRenderer}=window.require('electron');export default()=>{useEffect(()=>ipcRenderer.on('update-available',(_,i)=>alert(`Update ${i.version}`)),[]);return null;};
```

---

## Giai đoạn 6: Public Beta & Feedback (Tuần 9)

### 6.1 GitHub Issues Script

```ts
// scripts/createIssue.ts
import{Octokit}from'@octokit/rest';const o=new Octokit({auth:process.env.GH_TOKEN});async function reportBug(t,b){await o.issues.create({owner:'you',repo:'hackinos-builder',title:t,body:b});}
```

### 6.2 Code Signing

```js
// electron-builder.config.js additions
module.exports={mac:{identity:'Developer ID Application:Name (ID)',entitlements:'build/entitlements.mac.plist',hardenedRuntime:true},win:{certificateFile:'cert.pfx',certificatePassword:process.env.CERT_PASS}};
```

---

> **Notes:** Daily standup, review each sprint. Slack: #hackinos-dev. 阅读执行！

