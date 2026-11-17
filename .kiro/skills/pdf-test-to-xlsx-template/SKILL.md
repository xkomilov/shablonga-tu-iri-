---
name: pdf-test-to-xlsx-template
description: Uzbek tilidagi test savollari PDF faylini shablon.xlsx (Question, A, B, C, D, Correct, Comment) formatiga aylantirish, imlo va buzilgan matnlarni tuzatish, natija/ papkasiga saqlash va yangi branchga commit/push qilish.
---

# PDF test savollarini shablon.xlsx ga aylantirish

## Kirish
Foydalanuvchi GitHub PDF havolasini beradi, masalan
`https://github.com/<owner>/<repo>/blob/main/<fayl>.pdf`. Natija `natija/<manba-nomi>.xlsx` bo'ladi.

## Qoidalar
- Correct va Comment ustunlari bo'sh qoladi.
- Savol raqamlari chiqishda bo'lmasligi mumkin, foydalanuvchi bunga e'tiroz bildirmagan.
- Fayl nomidagi savol soni PDF dagi haqiqiy sondan farq qilsa, PDF dagi barcha savollar saqlanadi va buni xabarda aytiladi.
- Asosiy branch (`main`) ga to'g'ridan-to'g'ri commit qilinmaydi. Har bir fayl uchun `origin/main` dan yangi branch ochiladi: `natija/<manba-nomi>`.
- Mahalliy klon eski bo'lishi mumkin. PDF ni doim `raw.githubusercontent.com` dan yuklang.

## 1-qadam: PDF ni yuklash va matnini olish
Har bir bash chaqiruvida kutubxonani o'rnating va skriptni shu chaqiruvda ishga tushiring. `/tmp/pylibs` ba'zan chaqiruvlar orasida yo'qoladi.

```bash
mkdir -p /projects/sandbox/_work && cd /projects/sandbox/_work
curl -sSfL -o f.pdf "https://raw.githubusercontent.com/<owner>/<repo>/main/<fayl>.pdf"
python3 -m pip install -q --target /tmp/pylibs openpyxl pdfplumber >/tmp/pip.log 2>&1
PYTHONPATH=/tmp/pylibs python3 -c "import pdfplumber; \
  print('\n'.join((p.extract_text() or '') for p in pdfplumber.open('f.pdf').pages))" > f.txt
```

## 2-qadam: Savollarni ajratish
Savol boshida `N.` (raqam ketma-ketligi) keladi. Ketma-ketlik bo'yicha tekshiring: keyingi raqam faqat kutilgan raqam bo'lsa qabul qilinadi.

Ikki xil variant bor:
- **Variant A:** variantlar `- ` bilan boshlanadi. Har bir `- ` qatori alohida variant.
- **Variant B:** variantlar prefiksiz. Har bir savol blokining oxirgi 4 qatori variant. Blok 5 qatordan kam bo'lsa, xabar bering. Bu variant bilan ko'p qatorli variantni ajratish qiyin, shuning uchun natijani tekshiring.

Tekshirish qoidalari:
- Savol boshida raqam bo'lsa va raqam bo'sh qatorda bo'lsa (`144.` kabi), matn keyingi qatordan olinadi.
- Savol matni yoki variant bir necha qatorga bo'linsa, qatorlar bo'shliq bilan birlashtiriladi.
- Savol matni `- ` bilan boshlangan qator bo'lsa (Q176 kabi), u variant emas, savol davomi. Variantlar sonini tekshirib, 4 tadan oshsa, birinchi "variant"ni savol matniga qo'shing.
- Variantlar soni 4 dan farq qilsa, sababini tekshirib, qo'lda tuzating (ko'p qatorli variant birlashtiriladi).

