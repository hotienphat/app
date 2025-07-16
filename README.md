hackinos-builder/
├── public/           # Static assets (logo, icons, themes)
├── src/
│   ├── main/         # Process chính của Electron
│   │   ├── index.ts  # Entrypoint
│   │   └── updater/   # Module auto-update
│   ├── renderer/     # React UI
│   │   ├── App.tsx    # Component gốc
│   │   ├── components/
│   │   └── stores/
│   └── engine/       # Node.js scripts & Python bindings
│       ├── detect/   # Quét phần cứng
│       ├── build/    # Tạo EFI, config.plist
│       ├── kexts/    # Quản lý kexts & drivers
│       └── utils/    # Logging, snapshot, rollback
├── scripts/          # Các script hỗ trợ (build, lint, pkg)
├── tests/            # Unit & integration tests
├── package.json
├── tsconfig.json
└── electron-builder.yml
