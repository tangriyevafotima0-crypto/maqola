# Master Prompt: AutoRepo AI - macOS uchun Sun'iy Intellektli GitHub, Drive va Fayl Boshqaruv Dasturi

## 1. Dastur Falsafasi va Maqsadi (Core Vision)
Sizning vazifangiz macOS (Apple Silicon/Intel) uchun **SwiftUI va Python backend** (yoki butunlay mahalliy Python desktop app - masalan, PyQt6/PySide6 orqali) ishlatilgan, o'ta silliq (smooth), tezkor va intellektual desktop dasturini yaratishdir. Bu dasturning asosiy maqsadi foydalanuvchining belgilangan papkasidagi (masalan, `Downloads/AutoRepo_Staging`) loyiha versiyalarini analiz qilish, Gemini API orqali ularni to'g'ri tartiblash, avtomatik ravishda GitHub'ga repozitoriy ochib (yoki ulanganga) push qilish, Google Drive'ga zaxiralash va oxirida Mac xotirasidan xavfsiz o'chirishdir. Barcha jarayonlar aqlli, qulay (User-Friendly) va xavfsiz bo'lishi shart.

## 2. Asosiy Texnik Talablar va Arxitektura

### A. Texnik Stek
- **Frontend/UI:** SwiftUI (macOS native his etilishi uchun) yoki PyQt6/PySide6 (agar hamma mantiqni bitta Python loyihada saqlash maqsadga muvofiq bo'lsa). Dastur ko'rinishi macOS dizayn tiliga (Human Interface Guidelines) to'liq mos kelishi, chalg'ituvchi unsurlarsiz (minimalist), smooth animatsiyalar bilan ishlashi kerak.
- **Backend/Mantiq:** Python 3.10+
- **API integratsiyalari:** 
  - **Gemini API:** Bir nechta modellarni (Gemini 1.5 Flash, 1.5 Pro) navbat bilan yoki yuklamaga qarab (fallback/exchange mexanizmi) ishlatish imkoniyati. Token limiti yetsa Flash tezlik uchun, qiyin analizlar uchun Pro ishlatilishi kerak.
  - **GitHub API / Git CLI:** Repozitoriy ochish, SSH/HTTPS ulash, branch va taglarni boshqarish.
  - **Google Drive API:** Arxivlangan fayllarni belgilangan papkaga yuklash.
- **O'rnatish:** Dastur o'zini o'zi o'rnata oladigan `run.sh` yoki `install.sh` skriptiga ega bo'lishi kerak. Bu skript kerakli Python muhitini (venv), kutubxonalarni o'rnatib, Mac'ning `Applications` papkasiga `.app` (yoki Shortcut) yaratishi shart.

### B. Funksionallik va Ish Jarayoni (Workflow)
1. **Ruxsat (Permissions) va Sandbox:** 
   - Dastur faqat va faqat foydalanuvchi ko'rsatgan 1 ta maxsus papkaga (masalan, "Dedicated Staging Folder") kirish ruxsatiga ega bo'ladi. U butun tizim bo'ylab aylanmaydi.
   - Mac Settings'dan Full Disk Access yoki xavfli ruxsatlar so'ralmasligi uchun Folder Picker (NSOpenPanel) ishlatiladi.
2. **AI yordamida Analiz va Tartiblash:**
   - Masalan, `SpineGuard` ning 10 ta versiyasi (v1, v2, ... v10, old_backup) tashlangan bo'lsa, Gemini API ularning nomini, vaqtini, qisqacha mazmunini analiz qilib, qaysi biri birinchi, qaysi biri asosiy va oxirgi ekanligini xronologik tartibga soladi.
   - Gemini API shuningdek, papka ichida loyihaga aloqasi yo'q bo'lgan (general/ortiqcha) fayllarni ham aniqlab: *"Ushbu fayllarni push qilamizmi yoki boshqa joyga olib qo'yasizmi?"* deb aqlli maslahat beradi.
3. **GitHub Avtomatizatsiyasi:**
   - Foydalanuvchidan GitHub Token (yoki OAuth) bir marta so'raladi va xavfsiz (Keychain'da) saqlanadi.
   - Gemini aniqlagan tartib bo'yicha: yangi repo ochiladi (yoki mavjudiga bog'lanadi), versiyalar ketma-ket branch/tag qilib yoki xronologik commit qilib push qilinadi (og'ir kesh fayllar avtomat .gitignore qilinib tozalangach).
4. **Google Drive Integratsiyasi:**
   - Barcha jarayon tugagach, toza kodlar ZIP arxiv qilinib, orqa fonda Drive'ga yuklanadi.
5. **Aqlli O'chirish mexanizmi (Smart Deletion & Double-Warning):**
   - Push va Drive yuklash to'liq (100%) tasdiqlangandan so'ng dastur Mac xotirasini bo'shatishga o'tadi.
   - **Lekin hech narsa to'g'ridan-to'g'ri o'chirilmaydi.** Dastur ichidagi mini-Finder (fayl brauzer ro'yxati) ko'rinishida qaysi fayllar o'chirilayotgani ro'yxati chiqadi.
   - Ikki marta (Double Warning) tasdiq o'tish zarur: "Hammasi Drive'da xavfsiz. Haqiqatan ham Mac'dan o'chiraymi?" (Yes/Cancel).

## 3. UI/UX (Foydalanuvchi Tajribasi)
- **Settings/Onboarding:** Dastur ilk bor ochilganda GitHub Token, Gemini API (bir nechta zaxira kalitlari kiritish imkoni) va Google Drive ulanishi uchun sodda qadamli (stepper) oyna bo'lishi kerak.
- **Main Dashboard:** 
  - Tepada "Target Folder" ko'rinib turadi.
  - O'rtada Gemini tomonidan analiz qilingan fayllar va versiyalar "timeline" (xronologiya) tarzida chiroyli vizualizatsiya qilinadi.
  - Aloqasi yo'q deb topilgan fayllar alohida "Orphan / Unrelated Files" deb sariq rangda e'tiborga tortiladi.
- **Smooth & Overwhelm-free:** Holatlar (States) juda aniq: "Analiz qilinmoqda", "GitHub'ga yuklanmoqda", "Drive'ga zaxiralanmoqda", "O'chirish tasdig'i kutilmoqda". Har bir qadam o'z vaqtida, asabga tegmaydigan, mantiqiy tarzda o'tadi.

## 4. Kodlash Bo'yicha Asosiy Topshiriqlar (Deliverables)
Sizdan quyidagi to'liq kod strukturasini va fayllarni generatsiya qilish talab qilinadi:
1. **Dastur Arxitekturasi:** Backend (Python) va Frontend qanday bog'lanishi (REST API, Sockets yoki bevosita py-bridge).
2. **AI Logic Module:** Gemini API (Google Generative AI SDK) ni ishlatuvchi, rate-limit ga tushsa yoki model xato qilsa avtomat zaxira modelga (fallback) o'tuvchi va papka ichini tahlil qiluvchi mantiq (`ai_analyzer.py`).
3. **Git & Drive Module:** Avtomat repo yaratuvchi, commit va push qiluvchi, shuningdek zip qilib Drive'ga yuboruvchi mantiq.
4. **O'rnatish Skripti:** Barcha bog'liqliklarni o'rnatib, Mac ish stoli uchun ishlashga tayyor holda taqdim etuvchi `run.sh` yoki `setup.py`.

*Iltimos, har bir fayl uchun clean-code tamoyillariga asoslangan, to'liq ishlaydigan kodlarni, ayniqsa AI fallback mexanizmi va "Double-Warning" UI qismini e'tibor bilan yozing.*
