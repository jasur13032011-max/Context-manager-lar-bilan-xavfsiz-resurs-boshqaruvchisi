# Context-manager-lar-bilan-xavfsiz-resurs-boshqaruvchisi
Python tilida `with` operatori resurslarni (fayllar, ma'lumotlar bazasi ulanishlari, tarmoq soketlari) boshqarishni xavfsiz va qulay qiladi. Quyida siz so'ragan ikkala usul (klassga asoslangan va generator yordamida) bo'yicha to'liq tushuntirish va kod namunalari keltirilgan.

---

### 1. Klassga asoslangan Context Manager

Klass yordamida context manager yaratish uchun `__enter__` va `__exit__` maxsus dunder metodlarini amalga oshirish kerak.

```python
class ManagedResource:
    def __init__(self, name):
        self.name = name

    def __enter__(self):
        print(f"[Klass] '{self.name}' resursi ochildi (__enter__)")
        return self  # 'as' kalit so'zi orqali olinadigan obyektni qaytaramiz

    def __exit__(self, exc_type, exc_value, traceback):
        print(f"[Klass] '{self.name}' resursi yopildi / tozalandi (__exit__)")
        
        # False qaytarish orqali istisnolar tashqariga uzatilishini ta'minlaymiz.
        # Agar True qaytarilsa, blichdagi xatolik yutib yuboriladi (tashqariga chiqmaydi).
        return False

    def action(self):
        print(f"[Klass] Resurs ishlamoqda: {self.name}")

```

#### `with` bloki yordamida ishlatish va istisno (exception) tekshiruvi:

```python
print("--- Klassga asoslangan test boshlandi ---")
try:
    with ManagedResource("Fayl-1") as res:
        res.action()
        print("Blok ichida xatolik keltirib chiqaramiz...")
        raise ValueError("Ataylab tashlangan xatolik!")
except ValueError as e:
    print(f"Tashqarida ushlangan xato: {e}")
print("--- Test tugadi ---\n")

```

**Natija tahlili:** `raise ValueError` chaqirilganiga qaramay, `__exit__` metodi ishga tushdi va resurs tozalanishi kafolatlandi.

---

### 2. `__exit__` dan `False` va `True` qaytarishning farqi

* **`False` (yoki hech narsa qaytarmaslik - `None`):** Agar `with` bloki ichida xatolik yuz bersa, u `__exit__` metodiga uzatiladi. Metod `False` qaytarsa, xatolik tozalash ishlari tugagach, blokka tashqariga (yuqoriga) o'tkaziladi. Bu eng xavfsiz va tavsiya etilgan usul.
* **`True`:** Agar `__exit__` metodi `True` qaytarsa, Python blok ichida yuzaga kelgan istisnoni **jimgina yutib yuboradi** (ignorat qiladi) va dastur xatosiz davom etadi. Bu faqatgina maxsus holatlarda (masalan, xatolarni maxsus boshqarish logikasida) ishlatiladi.

---

### 3. Generator orqali `@contextlib.contextmanager` yordamida amalga oshirish

Python o'zining `contextlib` modulidagi dekorator yordamida klass yozmasdan, generator funksiya orqali context manager yaratishga imkon beradi.

* `yield` dan oldingi qism — sozlash (`__enter__` vazifasi).
* `yield` dan keyingi qism (odatda `finally` bloki ichida) — tozalash (`__exit__` vazifasi).

```python
from contextlib import contextmanager

@contextmanager
def managed_resource_gen(name):
    print(f"[Generator] '{name}' resursi ochildi (yield dan oldin)")
    try:
        yield name  # 'as' orqali qaytariladigan qiymat
    finally:
        # try/finally xatolik bo'lsa ham tozalash kafolatlanishini ta'minlaydi
        print(f"[Generator] '{name}' resursi yopildi / tozalandi (yield dan keyin)")

```

#### Generatordan foydalanish va xatolikni tekshirish:

```python
print("--- Generatorga asoslangan test boshlandi ---")
try:
    with managed_resource_gen("Tarmoq-Ulanishi") as res:
        print(f"Resurs nomi bilan ishlayapmiz: {res}")
        raise RuntimeError("Generator ichidagi xatolik!")
except RuntimeError as e:
    print(f"Tashqarida ushlangan xato: {e}")
print("--- Test tugadi ---")

```

---

### Xulosa va Taqqoslash

Ikkala yondashuv ham bir xil natijani beradi:

1. **Ochish/Sozlash:** Blokga kirishdan oldin resurs tayyorlanadi.
2. **Xavfsizlik:** Blok ichida kutilmagan xatolik (`Exception`) yuzaga kelsa ham, tozalash bosqichlari (`__exit__` yoki `finally`) bajarilishi kafolatlanadi.
3. **Qulaylik:** Klasslar ko'proq holatni boshqarish va murakkab logika uchun yaxshi bo'lsa, `@contextlib.contextmanager` dekoratori qisqa va sodda resurslarni boshqarishda kodni juda o'qishli qiladi.
