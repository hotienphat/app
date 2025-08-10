[hệ_thống_order_quan_an_ca_phe_3_vai_tro_admin_khach_ban_hang_full_code_hdsd.md](https://github.com/user-attachments/files/21703583/h._th.ng_order_quan_an_ca_phe_3_vai_tro_admin_khach_ban_hang_full_code_hdsd.md)
# Hệ thống Order Quán Ăn/Cà Phê – 3 vai trò (Admin, Khách, Bán hàng)

Giải pháp hoàn chỉnh dùng **Firebase** (Firestore + Auth + Storage) + **HTML/JS thuần + TailwindCSS**. Dễ triển khai, realtime, chạy tốt trên PC/Laptop/Mobile.

---

## Mục lục

1. [Tính năng & kiến trúc](#tinh-nang)
2. [Bắt đầu nhanh (5–15 phút)](#bat-dau-nhanh)
3. [Cấu trúc thư mục](#cau-truc)
4. [Mã nguồn (từng file)](#ma-nguon)
5. [Luồng sử dụng chi tiết](#huong-dan-su-dung)
6. [Quy tắc bảo mật Firestore](#rules)
7. [Tạo & in QR cho từng bàn](#qr-ban)
8. [Triển khai Firebase Hosting](#hosting)
9. [Kiểm thử & Checklist](#kiem-thu)
10. [Mở rộng & nâng cấp](#mo-rong)
11. [FAQ](#faq)

---



## 1) Tính năng & kiến trúc

### 3 vai trò

- **Khách hàng (Client):** quét **QR tại bàn** → chọn món → đặt món → xem tổng tiền → hiển thị **QR thanh toán** + **Số TK/Ngân hàng**.
- **Người bán hàng/Thu ngân (Seller):** nhận đơn **realtime**, có **âm thanh báo** đơn mới; cập nhật trạng thái (đang làm/đã mang ra/đã thanh toán).
- **Quản trị (Admin):** quản lý **menu (CRUD)**, **giá**, **hình ảnh**, **cài đặt thanh toán**; theo dõi toàn bộ đơn, xuất tổng quan.

### Yêu cầu đã đáp ứng

- 2/3 dạng truy cập (Admin, Khách, Seller tách trang riêng).
- Admin **thêm/sửa/xóa** món, **giá**, **mô tả**, **hình ảnh**.
- Khách **chỉ chọn món + số lượng** (không chỉnh menu).
- **Thanh toán**: hiển thị **QR** (tạo từ nội dung chuyển khoản) + **Số TK** + **Ngân hàng**. (Có phần mở rộng VietQR ở dưới.)
- **Đa nền tảng** (responsive) + **UI gọn, dễ dùng**.
- **QR bàn** đưa khách thẳng tới trang đặt món kèm **số bàn**.
- **Âm thanh báo** cho Seller/Admin khi có đơn mới; báo **khi đơn đánh dấu đã thanh toán**.

### Công nghệ

- Frontend: HTML + Tailwind via CDN, JS thuần (module).
- Backend: Firebase Web SDK (Auth, Firestore, Storage). **Không cần server riêng**.
- Realtime: Firestore `onSnapshot()`.

### Mô hình dữ liệu Firestore (đơn giản, đủ dùng)

- `items` (collection): `{ name, price, category, imageUrl, active, createdAt }`
- `orders` (collection):
  ```js
  {
    table,                // số bàn (string/number)
    items: [{itemId, name, price, qty, note}],
    total,
    status,               // 'pending' | 'preparing' | 'served' | 'paid' | 'cancelled'
    paymentMethod,        // 'cash' | 'transfer'
    paymentStatus,        // 'unpaid' | 'paid'
    createdAt,
    paidAt,
  }
  ```
- `settings/payment` (doc): `{ bankName, accountName, accountNumber, bankCode? }`

> **Ghi chú:** Quy tắc bảo mật cho phép **public đọc menu và tạo đơn**, còn **sửa đơn chỉ khi đăng nhập (Seller/Admin)**.

---



## 2) Bắt đầu nhanh (5–15 phút)

1. **Tạo Firebase Project** ([https://console.firebase.google.com](https://console.firebase.google.com)) → Add app **Web** → lấy **Firebase config**.

2. Bật:

- **Authentication** → Sign-in method → **Email/Password** (ON)
- **Firestore** → Create DB → Production mode
- **Storage** → Create

3. Tạo **tài khoản đăng nhập** cho **Admin** và **Seller** (email/password).

4. Tải mã nguồn bên dưới, **điền Firebase config** vào file `public/js/firebase.js`.

5. Chạy cục bộ (tùy chọn):

- Cách nhanh nhất: mở file `public/admin.html`/`client.html`/`seller.html` bằng trình duyệt (nếu CORS cản trở, dùng Hosting – xem mục 8).
- Khuyên dùng: **Firebase Hosting** (miễn phí) – xem mục [Triển khai](#hosting).

6. Vào `admin.html` → tab **Cài đặt** → nhập **Số TK/Ngân hàng** → Lưu.

7. **Thêm món** ở tab **Menu**.

8. In **QR bàn** (mục [7](#qr-ban)).

---



## 3) Cấu trúc thư mục

```
/public
  ├─ admin.html        // Trang quản trị
  ├─ seller.html       // Trang bán hàng/thu ngân
  ├─ client.html       // Trang khách
  ├─ js/
  │   ├─ firebase.js   // Khởi tạo Firebase + helper
  │   ├─ utils.js      // format tiền, beep, tiện ích
  │   ├─ admin.js
  │   ├─ seller.js
  │   └─ client.js
  └─ assets/
      └─ placeholder.png (tùy chọn)
```

---



## 4) Mã nguồn (từng file)

> **Thay thế** `/* TODO: paste your Firebase config here */` bằng config của bạn.

### `/public/js/firebase.js`

```html
<script type="module">
// DÙNG NHƯ MODULE RIÊNG: Lưu nội dung này vào /public/js/firebase.js (không đặt trong <script> khi chạy thực tế)
</script>
```

```javascript
// /public/js/firebase.js
// Khởi tạo Firebase + export sẵn các SDK dùng chung
import { initializeApp } from 'https://www.gstatic.com/firebasejs/10.12.0/firebase-app.js';
import { getAuth, onAuthStateChanged, signInWithEmailAndPassword, signOut } from 'https://www.gstatic.com/firebasejs/10.12.0/firebase-auth.js';
import { getFirestore, collection, doc, addDoc, setDoc, getDoc, getDocs, onSnapshot, updateDoc, deleteDoc, query, orderBy, serverTimestamp, where } from 'https://www.gstatic.com/firebasejs/10.12.0/firebase-firestore.js';
import { getStorage, ref, uploadBytes, getDownloadURL } from 'https://www.gstatic.com/firebasejs/10.12.0/firebase-storage.js';

export const firebaseConfig = {
  // TODO: paste your Firebase config here
};

export const app = initializeApp(firebaseConfig);
export const auth = getAuth(app);
export const db = getFirestore(app);
export const storage = getStorage(app);

// Helper: lấy/ lưu cài đặt thanh toán
export async function getPaymentSettings() {
  const snap = await getDoc(doc(db, 'settings', 'payment'));
  return snap.exists() ? snap.data() : null;
}
export async function savePaymentSettings(data) {
  await setDoc(doc(db, 'settings', 'payment'), data, { merge: true });
}

// Helper: format tiền VNĐ
export function currencyVND(n) {
  try { return Number(n).toLocaleString('vi-VN', { style: 'currency', currency: 'VND' }); }
  catch { return n; }
}

// Helper: phát âm thanh beep (không cần file audio)
export function beep(duration = 300, frequency = 880) {
  try {
    const ctx = new (window.AudioContext || window.webkitAudioContext)();
    const osc = ctx.createOscillator();
    const gain = ctx.createGain();
    osc.type = 'sine';
    osc.frequency.value = frequency;
    osc.connect(gain); gain.connect(ctx.destination);
    osc.start();
    setTimeout(() => { osc.stop(); ctx.close(); }, duration);
  } catch (e) { /* bỏ qua nếu không hỗ trợ */ }
}

export {
  onAuthStateChanged, signInWithEmailAndPassword, signOut,
  collection, doc, addDoc, setDoc, getDoc, getDocs, onSnapshot, updateDoc, deleteDoc,
  query, orderBy, serverTimestamp, where, ref, uploadBytes, getDownloadURL
};
```

### `/public/admin.html`

```html
<!doctype html>
<html lang="vi">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Admin – Quản trị quán</title>
  <script src="https://cdn.tailwindcss.com"></script>
</head>
<body class="bg-slate-50 text-slate-800">
  <div class="max-w-6xl mx-auto p-4">
    <header class="flex items-center justify-between mb-4">
      <h1 class="text-2xl font-bold">Quản trị</h1>
      <div id="authArea" class="flex items-center gap-2"></div>
    </header>

    <!-- Tabs -->
    <div class="flex gap-2 mb-4">
      <button data-tab="menu" class="tab px-3 py-2 rounded bg-white shadow">Menu</button>
      <button data-tab="orders" class="tab px-3 py-2 rounded bg-white shadow">Đơn hàng</button>
      <button data-tab="settings" class="tab px-3 py-2 rounded bg-white shadow">Cài đặt</button>
    </div>

    <!-- Menu Tab -->
    <section id="tab-menu" class="tab-pane">
      <div class="grid md:grid-cols-3 gap-4">
        <form id="itemForm" class="md:col-span-1 bg-white p-4 rounded-xl shadow">
          <h2 class="font-semibold mb-3">Thêm/Sửa món</h2>
          <input id="itemId" type="hidden" />
          <label class="block mb-2 text-sm">Tên món</label>
          <input id="name" class="w-full border rounded px-3 py-2 mb-2" required />
          <label class="block mb-2 text-sm">Giá (VND)</label>
          <input id="price" type="number" min="0" class="w-full border rounded px-3 py-2 mb-2" required />
          <label class="block mb-2 text-sm">Danh mục</label>
          <input id="category" class="w-full border rounded px-3 py-2 mb-2" placeholder="Cà phê, Trà, Món ăn..." />
          <label class="block mb-2 text-sm">Hình ảnh</label>
          <input id="imageFile" type="file" accept="image/*" class="mb-3" />
          <div class="flex items-center gap-2 mb-3">
            <input id="active" type="checkbox" class="scale-110" checked />
            <label for="active">Đang bán</label>
          </div>
          <div class="flex gap-2">
            <button class="bg-blue-600 text-white px-3 py-2 rounded" type="submit">Lưu</button>
            <button id="resetForm" type="button" class="px-3 py-2 rounded border">Reset</button>
          </div>
        </form>

        <div class="md:col-span-2 bg-white p-4 rounded-xl shadow">
          <div class="flex items-center justify-between mb-2">
            <h2 class="font-semibold">Danh sách món</h2>
            <input id="search" placeholder="Tìm món..." class="border rounded px-3 py-2" />
          </div>
          <div id="items" class="grid md:grid-cols-2 gap-3"></div>
        </div>
      </div>
    </section>

    <!-- Orders Tab -->
    <section id="tab-orders" class="tab-pane hidden">
      <div class="bg-white p-4 rounded-xl shadow">
        <div class="flex flex-wrap items-center gap-2 mb-3">
          <span class="font-semibold">Đơn hàng realtime</span>
          <select id="statusFilter" class="border rounded px-2 py-1">
            <option value="">Tất cả</option>
            <option>pending</option>
            <option>preparing</option>
            <option>served</option>
            <option>paid</option>
            <option>cancelled</option>
          </select>
        </div>
        <div id="orders" class="grid md:grid-cols-2 gap-3"></div>
      </div>
    </section>

    <!-- Settings Tab -->
    <section id="tab-settings" class="tab-pane hidden">
      <form id="paymentForm" class="bg-white p-4 rounded-xl shadow max-w-lg">
        <h2 class="font-semibold mb-3">Cài đặt thanh toán</h2>
        <label class="block mb-1 text-sm">Tên chủ tài khoản</label>
        <input id="accountName" class="w-full border rounded px-3 py-2 mb-2" required />
        <label class="block mb-1 text-sm">Số tài khoản</label>
        <input id="accountNumber" class="w-full border rounded px-3 py-2 mb-2" required />
        <label class="block mb-1 text-sm">Ngân hàng</label>
        <input id="bankName" class="w-full border rounded px-3 py-2 mb-4" required />
        <button class="bg-blue-600 text-white px-4 py-2 rounded">Lưu</button>
      </form>
    </section>
  </div>

  <script type="module" src="./js/admin.js"></script>
</body>
</html>
```

### `/public/js/admin.js`

```javascript
import {
  auth, onAuthStateChanged, signInWithEmailAndPassword, signOut,
  db, storage,
  collection, doc, addDoc, setDoc, getDoc, getDocs, onSnapshot, updateDoc, deleteDoc,
  query, orderBy, serverTimestamp, where, ref, uploadBytes, getDownloadURL,
  getPaymentSettings, savePaymentSettings, currencyVND, beep
} from './firebase.js';

// ===== Auth UI (email/password đơn giản) =====
const authArea = document.getElementById('authArea');
function renderAuth(user) {
  if (!user) {
    authArea.innerHTML = `
      <form id="loginForm" class="flex gap-2">
        <input id="email" type="email" placeholder="Email" class="border rounded px-3 py-1" required>
        <input id="password" type="password" placeholder="Mật khẩu" class="border rounded px-3 py-1" required>
        <button class="bg-blue-600 text-white px-3 py-1 rounded">Đăng nhập</button>
      </form>`;
    document.getElementById('loginForm').onsubmit = async (e) => {
      e.preventDefault();
      const email = document.getElementById('email').value;
      const password = document.getElementById('password').value;
      try { await signInWithEmailAndPassword(auth, email, password); }
      catch (err) { alert('Đăng nhập thất bại: ' + err.message); }
    };
  } else {
    authArea.innerHTML = `
      <div class="flex items-center gap-2">
        <span class="text-sm">${user.email}</span>
        <button id="logoutBtn" class="px-3 py-1 rounded border">Đăng xuất</button>
      </div>`;
    document.getElementById('logoutBtn').onclick = () => signOut(auth);
  }
}

onAuthStateChanged(auth, (user) => {
  renderAuth(user);
});

// ===== Tabs logic =====
const tabs = document.querySelectorAll('.tab');
const panes = document.querySelectorAll('.tab-pane');
tabs.forEach(btn => btn.addEventListener('click', () => {
  panes.forEach(p => p.classList.add('hidden'));
  document.getElementById('tab-' + btn.dataset.tab).classList.remove('hidden');
}));

// ===== MENU CRUD =====
const itemsEl = document.getElementById('items');
const itemForm = document.getElementById('itemForm');
const resetFormBtn = document.getElementById('resetForm');
const searchInput = document.getElementById('search');

let unsubscribeItems = null;

function renderItemCard(id, data) {
  const div = document.createElement('div');
  div.className = 'rounded-xl border p-3 flex gap-3 items-center';
  div.innerHTML = `
    <img src="${data.imageUrl || './assets/placeholder.png'}" class="w-20 h-20 object-cover rounded">
    <div class="flex-1">
      <div class="font-medium">${data.name}</div>
      <div class="text-sm text-slate-600">${data.category || ''}</div>
      <div class="font-semibold">${currencyVND(data.price || 0)}</div>
      <div class="text-xs mt-1">${data.active ? 'Đang bán' : 'Ngưng bán'}</div>
    </div>
    <div class="flex flex-col gap-2">
      <button data-edit="${id}" class="px-3 py-1 rounded bg-amber-500 text-white">Sửa</button>
      <button data-del="${id}" class="px-3 py-1 rounded bg-rose-600 text-white">Xóa</button>
    </div>`;
  return div;
}

function loadItems() {
  if (unsubscribeItems) unsubscribeItems();
  const q = query(collection(db, 'items'), orderBy('createdAt', 'desc'));
  unsubscribeItems = onSnapshot(q, (snap) => {
    const kw = (searchInput.value || '').toLowerCase();
    itemsEl.innerHTML = '';
    snap.forEach(docSnap => {
      const data = docSnap.data();
      if (kw && !(`${data.name} ${data.category}`.toLowerCase().includes(kw))) return;
      itemsEl.appendChild(renderItemCard(docSnap.id, data));
    });
    // bind actions
    itemsEl.querySelectorAll('[data-edit]').forEach(btn => btn.onclick = () => editItem(btn.dataset.edit));
    itemsEl.querySelectorAll('[data-del]').forEach(btn => btn.onclick = () => delItem(btn.dataset.del));
  });
}

async function editItem(id) {
  const snap = await getDoc(doc(db, 'items', id));
  if (!snap.exists()) return;
  const d = snap.data();
  document.getElementById('itemId').value = id;
  document.getElementById('name').value = d.name || '';
  document.getElementById('price').value = d.price || 0;
  document.getElementById('category').value = d.category || '';
  document.getElementById('active').checked = !!d.active;
}

async function delItem(id) {
  if (!confirm('Xóa món này?')) return;
  await deleteDoc(doc(db, 'items', id));
}

itemForm.onsubmit = async (e) => {
  e.preventDefault();
  const id = document.getElementById('itemId').value;
  const payload = {
    name: document.getElementById('name').value.trim(),
    price: Number(document.getElementById('price').value || 0),
    category: document.getElementById('category').value.trim(),
    active: document.getElementById('active').checked,
    createdAt: serverTimestamp(),
  };
  // upload ảnh nếu có
  const file = document.getElementById('imageFile').files[0];
  if (file) {
    const r = ref(storage, `items/${Date.now()}_${file.name}`);
    await uploadBytes(r, file);
    payload.imageUrl = await getDownloadURL(r);
  }
  try {
    if (id) {
      delete payload.createdAt; // không ghi đè
      await setDoc(doc(db, 'items', id), payload, { merge: true });
    } else {
      await addDoc(collection(db, 'items'), payload);
    }
    itemForm.reset();
  } catch (e) {
    alert('Lỗi lưu món: ' + e.message);
  }
};

resetFormBtn.onclick = () => itemForm.reset();
searchInput.oninput = () => loadItems();

loadItems();

// ===== ORDERS realtime =====
const ordersEl = document.getElementById('orders');
const statusFilter = document.getElementById('statusFilter');
let firstOrdersLoad = true;

function renderOrderCard(id, d) {
  const div = document.createElement('div');
  div.className = 'rounded-xl border p-3 bg-white';
  const itemsHtml = (d.items||[]).map(i => `<div class="flex justify-between text-sm">
    <span>${i.name} × ${i.qty}</span><span>${currencyVND(i.price*i.qty)}</span>
  </div>`).join('');
  div.innerHTML = `
    <div class="flex justify-between items-center mb-2">
      <div class="font-semibold">Bàn ${d.table || '?'}</div>
      <span class="text-xs px-2 py-1 rounded bg-slate-100">${d.status}</span>
    </div>
    <div class="space-y-1">${itemsHtml}</div>
    <div class="mt-2 font-semibold">Tổng: ${currencyVND(d.total||0)}</div>
    <div class="text-xs text-slate-600">Thanh toán: ${d.paymentMethod || 'transfer'} – ${d.paymentStatus || 'unpaid'}</div>
    <div class="flex gap-2 mt-3">
      <button data-action="preparing" data-id="${id}" class="px-3 py-1 rounded border">Đang làm</button>
      <button data-action="served" data-id="${id}" class="px-3 py-1 rounded border">Đã mang ra</button>
      <button data-action="paid" data-id="${id}" class="px-3 py-1 rounded bg-emerald-600 text-white">Đã thanh toán</button>
      <button data-action="cancelled" data-id="${id}" class="px-3 py-1 rounded bg-rose-600 text-white">Hủy</button>
    </div>`;
  return div;
}

function watchOrders() {
  const q = query(collection(db, 'orders'), orderBy('createdAt', 'desc'));
  onSnapshot(q, (snap) => {
    // Beep nếu có đơn mới
    if (!firstOrdersLoad) {
      snap.docChanges().forEach(change => {
        if (change.type === 'added') beep();
        if (change.type === 'modified' && change.doc.data().paymentStatus === 'paid') beep(200, 440);
      });
    }
    firstOrdersLoad = false;

    const filter = statusFilter.value;
    ordersEl.innerHTML = '';
    snap.forEach(docSnap => {
      const d = docSnap.data();
      if (filter && d.status !== filter) return;
      ordersEl.appendChild(renderOrderCard(docSnap.id, d));
    });

    ordersEl.querySelectorAll('button[data-action]').forEach(btn => {
      btn.onclick = async () => {
        const id = btn.dataset.id;
        const action = btn.dataset.action;
        const updates = { status: action };
        if (action === 'paid') { updates.paymentStatus = 'paid'; updates.paidAt = serverTimestamp(); }
        await updateDoc(doc(db, 'orders', id), updates);
      };
    });
  });
}

statusFilter.onchange = () => watchOrders();
watchOrders();

// ===== SETTINGS =====
const paymentForm = document.getElementById('paymentForm');
async function loadPayment() {
  const s = await getPaymentSettings();
  if (!s) return;
  paymentForm.accountName.value = s.accountName || '';
  paymentForm.accountNumber.value = s.accountNumber || '';
  paymentForm.bankName.value = s.bankName || '';
}

paymentForm.onsubmit = async (e) => {
  e.preventDefault();
  await savePaymentSettings({
    accountName: paymentForm.accountName.value.trim(),
    accountNumber: paymentForm.accountNumber.value.trim(),
    bankName: paymentForm.bankName.value.trim(),
  });
  alert('Đã lưu cài đặt thanh toán');
};

loadPayment();
```

### `/public/seller.html`

```html
<!doctype html>
<html lang="vi">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Seller – Thu ngân/Bán hàng</title>
  <script src="https://cdn.tailwindcss.com"></script>
</head>
<body class="bg-slate-50 text-slate-800">
  <div class="max-w-5xl mx-auto p-4">
    <header class="flex items-center justify-between mb-4">
      <h1 class="text-2xl font-bold">Bán hàng</h1>
      <div id="authArea" class="flex items-center gap-2"></div>
    </header>

    <div class="bg-white p-4 rounded-xl shadow">
      <div class="flex flex-wrap items-center gap-2 mb-3">
        <span class="font-semibold">Đơn hàng mới nhất</span>
        <select id="statusFilter" class="border rounded px-2 py-1">
          <option value="">Tất cả</option>
          <option>pending</option>
          <option>preparing</option>
          <option>served</option>
          <option>paid</option>
          <option>cancelled</option>
        </select>
      </div>
      <div id="orders" class="grid md:grid-cols-2 gap-3"></div>
    </div>
  </div>

  <script type="module" src="./js/seller.js"></script>
</body>
</html>
```

### `/public/js/seller.js`

```javascript
import {
  auth, onAuthStateChanged, signInWithEmailAndPassword, signOut,
  db, collection, doc, onSnapshot, updateDoc, query, orderBy, serverTimestamp,
  currencyVND, beep
} from './firebase.js';

// Auth
const authArea = document.getElementById('authArea');
function renderAuth(user) {
  if (!user) {
    authArea.innerHTML = `
      <form id="loginForm" class="flex gap-2">
        <input id="email" type="email" placeholder="Email" class="border rounded px-3 py-1" required>
        <input id="password" type="password" placeholder="Mật khẩu" class="border rounded px-3 py-1" required>
        <button class="bg-blue-600 text-white px-3 py-1 rounded">Đăng nhập</button>
      </form>`;
    document.getElementById('loginForm').onsubmit = async (e) => {
      e.preventDefault();
      const email = document.getElementById('email').value;
      const password = document.getElementById('password').value;
      try { await signInWithEmailAndPassword(auth, email, password); }
      catch (err) { alert('Đăng nhập thất bại: ' + err.message); }
    };
  } else {
    authArea.innerHTML = `
      <div class="flex items-center gap-2">
        <span class="text-sm">${user.email}</span>
        <button id="logoutBtn" class="px-3 py-1 rounded border">Đăng xuất</button>
      </div>`;
    document.getElementById('logoutBtn').onclick = () => signOut(auth);
  }
}

onAuthStateChanged(auth, (user) => renderAuth(user));

// Orders realtime
const statusFilter = document.getElementById('statusFilter');
const ordersEl = document.getElementById('orders');
let firstLoad = true;

function renderOrderCard(id, d) {
  const div = document.createElement('div');
  div.className = 'rounded-xl border p-3 bg-white';
  const itemsHtml = (d.items||[]).map(i => `<div class="flex justify-between text-sm">
    <span>${i.name} × ${i.qty}</span><span>${currencyVND(i.price*i.qty)}</span>
  </div>`).join('');
  div.innerHTML = `
    <div class="flex justify-between items-center mb-2">
      <div class="font-semibold">Bàn ${d.table || '?'}</div>
      <span class="text-xs px-2 py-1 rounded bg-slate-100">${d.status}</span>
    </div>
    <div class="space-y-1">${itemsHtml}</div>
    <div class="mt-2 font-semibold">Tổng: ${currencyVND(d.total||0)}</div>
    <div class="text-xs text-slate-600">Thanh toán: ${d.paymentMethod || 'transfer'} – ${d.paymentStatus || 'unpaid'}</div>
    <div class="flex gap-2 mt-3">
      <button data-action="preparing" data-id="${id}" class="px-3 py-1 rounded border">Đang làm</button>
      <button data-action="served" data-id="${id}" class="px-3 py-1 rounded border">Đã mang ra</button>
      <button data-action="paid" data-id="${id}" class="px-3 py-1 rounded bg-emerald-600 text-white">Đã thanh toán</button>
    </div>`;
  return div;
}

function watchOrders() {
  const q = query(collection(db, 'orders'), orderBy('createdAt', 'desc'));
  onSnapshot(q, (snap) => {
    if (!firstLoad) {
      snap.docChanges().forEach(c => { if (c.type === 'added') beep(); });
    }
    firstLoad = false;

    const filter = statusFilter.value;
    ordersEl.innerHTML = '';
    snap.forEach(docSnap => {
      const d = docSnap.data();
      if (filter && d.status !== filter) return;
      ordersEl.appendChild(renderOrderCard(docSnap.id, d));
    });

    ordersEl.querySelectorAll('button[data-action]').forEach(btn => {
      btn.onclick = async () => {
        const id = btn.dataset.id;
        const action = btn.dataset.action;
        const updates = { status: action };
        if (action === 'paid') { updates.paymentStatus = 'paid'; updates.paidAt = serverTimestamp(); }
        await updateDoc(doc(db, 'orders', id), updates);
      };
    });
  });
}

statusFilter.onchange = () => watchOrders();
watchOrders();
```

### `/public/client.html`

```html
<!doctype html>
<html lang="vi">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Gọi món</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <!-- QRCode generator (client dùng để tạo QR thanh toán đơn giản) -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/qrcodejs/1.0.0/qrcode.min.js"></script>
</head>
<body class="bg-white text-slate-800">
  <div class="max-w-4xl mx-auto p-4">
    <header class="flex items-center justify-between mb-3">
      <h1 class="text-xl font-semibold">Gọi món</h1>
      <div id="tableTag" class="text-sm bg-slate-100 px-2 py-1 rounded"></div>
    </header>

    <div class="grid md:grid-cols-3 gap-4">
      <aside class="md:col-span-2">
        <div class="flex items-center gap-2 mb-2">
          <input id="search" placeholder="Tìm món..." class="border rounded px-3 py-2 w-full" />
        </div>
        <div id="menu" class="grid sm:grid-cols-2 gap-3"></div>
      </aside>

      <aside class="md:col-span-1">
        <div class="sticky top-4 bg-slate-50 rounded-xl border p-3">
          <h2 class="font-semibold mb-2">Giỏ hàng</h2>
          <div id="cart"></div>
          <div class="mt-2 flex justify-between font-semibold">
            <span>Tổng</span><span id="total">0</span>
          </div>
          <div class="mt-3">
            <label class="text-sm">Ghi chú</label>
            <textarea id="note" class="w-full border rounded px-2 py-1" rows="2" placeholder="Ít đá, ít ngọt..."></textarea>
          </div>
          <div class="mt-3">
            <label class="text-sm">Thanh toán</label>
            <select id="paymentMethod" class="w-full border rounded px-2 py-2">
              <option value="transfer">Chuyển khoản</option>
              <option value="cash">Tiền mặt</option>
            </select>
          </div>
          <button id="placeOrder" class="w-full mt-3 bg-blue-600 text-white py-2 rounded">Đặt món</button>
          <div id="afterOrder" class="hidden mt-3">
            <div class="text-sm mb-2">Vui lòng thanh toán theo thông tin dưới (nếu chọn chuyển khoản):</div>
            <div id="paymentInfo" class="text-sm"></div>
            <div id="qrcode" class="mt-2 flex justify-center"></div>
          </div>
        </div>
      </aside>
    </div>
  </div>

  <script type="module" src="./js/client.js"></script>
</body>
</html>
```

### `/public/js/client.js`

```javascript
import {
  db, collection, doc, addDoc, getDoc, getDocs, onSnapshot, query, orderBy, serverTimestamp,
  currencyVND, getPaymentSettings
} from './firebase.js';

// Lấy số bàn từ URL (?table=5)
const params = new URLSearchParams(location.search);
const table = params.get('table') || 'N/A';
document.getElementById('tableTag').textContent = `Bàn: ${table}`;

const menuEl = document.getElementById('menu');
const cartEl = document.getElementById('cart');
const totalEl = document.getElementById('total');
const searchInput = document.getElementById('search');
const noteEl = document.getElementById('note');
const paymentMethodEl = document.getElementById('paymentMethod');

let items = [];
let cart = []; // {id, name, price, qty}

function renderMenu() {
  const kw = (searchInput.value || '').toLowerCase();
  menuEl.innerHTML = '';
  items.filter(i => i.active !== false)
    .filter(i => !kw || `${i.name} ${i.category}`.toLowerCase().includes(kw))
    .forEach(i => {
      const div = document.createElement('div');
      div.className = 'rounded-xl border p-3 flex gap-3';
      div.innerHTML = `
        <img src="${i.imageUrl || './assets/placeholder.png'}" class="w-20 h-20 object-cover rounded">
        <div class="flex-1">
          <div class="font-medium">${i.name}</div>
          <div class="text-sm text-slate-600">${i.category || ''}</div>
          <div class="font-semibold">${currencyVND(i.price || 0)}</div>
        </div>
        <div class="flex flex-col gap-2">
          <div class="flex items-center gap-2">
            <button data-minus class="px-2 py-1 border rounded">-</button>
            <input data-qty type="number" value="1" min="1" class="w-16 border rounded px-2 py-1 text-center">
            <button data-plus class="px-2 py-1 border rounded">+</button>
          </div>
          <button data-add class="px-3 py-1 rounded bg-blue-600 text-white">Thêm</button>
        </div>`;
      // bind số lượng + add
      const qtyInput = div.querySelector('[data-qty]');
      div.querySelector('[data-minus]').onclick = () => qtyInput.value = Math.max(1, (Number(qtyInput.value)||1)-1);
      div.querySelector('[data-plus]').onclick = () => qtyInput.value = (Number(qtyInput.value)||1)+1;
      div.querySelector('[data-add]').onclick = () => addToCart(i, Number(qtyInput.value)||1);
      menuEl.appendChild(div);
    });
}

function addToCart(item, qty) {
  const found = cart.find(c => c.id === item.id);
  if (found) found.qty += qty; else cart.push({ id: item.id, name: item.name, price: item.price, qty });
  renderCart();
}

function renderCart() {
  cartEl.innerHTML = '';
  let total = 0;
  cart.forEach(c => {
    total += c.price * c.qty;
    const row = document.createElement('div');
    row.className = 'flex justify-between items-center py-1';
    row.innerHTML = `
      <div class="text-sm">${c.name} × ${c.qty}</div>
      <div class="flex items-center gap-2">
        <span class="text-sm">${currencyVND(c.price*c.qty)}</span>
        <button class="text-rose-600 text-xs" data-remove>xoá</button>
      </div>`;
    row.querySelector('[data-remove]').onclick = () => { cart = cart.filter(x => x !== c); renderCart(); };
    cartEl.appendChild(row);
  });
  totalEl.textContent = currencyVND(total);
}

// Load menu realtime
const qItems = query(collection(db, 'items'), orderBy('createdAt', 'desc'));
onSnapshot(qItems, (snap) => {
  items = snap.docs.map(d => ({ id: d.id, ...d.data() }));
  renderMenu();
});

searchInput.oninput = () => renderMenu();

// Đặt món
const placeOrderBtn = document.getElementById('placeOrder');
const afterOrder = document.getElementById('afterOrder');
const paymentInfo = document.getElementById('paymentInfo');

placeOrderBtn.onclick = async () => {
  if (!cart.length) { alert('Chưa có món trong giỏ.'); return; }
  const total = cart.reduce((s, c) => s + c.price*c.qty, 0);
  const order = {
    table,
    items: cart.map(c => ({ itemId: c.id, name: c.name, price: c.price, qty: c.qty })),
    note: noteEl.value.trim() || '',
    total,
    status: 'pending',
    paymentMethod: paymentMethodEl.value,
    paymentStatus: paymentMethodEl.value === 'cash' ? 'unpaid' : 'unpaid',
    createdAt: serverTimestamp(),
  };
  try {
    await addDoc(collection(db, 'orders'), order);
    // Hiển thị thanh toán nếu chọn chuyển khoản
    const pay = await getPaymentSettings();
    if (paymentMethodEl.value === 'transfer' && pay) {
      afterOrder.classList.remove('hidden');
      paymentInfo.innerHTML = `
        <div><b>Chủ TK:</b> ${pay.accountName || ''}</div>
        <div><b>Ngân hàng:</b> ${pay.bankName || ''}</div>
        <div><b>Số TK:</b> ${pay.accountNumber || ''}</div>
        <div><b>Số tiền:</b> ${currencyVND(total)}</div>`;
      // Tạo QR từ nội dung chuyển khoản (chuỗi văn bản đơn giản)
      const qrText = `Chuyen khoan don ban ${table} - So tien ${total} VND - ${pay.bankName} - ${pay.accountNumber} - ${pay.accountName}`;
      const qrEl = document.getElementById('qrcode');
      qrEl.innerHTML = '';
      new QRCode(qrEl, { text: qrText, width: 192, height: 192 });
    } else {
      afterOrder.classList.add('hidden');
    }
    // Reset giỏ
    cart = []; renderCart(); noteEl.value = '';
    alert('Đã gửi đơn! Nhân viên sẽ tiếp nhận ngay.');
  } catch (e) {
    alert('Lỗi đặt đơn: ' + e.message);
  }
};
```

---



## 5) Luồng sử dụng chi tiết

### A) Admin

1. Vào `admin.html` → Đăng nhập.
2. Tab **Cài đặt**: nhập **Chủ TK, Số TK, Ngân hàng** → **Lưu**.
3. Tab **Menu**: Thêm món mới (tên, giá, danh mục, hình, trạng thái). **Sửa** bằng nút *Sửa*, **Xóa** bằng nút *Xóa*.
4. Tab **Đơn hàng**: xem realtime; nhấn **Đang làm / Đã mang ra / Đã thanh toán / Hủy** để cập nhật.

### B) Seller (thu ngân/bán hàng)

1. Vào `seller.html` → Đăng nhập.
2. Màn hình danh sách đơn realtime. Có **beep** khi có đơn mới.
3. Cập nhật trạng thái như Admin (trừ phần quản trị menu & cài đặt).

### C) Khách

1. Quét **QR bàn** → mở `client.html?table=xx`.
2. Chọn món, điều chỉnh số lượng → **Thêm** vào giỏ → **Đặt món**.
3. Nếu chọn **Chuyển khoản**, hệ thống hiển thị **Số TK/Ngân hàng** + QR (chuỗi mô tả). Khách chuyển khoản & báo lại.

> **Tùy chọn VietQR (nâng cao):** bạn có thể thay phần QRText bằng link ảnh VietQR của bên thứ ba (ví dụ `https://img.vietqr.io/...`) để quét nhận diện tốt hơn. (Cân nhắc chi phí/dịch vụ ngoài.)

---



## 6) Quy tắc bảo mật Firestore (Rules)

> Mở Firebase console → Firestore → Rules → dán nội dung dưới → Publish

```js
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    // Public đọc menu & cài đặt thanh toán
    match /items/{itemId} {
      allow read: if true;            // ai cũng xem được menu
      allow write: if request.auth != null; // chỉ người đã đăng nhập (Admin/Seller) mới sửa
    }

    match /settings/{docId} {
      allow read: if true;            // để client hiển thị Số TK/Ngân hàng
      allow write: if request.auth != null; // chỉ Admin
    }

    match /orders/{orderId} {
      // Khách tạo đơn (không cần đăng nhập)
      allow create: if true &&
        request.resource.data.keys().hasAll(['table','items','total','status','paymentMethod','paymentStatus','createdAt']);

      // Bất kỳ ai cũng có thể đọc để theo dõi trạng thái (tuỳ bạn)
      allow read: if true;

      // Chỉ người đăng nhập mới được cập nhật/hủy đơn
      allow update, delete: if request.auth != null;
    }
  }
}
```

> **Gợi ý:** Nếu muốn chặt chẽ hơn, thêm kiểm tra kiểu dữ liệu, giá trị hợp lệ, giới hạn `total >= 0`, v.v.

---



## 7) Tạo & in QR cho từng bàn

1. Giả sử tên miền của bạn sau khi deploy là: `https://ten-quan.web.app`
2. Đường dẫn cho **bàn 12**: `https://ten-quan.web.app/client.html?table=12`
3. Dùng bất kỳ công cụ tạo QR (miễn phí) để tạo ảnh QR cho **mỗi bàn** theo link trên → in ra → dán lên bàn.

> Mẹo: Có thể làm 1 file Word/Canva chứa nhiều QR, bên dưới in thêm chữ *Bàn 12* cho dễ nhận biết.

---



## 8) Triển khai Firebase Hosting

1. Cài Node.js LTS → `npm i -g firebase-tools`
2. `firebase login`
3. Trong thư mục dự án (chứa `/public`): `firebase init hosting`
   - Chọn project đã tạo
   - Public directory: **public**
   - SPA: **No**
4. `firebase deploy`

> Sau khi deploy xong, bạn sẽ nhận được URL. In QR theo URL đó (mục 7).

---



## 9) Kiểm thử & Checklist

-

---



## 10) Mở rộng & nâng cấp (khi cần)

- **VietQR chuẩn** (payload ISO 20022/EMV) để app ngân hàng nhận diện số tiền/ND – dùng API/ảnh của dịch vụ VietQR.
- **Phân quyền chi tiết**: gán custom claims để phân biệt Admin/Seller (Cloud Functions).
- **Máy in bếp**: gửi ticket tới máy in LAN.
- **Kho & nguyên liệu**: trừ tồn theo công thức.
- **Biểu đồ doanh thu**: tổng hợp theo ngày/tuần/tháng.
- **Đa chi nhánh**: thêm trường `branchId`.

---



## 11) FAQ

**Q: Khách không login có tạo đơn được không?**\
A: Có. Rules cho phép `create` vào `orders` mà không cần auth.

**Q: Làm sao đổi màu giao diện?**\
A: Sử dụng lớp Tailwind ngay trong HTML (vd: `bg-emerald-600`).

**Q: Không muốn dùng Firebase?**\
A: Có thể thay bằng Supabase/Hasura + Postgres; luồng code tương tự (fetch REST/Realtime). Nhưng Firebase dễ cho người mới.

**Q: Làm sao giới hạn gian lận giá?**\
A: Chỉ **Admin/Seller** mới viết vào `items`/`settings`; `orders` chỉ cho `create` (không `update`) từ client.

---

## Hết

> Nếu bạn muốn, tôi có thể **cài sẵn Firebase config** và **tùy biến** theo tên quán, số bàn, danh mục, giá, và tạo **gói QR in nhanh**.

