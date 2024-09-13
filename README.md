# Cift-Google-Translate-Cevirisi

### ADIM 1

<p>- Tampermonkey eklentisi tarayıcınıza ekleyin</p>
<p>- Yeni Betik oluştura tıklayın</p>

![image](https://github.com/DenizKod/ARK-ISTEGI-IPTAL-ETME/assets/168285638/7e1b2696-803e-447a-ae3f-f7844a44d28f)

#### Aşağıdaki kodu betik oluşturma sayfasına yapıştırıp betiği kaydedin

```
// ==UserScript==
// @name         Deniz Gölcü Çeviri
// @namespace    http://tampermonkey.net/
// @version      3.0
// @description  Alt tuşuna basılı tutunca çeviri açma
// @author
// @match        *://*/*
// @grant        GM_xmlhttpRequest
// @grant        GM_setClipboard
// ==/UserScript==

(function() {
    'use strict';

    let isPopupOpen = false;
    let popupDiv;
    let textarea1, textarea2, textarea3;
    let sourceLang = 'tr';
    let targetLang = 'en';  // bu kısımdan hedef dili değiştirebilirsiniz
    let isAltPressed = false;
    let timeoutId;

    //  (event.key === 'Alt' kısmından tuşu değiştirebilirsin
    document.addEventListener('keydown', function(event) {
        if (event.key === 'Alt' && event.location === 1 && !isAltPressed) {
            isAltPressed = true;
            timeoutId = setTimeout(function() {
                openTranslationPopup();
            }, 100); // Tuşa 100 ms basılı tutarsan çeviri açılır
        }
    });
    //  (event.key === 'Alt' kısmından tuşu değiştirebilirsin
    document.addEventListener('keyup', function(event) {
        if (event.key === 'Alt' && event.location === 1) {
            isAltPressed = false;
            clearTimeout(timeoutId);
        }
    });

    document.addEventListener('click', function(event) {
        if (isPopupOpen && popupDiv && !popupDiv.contains(event.target)) {
            closeTranslationPopup();
        }
    });

    function openTranslationPopup() {
        if (isPopupOpen) return;

        popupDiv = document.createElement('div');
        popupDiv.style.position = 'fixed';
        popupDiv.style.right = '20px';
        popupDiv.style.bottom = '100px';
        popupDiv.style.width = '500px';
        popupDiv.style.backgroundColor = '#fff';
        popupDiv.style.border = '1px solid #ddd';
        popupDiv.style.padding = '20px';
        popupDiv.style.zIndex = '9999';
        popupDiv.style.boxShadow = '0 2px 10px rgba(0, 0, 0, 0.1)';
        document.body.appendChild(popupDiv);

        textarea1 = document.createElement('textarea');
        textarea2 = document.createElement('textarea');
        textarea3 = document.createElement('textarea');

        textarea1.style.width = '100%';
        textarea1.style.minHeight = '80px';
        textarea1.style.resize = 'none';

        textarea2.style.width = '100%';
        textarea2.style.minHeight = '80px';
        textarea2.style.resize = 'none';
        textarea2.disabled = true;

        textarea3.style.width = '100%';
        textarea3.style.minHeight = '80px';
        textarea3.style.resize = 'none';
        textarea3.disabled = true;

        textarea1.style.boxSizing = 'border-box';
        textarea2.style.boxSizing = 'border-box';
        textarea3.style.boxSizing = 'border-box';

        popupDiv.appendChild(textarea1);
        popupDiv.appendChild(textarea2);
        popupDiv.appendChild(textarea3);

        textarea1.placeholder = "Türkçe metin yazın...";
        textarea2.placeholder = "İngilizce çeviri...";
        textarea3.placeholder = "Geri Türkçeye çeviri...";

        textarea1.addEventListener('input', function() {
            const originalText = textarea1.value;

            translateText(originalText, sourceLang, targetLang, function(translatedText) {
                textarea2.value = translatedText;

                translateText(translatedText, targetLang, sourceLang, function(backTranslatedText) {
                    textarea3.value = backTranslatedText;
                });
            });
        });

        const copyButton = document.createElement('button');
        copyButton.innerText = 'İngilizceyi Kopyala';
        copyButton.style.cursor = 'pointer';
        copyButton.style.marginTop = '10px';
        copyButton.style.padding = '10px 20px';
        copyButton.style.backgroundColor = '#4285F4';
        copyButton.style.color = 'white';
        copyButton.style.border = 'none';
        copyButton.style.borderRadius = '5px';
        popupDiv.appendChild(copyButton);

        copyButton.addEventListener('click', function() {
            GM_setClipboard(textarea2.value);
            copyButton.style.backgroundColor = '#34a853';
            setTimeout(() => {
                copyButton.style.backgroundColor = '#4285F4';
            }, 1000);
        });

        isPopupOpen = true;
    }

    function closeTranslationPopup() {
        if (popupDiv) {
            document.body.removeChild(popupDiv);
            popupDiv = null;
        }
        isPopupOpen = false;
    }

    function translateText(text, sourceLang, targetLang, callback) {
        const url = `https://translate.googleapis.com/translate_a/single?client=gtx&sl=${sourceLang}&tl=${targetLang}&dt=t&q=${encodeURIComponent(text)}`;
        GM_xmlhttpRequest({
            method: 'GET',
            url: url,
            onload: function(response) {
                const result = JSON.parse(response.responseText);
                const translatedText = result[0].map(t => t[0]).join('');
                callback(translatedText);
            },
            onerror: function() {
                console.error('Çeviri hatası!');
            }
        });
    }

    function checkGrammar(text, callback) {
        GM_xmlhttpRequest({
            method: 'POST',
            url: 'https://api.languagetool.org/v2/check',
            headers: {
                'Content-Type': 'application/x-www-form-urlencoded'
            },
            data: `text=${encodeURIComponent(text)}&language=tr`,
            onload: function(response) {
                try {
                    const result = JSON.parse(response.responseText);
                    const corrections = [];

                    result.matches.forEach(match => {
                        const message = match.message;
                        const suggestion = match.replacements.length > 0 ? match.replacements[0].value : '';
                        const context = match.context.text;

                        corrections.push({
                            message: message,
                            suggestion: suggestion,
                            context: context
                        });
                    });

                    callback(corrections);
                } catch (e) {
                    console.error('Dil bilgisi hatası:', e);
                }
            },
            onerror: function() {
                console.error('Dil bilgisi kontrol hatası!');
            }
        });
    }
})();
```

<p> Sayfa tam yüklendiğince Klavyenin sol tarafındaki Alt tuşuna 1 kere basın çeviri açılacaktır.

### ADIM 2

İlk defa çeviri yaptığınızda aşağıdaki sayfa açılacaktır "Her zaman izin ver" seçeneğine tıklayın. böylelikle google translate sunucusuna otomatik olarak istek gönderecektir

![image](https://github.com/user-attachments/assets/1ddf6c6d-f45c-4ccd-918a-e183dd6583d9)


<p> NOT : sayfa ilk yüklendiğinde sorun olabiliyor ama 10 saniye sonra düzeliyor.
<p> NOT 2 : kodun içerisinden çeviri açma tuşunu değiştirebilirsiniz. (chat gpt'ye sorarakta yaptırabilirsiniz)
