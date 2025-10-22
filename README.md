https://rstr.in/wtjexky6vgwx87/my-library/Ts_xib7lj7j

# gittisak-lnc README_SETUP_FULL

## สรุปงานที่จะทำ
ไฟล์ที่รวมในโปรเจกต์:
- `frontend/index.html` (client UI + Firebase client auth)
- `backend/index.js` (Node/Express server — รวม Facebook OAuth exchange + webhook + Firebase Admin verify token)
- `mcp/mcp.py` (FastAPI proxy to LLM)
- `Dockerfile` (frontend build / nginx)
- `docker-compose.yml` (frontend, backend, mcp)
- `.env.example` (environment variables)
- `firebase.rules` (ตัวอย่าง Firestore security rules)
- `README_SETUP.md` (คู่มือสั้น)

---

## STEP-BY-STEP (ละเอียด พร้อมตำแหน่งที่ต้องกรอก)
ต่อไปนี้เป็นคำแนะนำทีละขั้นตอน (พร้อมภาพตัวอย่างตำแหน่งที่ต้องคลิก — ผมใส่ placeholder ให้ คุณสามารถแทนที่ด้วย screenshot ของคุณเองได้):

### A. สร้าง Firebase Project และตั้งค่า Authentication
1. ไปที่ https://console.firebase.google.com และกด **Add project** → ตั้งชื่อโปรเจกต์ (เช่น `gitmint-agent`) → Continue.
2. เปิด **Authentication** ในแถบซ้าย → เมนู **Sign-in method** → เปิด **Google** (Enable) → Save.
   - (Screenshot placeholder: `screenshots/firebase-auth-google.png`)
3. เปิด **Facebook** sign-in method:
   - สร้าง Facebook App ที่ developers.facebook.com → จากนั้นใน Firebase console ใส่ **App ID** และ **App Secret** ที่ได้.
   - ตั้ง **OAuth redirect URI** ใน Facebook app: `https://<PROJECT>.firebaseapp.com/__/auth/handler` (เอา `<PROJECT>` เป็น projectId ของคุณ)
   - (Screenshot placeholder: `screenshots/firebase-auth-facebook.png`)
4. เพิ่ม **OIDC provider** สำหรับ LINE (Authentication → Sign-in method → Add provider → OIDC):
   - Provider ID: `oidc.line`
   - Issuer: `https://access.line.me`
   - Client ID: (LINE Channel ID)
   - Client secret: (LINE Channel Secret)
   - Scopes: `openid profile email`
   - (Screenshot placeholder: `screenshots/firebase-auth-line-oidc.png`)
5. ใน **Project settings** → **General** → ภายใต้ "Your apps" ให้เพิ่ม Web app แล้วคัดค่าของ `firebaseConfig` (apiKey, projectId, authDomain...) — นำไปใส่ใน `frontend/index.html` ตัวแปร `FIREBASE_CONFIG`.
   - (Screenshot placeholder: `screenshots/firebase-config.png`)

---
### B. ตั้งค่า Firestore
1. ไปที่ Firestore → Create database → เลือก Production/Start in native mode → เลือก location.

2. เปิดแท็บ **Rules** แล้ววางค่า `firebase.rules` ตัวอย่างที่อยู่ใน repo (หรือแก้ให้เหมาะสมกับ requirement).
   - (Screenshot placeholder: `screenshots/firestore-rules.png`)

---

### C. สร้าง Facebook App (Login + Page) และตั้งค่าการอนุญาต
1. ไปที่ https://developers.facebook.com → My Apps → Create App → เลือกประเภท (Business หรือ Others) → Create.
2. ใน Dashboard: Add product → Facebook Login → ตั้งค่า **Valid OAuth Redirect URIs** เป็น `https://<PROJECT>.firebaseapp.com/__/auth/handler` (หรือ callback ที่ backend ถ้าคุณใช้ server flow).
3. ขอ Permissions ที่ต้องใช้: `pages_manage_metadata`, `pages_read_engagement`, `pages_read_user_content`, `pages_show_list`, `pages_manage_posts`, `pages_messaging` (ขอเฉพาะที่จำเป็น)
4. สร้าง **Page Access Token** (ผ่าน Graph API Explorer หรือผ่านการแลก short-lived token → exchange เป็น long-lived token) — โค้ดตัวอย่าง server ผมจะแสดงการแลก long-lived token และดึง Page access token.
   - (Screenshot placeholder: `screenshots/fb-app-setup.png`)

---

### D. สร้าง LINE Channel (Login)
1. ไปที่ LINE Developers Console → Create Provider → Create Channel (LINE Login หรือ Messaging API ขึ้นกับการใช้งาน)
2. ใส่ Redirect URI ใน LINE console ให้ชี้ไปที่ Firebase OIDC redirect หรือ backend callback ที่คุณจะใช้
   - Example redirect: `https://<PROJECT>.firebaseapp.com/__/auth/handler`
3. คัดค่าที่จำเป็น (Channel ID, Channel secret) ไปใส่ใน Firebase OIDC provider
   - (Screenshot placeholder: `screenshots/line-channel.png`)

---

### E. เตรียม Firebase Admin (service account)
1. Firebase Console → Project settings → Service accounts → Generate new private key → ดาวน์โหลดไฟล์ `serviceAccountKey.json`.
2. ทำ base64 encode แล้วใส่ค่าใน `.env` เป็น `FIREBASE_ADMIN_KEY` (เพื่อใช้ใน Docker env โดยไม่เก็บ JSON ตรงๆ)
   - คำสั่งตัวอย่าง: `cat serviceAccountKey.json | base64 -w 0`

---
