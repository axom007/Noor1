(function () {
    let screenReaderEnabled = false;
    let speech = window.speechSynthesis;
    let addedTabIndexes = [];

    function speak(text) {
        if (!text) return;
        speech.cancel();
        let utterance = new SpeechSynthesisUtterance(text);
        speech.speak(utterance);
    }

    function getElementDescription(el) {
        return (
            el.getAttribute('aria-label') ||
            el.alt ||
            el.innerText.trim()
        );
    }

    function onFocus(e) {
        if (!screenReaderEnabled) return;
        const desc = getElementDescription(e.target);
        if (desc) speak(desc);
    }

    function onKeyDown(e) {
        if (!screenReaderEnabled) return;
        if (e.key === 'Enter' && document.activeElement) {
            e.preventDefault();
            document.activeElement.click();
        }
    }

    function enableFocusForAll() {
        const elements = document.querySelectorAll('h1, h2, h3, h4, h5, h6, p, span, div, img, a, button');
        elements.forEach(el => {
            const hasText = getElementDescription(el);
            const isNaturallyFocusable = el.tabIndex >= 0;
            if (hasText && !isNaturallyFocusable) {
                el.tabIndex = 0;
                addedTabIndexes.push(el);
            }
        });
    }

    function removeAddedFocus() {
        addedTabIndexes.forEach(el => el.removeAttribute('tabindex'));
        addedTabIndexes = [];
    }

    function toggleScreenReader() {
        screenReaderEnabled = !screenReaderEnabled;
        speech.cancel();
        if (screenReaderEnabled) {
            enableFocusForAll();
            speak('Screen reader mode enabled');
        } else {
            removeAddedFocus();
            speak('Screen reader mode disabled');
        }
    }

    document.addEventListener('focus', onFocus, true);
    document.addEventListener('keydown', onKeyDown);

    document.addEventListener('DOMContentLoaded', function () {
        const toggleBtn = document.getElementById('sr-toggle-btn');
        if (toggleBtn) {
            toggleBtn.addEventListener('click', toggleScreenReader);
        }
    });
})();
