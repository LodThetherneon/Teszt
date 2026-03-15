# 💎 Teszt Webshop – Supabase + Netlify Demo

Egy egylapos (ékszer) webshop demo, amely bemutatja hogyan kötheted össze a statikus HTML/CSS/JS oldaladat a **Supabase** backenddel, bejelentkezéssel.

---

## 🚀 Gyors beállítás (3 lépés)

### 1. Hozz létre egy Supabase projektet
1. Menj ide: [https://app.supabase.com](https://app.supabase.com)
2. Kattints a **New Project** gombra
3. Add meg a projekt nevét, adatbázis jelszavát, válassz régiót (**Frankfurt** a legközelebbi)
4. Várd meg amíg az adatbázis elindul (~1-2 perc)

### 2. Másold ki az API kulcsokat
1. Supabase dashboard → **Settings → API**
2. Másold ki:
   - **Project URL** → pl. `https://abcdefgh.supabase.co`
   - **Project API Keys → anon / public** → hosszú JWT token

### 3. Illeszd be az `index.html` fájlba
Az `index.html` tetején cseréld ki ezt a két sort:

```javascript
const SUPABASE_URL = 'ILLESZD_BE_A_SUPABASE_URL_CIMET';
const SUPABASE_ANON_KEY = 'ILLESZD_BE_AZ_ANON_KEY_KULCSOT';
```

Példa:
```javascript
const SUPABASE_URL = 'https://abcdefgh.supabase.co';
const SUPABASE_ANON_KEY = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...';
```

---

## 📦 Termékek betöltése az adatbázisból

A Supabase **SQL Editor**-ban futtasd le:

```sql
CREATE TABLE products (
  id SERIAL PRIMARY KEY,
  name TEXT NOT NULL,
  price INTEGER NOT NULL,
  emoji TEXT,
  shop_id INTEGER DEFAULT 1
);

INSERT INTO products (name, price, emoji) VALUES
  ('Ezüst gyűrű', 4900, '💍'),
  ('Kézzel font nyaklánc', 6500, '📿'),
  ('Ametiszt fülbevaló', 3200, '🔮'),
  ('Rozé arany karkötő', 8900, '🌸');

ALTER TABLE products ENABLE ROW LEVEL SECURITY;
CREATE POLICY "Mindenki olvashat" ON products FOR SELECT USING (true);
```

Ezután az `index.html`-ben a statikus kártyák helyett:

```javascript
async function loadProducts() {
  const { data: products } = await supabaseClient
    .from('products')
    .select('*')
    .eq('shop_id', 1);

  const grid = document.getElementById('product-grid');
  grid.innerHTML = '';
  products.forEach(product => {
    grid.innerHTML += `
      <div class="product-card">
        <div class="card-img">${product.emoji}</div>
        <div class="card-body">
          <div class="card-title">${product.name}</div>
          <div class="card-price">${product.price.toLocaleString('hu-HU')} Ft</div>
          <button class="btn-cart" onclick="addToCart('${product.name}')">Kosárba</button>
        </div>
      </div>`;
  });
}
loadProducts();
```

---

## 🌐 Netlify deploy

1. Pushold a kódot GitHub-ra (már ott van!)
2. [Netlify](https://netlify.com) → **Add new site → Import from Git**
3. Válaszd ki a `Teszt` repót → **Deploy** ✅
4. Supabase-ben: **Authentication → URL Configuration → Site URL** = Netlify címed

---

## 🔐 Biztonság

Az `anon` kulcs biztonságos frontenden, mert az **RLS** védi az adatbázist. Soha ne rakd ki a `service_role` kulcsot!
