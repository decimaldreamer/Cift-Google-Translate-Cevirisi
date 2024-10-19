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
// @version      3.1
// @description  Alt tuşuna basılı tutunca çeviri açma
// @match        *://*/*
// @grant        GM_xmlhttpRequest
// @grant        GM_setClipboard
// ==/UserScript==

(function () {
    'use strict';

    let isPopupOpen = false;
    let popupDiv, textarea1, textarea2, textarea3;
    let isAltPressed = false;
    let timeoutId;
    const sourceLang = 'tr';
    const targetLang = 'en';

    document.addEventListener('keydown', (event) => handleKeyEvent(event, true));
    document.addEventListener('keyup', (event) => handleKeyEvent(event, false));
    document.addEventListener('click', (event) => {
        if (isPopupOpen && popupDiv && !popupDiv.contains(event.target)) {
            closeTranslationPopup();
        }
    });

    function handleKeyEvent(event, isKeyDown) {
        if (event.key === 'Alt' && event.location === 1) {
            isAltPressed = isKeyDown;
            if (isKeyDown) {
                timeoutId = setTimeout(openTranslationPopup, 100);
            } else {
                clearTimeout(timeoutId);
            }
        }
    }

    function openTranslationPopup() {
        if (isPopupOpen) return;

        popupDiv = document.createElement('div');
        Object.assign(popupDiv.style, {
            position: 'fixed',
            right: '20px',
            bottom: '100px',
            width: '500px',
            backgroundColor: '#fff',
            border: '1px solid #ddd',
            padding: '20px',
            zIndex: '9999',
            boxShadow: '0 2px 10px rgba(0, 0, 0, 0.1)',
        });
        document.body.appendChild(popupDiv);

        [textarea1, textarea2, textarea3] = ['Türkçe metin yazın...', 'Çevirilmiş Metin', 'Geri Türkçeye çeviri...'].map((placeholder, index) => {
            const textarea = document.createElement('textarea');
            Object.assign(textarea.style, {
                width: '100%',
                minHeight: '80px',
                resize: 'none',
                boxSizing: 'border-box',
            });
            textarea.placeholder = placeholder;
            if (index > 0) textarea.disabled = true;
            popupDiv.appendChild(textarea);
            return textarea;
        });

        textarea1.addEventListener('input', handleTextInput);

        const copyButton = createButton('Çeviriyi Kopyala', '#4285F4', () => {
            GM_setClipboard(textarea2.value);
            updateButtonColor(copyButton, '#34a853', '#4285F4');
        });
        popupDiv.appendChild(copyButton);

        isPopupOpen = true;
    }

    function closeTranslationPopup() {
        if (popupDiv) {
            document.body.removeChild(popupDiv);
            popupDiv = null;
            isPopupOpen = false;
        }
    }

    function handleTextInput() {
        translateText(textarea1.value, sourceLang, targetLang, (translatedText) => {
            textarea2.value = translatedText || 'Çeviri başarısız!';
            translateText(translatedText, targetLang, sourceLang, (backTranslatedText) => {
                textarea3.value = backTranslatedText || 'Çeviri başarısız!';
            });
        });
    }

    function translateText(text, source, target, callback) {
        const url = `https://translate.googleapis.com/translate_a/single?client=gtx&sl=${source}&tl=${target}&dt=t&q=${encodeURIComponent(text)}`;
        GM_xmlhttpRequest({
            method: 'GET',
            url: url,
            onload: (response) => {
                try {
                    const result = JSON.parse(response.responseText);
                    const translatedText = result[0].map(t => t[0]).join('');
                    callback(translatedText);
                } catch (e) {
                    console.error('Çeviri hatası:', e);
                    callback(null);
                }
            },
            onerror: () => {
                console.error('Çeviri hatası!');
                callback(null);
            },
        });
    }

    function createButton(text, bgColor, onClick) {
        const button = document.createElement('button');
        button.innerText = text;
        Object.assign(button.style, {
            cursor: 'pointer',
            marginTop: '10px',
            padding: '10px 20px',
            backgroundColor: bgColor,
            color: 'white',
            border: 'none',
            borderRadius: '5px',
        });
        button.addEventListener('click', onClick);
        return button;
    }

    function updateButtonColor(button, activeColor, defaultColor) {
        button.style.backgroundColor = activeColor;
        setTimeout(() => {
            button.style.backgroundColor = defaultColor;
        }, 1000);
    }
})();

```
<p> Sayfa tam yüklendiğince Klavyenin sol tarafındaki Alt tuşuna 1 kere basın çeviri açılacaktır.

### ADIM 2

İlk defa çeviri yaptığınızda aşağıdaki sayfa açılacaktır "Her zaman izin ver" seçeneğine tıklayın. böylelikle google translate sunucusuna otomatik olarak istek gönderecektir

![image](https://github.com/user-attachments/assets/1ddf6c6d-f45c-4ccd-918a-e183dd6583d9)


<p> NOT : sayfa ilk yüklendiğinde sorun olabiliyor ama 10 saniye sonra düzeliyor.
<p> NOT 2 : kodun içerisinden çeviri açma tuşunu değiştirebilirsiniz. (chat gpt'ye sorarakta yaptırabilirsiniz)