## 3-qadam: Imlo va matnni tuzatish
Quyidagilar tuzatiladi:
- **Apostrof:** harf ichidagi `'`, `’`, `ʻ`, `ʼ`, `` ` `` belgilari `‘` (U+2018) ga o'zgartiriladi.
- **Kirill harflari:** lotin so'z ichida kirill o'xshash harflar (`о а е с р х у к н т в м`, katta harflar ham) lotinga almashtiriladi. Kirill harflari yolg'iz so'zda ham bo'lsa, qo'lda tuzatiladi (masalan, `куйдирги` → `kuydirgi`).
- **Ukrain/rus harflari:** `і`→`i`, `щ`/`ш`→`sh`, `ь` lotin so'z ichidan olib tashlanadi.
- **Birlashgan so'zlar:** `aholisoniga,ishlabchiqarish` kabi so'zlar qo'lda ajratiladi. Uzun so'zlar ro'yxati bilan (≥26 belgi, raqamsiz) topiladi.
- **Vergul:** verguldan keyin bo'shliq yo'q bo'lsa qo'shiladi. Vergul oldidagi bo'shliq olib tashlanadi.
- **Qavs:** `so'z(` → `so'z (`.
- **Keng tarqalgan imlo:** `xujjat→hujjat`, `muxit→muhit`, `ventelatsiya→ventilyatsiya`, `texnalogiya→texnologiya`, `zaxarlanish→zaharlanish`, `xisobot→hisobot`, `xavo→havo`, `xarorat→harorat`, `labarator→laborator`.

Bu imlo lug'ati to'liq emas. Har bir faylda noyob so'zlarning ro'yxati chiqariladi, lekin barchasi qo'lda tekshirilmaydi. Yakuniy xabarda bu cheklov aytiladi.

## 4-qadam: Takroriy variantlar
Bir savolda bir xil variantlar bo'lsa:
- Birinchi variant o'zgarmaydi (to'g'ri javob bo'lishi mumkin).
- Keyingi takroriy variant o'xshash, lekin noto'g'ri variant bilan almashtiriladi. Masalan, `chivin` → `it` yoki `pashsha`.
- To'g'ri javob matni o'zgartirilmaydi.
- Almashtirilgan savollar xabarda sanab o'tiladi.

## 5-qadam: Qo'lda tuzatish
Buzilgan savollar (birlashgan matn, kimyoviy formula, tushunarsiz variant) uchun `Q_OVR` / `O_OVR` lug'atlarida qo'lda matn yoziladi. Har bir o'zgarish xabarda ko'rsatiladi va PDF bilan tekshirish so'raladi.

## 6-qadam: xlsx yaratish
`shablon.xlsx` faylining sarlavhasi saqlanadi (`Question, A, B, C, D, Correct, Comment (Faqat to'g'ri javob uchun)`). Eski qatorlar o'chiriladi va savollar yoziladi. F va G ustunlari bo'sh qoladi.

Natija: `natija/<manba-nomi>.xlsx`

Tekshirish:
- Qator soni PDF dagi savol soniga teng.
- Har qatorda 4 ta variant bor.
- F va G ustunlari bo'sh.
- Kirill harflari qolmagan.
- Dublikat variantlar yo'q.

## 7-qadam: Commit va push
Mahalliy klon `main` da bo'lsa ham, natijani `origin/main` dan yangi branchga commit qiling. Asosiy repoda untracked `natija/` fayllarga tegmang.

```bash
cd <repo>
git fetch -q origin main
git ls-remote --heads origin 'natija/<nom>*'          # nomi band emasligini tekshirish
git worktree add -q <tmp-wt> -b natija/<manba-nomi> origin/main
mkdir -p <tmp-wt>/natija
cp natija/<manba-nomi>.xlsx <tmp-wt>/natija/
cd <tmp-wt>
git add natija/<manba-nomi>.xlsx
git commit -q -m "Add <manba-nomi> test questions in template format"
git push -q -u origin natija/<manba-nomi>
cd <repo> && git worktree remove <tmp-wt>
gh api "repos/<owner>/<repo>/contents/natija/<manba-nomi>.xlsx?ref=natija/<manba-nomi>" --jq '.name'
```

Agar `git ls-remote` branch topsa, yangi nom tanlang va buni foydalanuvchiga ayting.

## 8-qadam: Yakuniy xabar
Quyidagilarni yozing:
- Fayl havolasi: `https://github.com/<owner>/<repo>/blob/natija/<manba-nomi>/natija/<manba-nomi>.xlsx`
- PR havolasi: `https://github.com/<owner>/<repo>/pull/new/natija/<manba-nomi>`
- Savollar soni va PDF dagi haqiqiy soni (farq bo'lsa, sababi)
- Qo'lda tuzatilgan savollar ro'yxati (raqam bilan)
- Taxminiy tuzatishlar (masalan, almashtirilgan variantlar), ularni PDF bilan tekshirish so'rovi
- Imlo to'liq tekshirilmaganligi haqida ochiq eslatma

Oxirida scratch papkani o'chiring: `rm -rf /projects/sandbox/_work`

## Cheklovlar
- Imlo to'liq tekshirilmaydi. Foydalanuvchi aniq savol raqamlarini aytsa, faqat o'sha savollar ko'rib chiqiladi.
- PDF dagi xato ma'lumotlar (noto'g'ri javob, kimyoviy formula) tuzatilmaydi, faqat belgilanadi.
- Dublikat variantni almashtirish to'g'ri javobni o'zgartirmaydi, lekin almashtirilgan variant mazmunan to'g'ri bo'lib qolishi mumkin. Buni foydalanuvchi tekshirishi kerak.
