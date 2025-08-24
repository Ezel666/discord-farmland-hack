# [Discord] Farmland Coin Hack

![Logo](https://i.imgpost.net/2025/08/23/Cj6xK.png)

---

## 📌 Description / Açıklama

**TR:**  
Bu script, tarayıcı konsoluna yapıştırılarak çalıştırılır. Kod çalıştıktan sonra oyun kaydınızı değiştirir. Oyunu yeniden açtığınızda değişiklikler uygulanmış olacaktır.

**EN:**  
This script is intended to be run inside the browser console. After pasting and executing the code, it modifies your game save. When you restart the game, the changes will take effect.

---

## ⚙️ Installation & Usage / Kurulum ve Kullanım

<details>
<summary><strong><img src="https://flagcdn.com/w20/tr.png" width="20"/> Türkçe Talimatlar — tıklayarak aç</strong></summary>

1. Oyuna giriş yapın. (Chrome veya Discord fark etmez)  
2. **Ctrl + Shift + I** tuşlarına basarak Geliştirici Araçlarını açın.  
3. **Ctrl + Shift + C** ile oyun ekranını seçip tıklayın.  
4. **Console** sekmesine geçin.  
5. Aşağıdaki kodu kopyalayın ve yapıştırın, ardından **Enter**’a basın.  

<details>
<summary><strong>📜 Kodu Görmek için tıkla </strong></summary>

```javascript
(() => {
  const RE = /\/server\/v2\/storage\b/i;
  const EM = { e: 3, m: 999 }; // ≈ 999 milyar
  let armed = true;

  const fetch0 = window.fetch.bind(window);
  const XHROpen0 = XMLHttpRequest.prototype.open;
  const XHRSend0 = XMLHttpRequest.prototype.send;
  const beacon0 = navigator.sendBeacon ? navigator.sendBeacon.bind(navigator) : null;

  const looksJSON = s => typeof s === 'string' && s.trim().startsWith('{');
  function setAllMoney(save){
    if (!save?.worldSaves) return 0;
    let touched = 0;
    for (const w of save.worldSaves){
      if (!w) continue;
      if (w.balance) { w.balance = { e: EM.e, m: EM.m }; touched++; }
      if (Array.isArray(w.parcelWallets)){
        for (const p of w.parcelWallets){
          if (p?.wallet){ p.wallet = { e: EM.e, m: EM.m }; touched++; }
        }
      }
    }
    return touched;
  }
  function patchBodyText(txt, tag){
    try{
      const body = JSON.parse(txt);
      const objs = Array.isArray(body.objects) ? body.objects : [];
      let patched = 0;
      for (const o of objs){
        if (!o || typeof o.value !== 'string' || !looksJSON(o.value)) continue;
        try {
          const save = JSON.parse(o.value);
          patched += setAllMoney(save);
          o.value = JSON.stringify(save);
        } catch {}
      }
      if (patched>0) console.log(`✅ [money] ${tag}: set money in ${patched} place(s) → e=${EM.e}, m=${EM.m}`);
      return patched>0 ? JSON.stringify(body) : null;
    } catch { return null; }
  }
  function unhookAll(){
    window.fetch = fetch0;
    XMLHttpRequest.prototype.open = XHROpen0;
    XMLHttpRequest.prototype.send = XHRSend0;
    if (beacon0) navigator.sendBeacon = beacon0;
    console.log('🧹 [money] unhooked');
  }

  // FETCH
  window.fetch = async (input, init={}) => {
    const url = typeof input==='string' ? input : input?.url || '';
    const method = (init.method || (typeof input!=='string' ? input.method : 'GET') || 'GET').toUpperCase();
    if (!RE.test(url) || method!=='PUT' || !armed) return fetch0(input, init);

    let txt = null;
    if (typeof init.body === 'string') txt = init.body;
    else if (typeof input!=='string' && input instanceof Request) txt = await input.clone().text();

    const patched = txt ? patchBodyText(txt, 'fetch') : null;
    if (patched){
      if (typeof init.body === 'string') init.body = patched;
      else if (typeof input!=='string' && input instanceof Request){
        const headers = new Headers(input.headers||{}); headers.set('content-type','application/json');
        input = new Request(input, { method, headers, body: patched });
      }
      const res = await fetch0(input, init);
      armed = false; unhookAll();
      return res;
    }
    return fetch0(input, init);
  };

  // XHR
  XMLHttpRequest.prototype.open = function(method, url, ...rest){
    this.__m = (method||'GET').toUpperCase();
    this.__u = url||'';
    return XHROpen0.call(this, method, url, ...rest);
  };
  XMLHttpRequest.prototype.send = function(body){
    const isPUT = (this.__m||'GET')==='PUT';
    const hit = RE.test(this.__u||'');
    if (!armed || !isPUT || !hit) return XHRSend0.call(this, body);

    const tryPatch = (text) => {
      const patched = patchBodyText(text, 'xhr');
      if (patched){
        armed = false; unhookAll();
        return XHRSend0.call(this, patched);
      }
      return XHRSend0.call(this, body);
    };

    if (typeof body === 'string') return tryPatch(body);
    if (body instanceof ArrayBuffer) {
      try { return tryPatch(new TextDecoder().decode(body)); } catch { return XHRSend0.call(this, body); }
    }
    if (body && typeof Blob !== 'undefined' && body instanceof Blob) {
      const fr = new FileReader();
      fr.onload = () => {
        const txt = typeof fr.result === 'string' ? fr.result : '';
        const patched = patchBodyText(txt, 'xhr/blob');
        if (patched){
          armed = false; unhookAll();
          XHRSend0.call(this, patched);
        } else {
          XHRSend0.call(this, body);
        }
      };
      fr.readAsText(body);
      return;
    }

    return XHRSend0.call(this, body);
  };

  if (beacon0){
    navigator.sendBeacon = (url, data) => {
      if (!armed || !RE.test(url)) return beacon0(url, data);
      if (typeof data === 'string'){
        const patched = patchBodyText(data, 'beacon');
        if (patched){ armed = false; unhookAll(); return beacon0(url, patched); }
      }
      return beacon0(url, data);
    };
  }

  console.log('💰 Money successfully added - Click the game settings button to trigger the save, then close and reopen the game..');
  console.log('💰 Para başarıyla eklendi - Oyunun ayar butonuna tıklayarak save almasını tetikle sonra oyunu kapatıp aç..');
})();
```


</details>


6. Eğer yapıştıramıyorsanız, önce konsola `allow pasting` yazıp Enter’a basın, ardından tekrar deneyin.  
7. **ÖNEMLİ!** : Kodu çalıştırdıktan sonra oyunun **Ayarlar** butonuna tıklayın ki değişiklikler kaydedilsin.  
8. Oyunu kapatıp tekrar açın — değişiklikler artık aktif olacaktır.  

</details>

---

<details>
<summary><strong><img src="https://flagcdn.com/w20/gb.png" width="20"/> English Instructions — click to expand</strong></summary>

1. Log in to the game. (Chrome or Discord doesn’t matter)  
2. Press **Ctrl + Shift + I** to open Developer Tools.  
3. Use **Ctrl + Shift + C** to select and click the game screen.  
4. Switch to the **Console** tab.  
5. Copy and paste the following code, then press **Enter**.  

<details>
<summary><strong>📜 Click to expand the code</strong></summary>

```javascript
(() => {
  const RE = /\/server\/v2\/storage\b/i;
  const EM = { e: 3, m: 999 }; // ≈ 999 milyar
  let armed = true;

  const fetch0 = window.fetch.bind(window);
  const XHROpen0 = XMLHttpRequest.prototype.open;
  const XHRSend0 = XMLHttpRequest.prototype.send;
  const beacon0 = navigator.sendBeacon ? navigator.sendBeacon.bind(navigator) : null;

  const looksJSON = s => typeof s === 'string' && s.trim().startsWith('{');
  function setAllMoney(save){
    if (!save?.worldSaves) return 0;
    let touched = 0;
    for (const w of save.worldSaves){
      if (!w) continue;
      if (w.balance) { w.balance = { e: EM.e, m: EM.m }; touched++; }
      if (Array.isArray(w.parcelWallets)){
        for (const p of w.parcelWallets){
          if (p?.wallet){ p.wallet = { e: EM.e, m: EM.m }; touched++; }
        }
      }
    }
    return touched;
  }
  function patchBodyText(txt, tag){
    try{
      const body = JSON.parse(txt);
      const objs = Array.isArray(body.objects) ? body.objects : [];
      let patched = 0;
      for (const o of objs){
        if (!o || typeof o.value !== 'string' || !looksJSON(o.value)) continue;
        try {
          const save = JSON.parse(o.value);
          patched += setAllMoney(save);
          o.value = JSON.stringify(save);
        } catch {}
      }
      if (patched>0) console.log(`✅ [money] ${tag}: set money in ${patched} place(s) → e=${EM.e}, m=${EM.m}`);
      return patched>0 ? JSON.stringify(body) : null;
    } catch { return null; }
  }
  function unhookAll(){
    window.fetch = fetch0;
    XMLHttpRequest.prototype.open = XHROpen0;
    XMLHttpRequest.prototype.send = XHRSend0;
    if (beacon0) navigator.sendBeacon = beacon0;
    console.log('🧹 [money] unhooked');
  }

  // FETCH
  window.fetch = async (input, init={}) => {
    const url = typeof input==='string' ? input : input?.url || '';
    const method = (init.method || (typeof input!=='string' ? input.method : 'GET') || 'GET').toUpperCase();
    if (!RE.test(url) || method!=='PUT' || !armed) return fetch0(input, init);

    let txt = null;
    if (typeof init.body === 'string') txt = init.body;
    else if (typeof input!=='string' && input instanceof Request) txt = await input.clone().text();

    const patched = txt ? patchBodyText(txt, 'fetch') : null;
    if (patched){
      if (typeof init.body === 'string') init.body = patched;
      else if (typeof input!=='string' && input instanceof Request){
        const headers = new Headers(input.headers||{}); headers.set('content-type','application/json');
        input = new Request(input, { method, headers, body: patched });
      }
      const res = await fetch0(input, init);
      armed = false; unhookAll();
      return res;
    }
    return fetch0(input, init);
  };

  // XHR
  XMLHttpRequest.prototype.open = function(method, url, ...rest){
    this.__m = (method||'GET').toUpperCase();
    this.__u = url||'';
    return XHROpen0.call(this, method, url, ...rest);
  };
  XMLHttpRequest.prototype.send = function(body){
    const isPUT = (this.__m||'GET')==='PUT';
    const hit = RE.test(this.__u||'');
    if (!armed || !isPUT || !hit) return XHRSend0.call(this, body);

    const tryPatch = (text) => {
      const patched = patchBodyText(text, 'xhr');
      if (patched){
        armed = false; unhookAll();
        return XHRSend0.call(this, patched);
      }
      return XHRSend0.call(this, body);
    };

    if (typeof body === 'string') return tryPatch(body);
    if (body instanceof ArrayBuffer) {
      try { return tryPatch(new TextDecoder().decode(body)); } catch { return XHRSend0.call(this, body); }
    }
    if (body && typeof Blob !== 'undefined' && body instanceof Blob) {
      const fr = new FileReader();
      fr.onload = () => {
        const txt = typeof fr.result === 'string' ? fr.result : '';
        const patched = patchBodyText(txt, 'xhr/blob');
        if (patched){
          armed = false; unhookAll();
          XHRSend0.call(this, patched);
        } else {
          XHRSend0.call(this, body);
        }
      };
      fr.readAsText(body);
      return;
    }

    return XHRSend0.call(this, body);
  };

  if (beacon0){
    navigator.sendBeacon = (url, data) => {
      if (!armed || !RE.test(url)) return beacon0(url, data);
      if (typeof data === 'string'){
        const patched = patchBodyText(data, 'beacon');
        if (patched){ armed = false; unhookAll(); return beacon0(url, patched); }
      }
      return beacon0(url, data);
    };
  }

  console.log('💰 Money successfully added - Click the game settings button to trigger the save, then close and reopen the game..');
  console.log('💰 Para başarıyla eklendi - Oyunun ayar butonuna tıklayarak save almasını tetikle sonra oyunu kapatıp aç..');
})();
```


</details>


6. If you cannot paste, first type `allow pasting` in the console, press Enter, then try pasting again.  
7. **IMPORTANT!** : After running the code, click the game’s **Settings** button to trigger save.  
8. Close and reopen the game — the changes will now be active.  

</details>

---

## 📸 Screenshots / Ekran Görüntüleri

![Uygulama Ekran Görüntüsü](https://i.imgpost.net/2025/08/23/Cj6sg.png)
![Uygulama Ekran Görüntüsü](https://i.imgpost.net/2025/08/23/Cj6QZ.png)

---
