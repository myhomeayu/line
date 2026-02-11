// ==UserScript==
// @name         LINE Login Auto Click
// @namespace    user scripts
// @version      1.0.1
// @description  Auto-click LINE login button once on login page
// @match        https://access.line.me/oauth2/v2.1/login*
// @match        https://access.line.me/oauth2/v2.1/authorize/consent*
// @run-at       document-idle
// @grant        none
// ==/UserScript==
// @updateURL    https://github.com/myhomeayu/line/raw/main/line.user.js
// @downloadURL  https://github.com/myhomeayu/line/raw/main/line.user.js


(() => {
  'use strict';

  const DEBUG = true;
  const LOG_PREFIX = '[TM-LINE-AUTOLOGIN]';
  const POLL_INTERVAL_MS = 100;
  const MAX_TRIES = 20; // 2s
  const LOGIN_GUARD_KEY = 'tm_line_login_clicked';
  const CONSENT_GUARD_KEY = 'tm_line_consent_clicked';

  function log(...args) {
    if (DEBUG) console.log(LOG_PREFIX, ...args);
  }

  function normalizeText(s) {
    return (s || '').replace(/\s+/g, ' ').trim();
  }

  function getPageMode() {
    if (location.hostname !== 'access.line.me') return '';
    if (location.pathname === '/oauth2/v2.1/login') return 'login';
    if (location.pathname.startsWith('/oauth2/v2.1/authorize/consent')) return 'consent';
    return '';
  }

  function isVisible(el) {
    if (!el) return false;
    const style = getComputedStyle(el);
    if (style.display === 'none' || style.visibility === 'hidden' || style.opacity === '0') return false;
    return el.offsetWidth > 0 || el.offsetHeight > 0 || el.getClientRects().length > 0;
  }

  function isButtonEnabled(btn) {
    if (!btn || !isVisible(btn)) return false;
    if (btn.disabled === true) return false;
    if ((btn.getAttribute('aria-disabled') || '').toLowerCase() === 'true') return false;
    return true;
  }

  function isExactSubmitButton(btn, label) {
    if (!btn) return false;
    const text = normalizeText(btn.textContent || btn.innerText || '');
    const type = (btn.getAttribute('type') || '').toLowerCase();
    return text === label && type === 'submit';
  }

  function findLoginButton() {
    const a = Array.from(document.querySelectorAll('button.MdBtn01')).find((el) => isExactSubmitButton(el, 'ログイン'));
    if (a) return { el: a, foundBy: 'A:button.MdBtn01' };

    const b = Array.from(document.querySelectorAll('button.c-button')).find((el) => isExactSubmitButton(el, 'ログイン'));
    if (b) return { el: b, foundBy: 'B:button.c-button' };

    const fallback = Array.from(document.querySelectorAll('button[type="submit"]')).find(
      (el) => normalizeText(el.textContent || el.innerText || '') === 'ログイン'
    );
    if (fallback) return { el: fallback, foundBy: 'fallback:button[type=submit]+text' };

    return { el: null, foundBy: '' };
  }

  function findConsentButton() {
    const primary = Array.from(document.querySelectorAll('button.ldsm-box-button'))
      .find((el) => isExactSubmitButton(el, '許可する'));
    if (primary) return { el: primary, foundBy: 'consent:button.ldsm-box-button' };

    const fallback = Array.from(document.querySelectorAll('button[type="submit"]'))
      .find((el) => normalizeText(el.textContent || el.innerText || '') === '許可する');
    if (fallback) return { el: fallback, foundBy: 'consent:fallback:button[type=submit]+text' };

    return { el: null, foundBy: '' };
  }

  function run() {
    const mode = getPageMode();
    if (!mode) {
      log('skip: non-target page');
      return;
    }
    const guardKey = mode === 'login' ? LOGIN_GUARD_KEY : CONSENT_GUARD_KEY;
    log('start', { href: location.href, mode, guardKey });

    if (sessionStorage.getItem(guardKey)) {
      log('skip: guard hit');
      return;
    }

    let tries = 0;
    const timer = setInterval(() => {
      tries += 1;

      const { el, foundBy } = mode === 'login' ? findLoginButton() : findConsentButton();
      if (!el) {
        if (tries >= MAX_TRIES) {
          clearInterval(timer);
          log('not found');
        }
        return;
      }

      if (!isButtonEnabled(el)) {
        clearInterval(timer);
        log('skip: button found but disabled/not visible', { foundBy });
        return;
      }

      sessionStorage.setItem(guardKey, '1');

      try {
        el.click();
        log('clicked', { foundBy });
      } catch (e) {
        log('click failed', { foundBy, error: String((e && e.message) || e) });
      } finally {
        clearInterval(timer);
      }
    }, POLL_INTERVAL_MS);
  }

  run();
})();
