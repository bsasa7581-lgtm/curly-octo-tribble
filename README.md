<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
<meta name="mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
<title>Geometry Dash — Neon Campaign</title>
<script src="https://unpkg.com/peerjs@1.5.4/dist/peerjs.min.js"></script>
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }
  html, body {
    width: 100%; height: 100%;
    height: 100dvh;
    background: #0d0d1a;
    overflow: hidden;
    font-family: Arial, sans-serif;
    touch-action: none;
    user-select: none;
    -webkit-user-select: none;
    -webkit-touch-callout: none;
    position: fixed;
    inset: 0;
  }
  #wrap {
    position: fixed; inset: 0;
    display: flex; justify-content: center; align-items: center;
    width: 100%; height: 100%;
    background: #0d0d1a;
  }
  canvas {
    display: block;
    background: #1e1e32;
    box-shadow: 0 0 40px rgba(0, 255, 200, 0.2);
    border-radius: 0;
    width: 100vw !important;
    height: 100vh !important;
    height: 100dvh !important;
    max-width: 100vw;
    max-height: 100dvh;
    cursor: pointer;
    object-fit: contain;
  }
  #hint {
    position: absolute; bottom: 8px; width: 100%;
    text-align: center;
    color: rgba(255,255,255,0.4);
    font-size: 13px;
    pointer-events: none;
  }
  #netStatus {
    position: absolute; top: 8px; right: 12px;
    font-size: 12px;
    color: rgba(255,255,255,0.5);
    pointer-events: none;
    z-index: 5;
  }
  #netStatus.online { color: #00ff88; }
  #netStatus.offline { color: #ff5050; }

  .uiBtn {
    position: absolute;
    z-index: 5;
    background: rgba(0,0,0,0.55);
    border: 1px solid #00ffc8;
    color: #00ffc8;
    border-radius: 8px;
    padding: 6px 12px;
    font-size: 13px;
    cursor: pointer;
    font-weight: bold;
  }
  .uiBtn:hover { background: rgba(0,255,200,0.15); }
  .uiBtn.muted { border-color: #888; color: #888; }
  #musicBtn { top: 8px; left: 12px; }
  #nextTrackBtn { top: 8px; left: 110px; }
  #pauseBtn { top: 8px; left: 175px; display: none; }
  #menuExitBtn { top: 8px; left: 290px; display: none; border-color: #ff8830; color: #ff8830; }
  #menuExitBtn:hover { background: rgba(255,136,48,0.15); }
  #scrollUpBtn, #scrollDownBtn {
    display: none; position: absolute; right: 16px; z-index: 6;
    width: 40px; height: 40px; border-radius: 10px;
    background: rgba(0,0,0,0.65); border: 1px solid #00ffc8; color: #00ffc8;
    font-size: 20px; font-weight: bold; cursor: pointer;
  }
  #scrollUpBtn { top: 120px; }
  #scrollDownBtn { bottom: 50px; }

  #trackLabel {
    position: absolute; top: 40px; left: 12px;
    z-index: 5;
    font-size: 11px;
    color: rgba(255,255,255,0.45);
    pointer-events: none;
    max-width: 280px;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  }
  #friendsOverlay, #chatOverlay {
    position: absolute; inset: 0;
    display: none;
    justify-content: center;
    align-items: center;
    background: rgba(0,0,0,0.82);
    z-index: 25;
  }
  #friendsOverlay.active, #chatOverlay.active { display: flex; }
  .panelBox {
    background: #1a1a32;
    border: 2px solid #00ffc8;
    border-radius: 14px;
    padding: 18px;
    width: 420px;
    max-width: 94vw;
    max-height: 88vh;
    overflow: auto;
    color: #fff;
    box-shadow: 0 0 30px rgba(0,255,200,0.25);
  }
  .panelBox h2 { color: #00ffc8; margin: 0 0 10px; font-size: 20px; text-align: center; }
  .panelBox .hint { font-size: 12px; color: #aaa; text-align: center; margin-bottom: 12px; }
  .panelBox input, .panelBox textarea {
    width: 100%; padding: 10px; margin-bottom: 8px;
    background: #0d0d1a; border: 2px solid #3c3c5a; border-radius: 8px;
    color: #fff; font-size: 14px; outline: none;
  }
  .panelBox input:focus, .panelBox textarea:focus { border-color: #00ffc8; }
  .panelBox .rowBtns { display: flex; gap: 8px; margin-top: 8px; }
  .panelBox .rowBtns button, .panelBox > button {
    flex: 1; padding: 10px; border: none; border-radius: 8px;
    background: #00ffc8; color: #001a14; font-weight: bold; cursor: pointer; font-size: 14px;
  }
  .panelBox .rowBtns button.secondary, .panelBox > button.secondary {
    background: #3c3c5a; color: #fff;
  }
  .panelBox .rowBtns button:hover, .panelBox > button:hover { opacity: 0.9; }
  .friendRow {
    display: flex; align-items: center; gap: 8px;
    background: #12122a; border-radius: 8px; padding: 8px 10px; margin-bottom: 6px;
    font-size: 13px;
  }
  .friendRow .fname { flex: 1; color: #00ffc8; font-weight: bold; overflow: hidden; text-overflow: ellipsis; }
  .friendRow .fid { color: #888; font-size: 11px; font-family: monospace; }
  .friendRow button {
    background: #4ab8ff; color: #001; border: none; border-radius: 6px;
    padding: 5px 8px; font-size: 11px; font-weight: bold; cursor: pointer; white-space: nowrap;
  }
  .friendRow button.del { background: #ff5050; color: #fff; }
  #chatLog {
    height: 220px; overflow-y: auto; background: #0d0d1a; border-radius: 8px;
    padding: 10px; margin-bottom: 8px; font-size: 13px; line-height: 1.4;
  }
  #chatLog .msg { margin-bottom: 6px; }
  #chatLog .msg .who { color: #00ffc8; font-weight: bold; }
  #chatLog .msg .who.me { color: #ffdd44; }
  #chatLog .msg .who.sys { color: #888; }
  #chatBadge {
    position: absolute; top: 6px; right: 6px;
    background: #ff5050; color: #fff; border-radius: 10px;
    font-size: 11px; padding: 1px 6px; display: none; font-weight: bold;
  }
  #onlineOverlay {
    position: absolute; inset: 0;
    display: none;
    justify-content: center;
    align-items: center;
    background: rgba(0,0,0,0.85);
    z-index: 26;
  }
  #onlineOverlay.active { display: flex; }
  .onlineRow {
    display: flex; align-items: center; gap: 10px;
    background: #12122a; border-radius: 8px; padding: 10px 12px;
    margin-bottom: 6px; font-size: 14px;
  }
  .onlineRow .dot {
    width: 10px; height: 10px; border-radius: 50%;
    background: #00ff88; box-shadow: 0 0 8px #00ff88;
    flex-shrink: 0;
  }
  .onlineRow .dot.off { background: #555; box-shadow: none; }
  .onlineRow .oname { flex: 1; color: #00ffc8; font-weight: bold; }
  .onlineRow .ometa { color: #888; font-size: 11px; }

  #authOverlay {
    position: absolute; inset: 0;
    display: none;
    justify-content: center;
    align-items: center;
    background: rgba(0,0,0,0.85);
    z-index: 10;
    overflow: auto;
  }
  #authOverlay.active { display: flex; }
  .authBox {
    background: #1e1e3c;
    border: 2px solid #00ffc8;
    border-radius: 16px;
    padding: 30px;
    width: 360px;
    box-shadow: 0 0 40px rgba(0,255,200,0.4);
    color: #fff;
    text-align: center;
    max-height: 95vh;
    overflow-y: auto;
  }
  .authBox h2 { color: #00ffc8; margin-bottom: 20px; font-size: 24px; }
  .authBox input {
    width: 100%;
    padding: 12px;
    margin-bottom: 12px;
    background: #0d0d1a;
    border: 2px solid #3c3c5a;
    border-radius: 8px;
    color: #fff;
    font-size: 16px;
    outline: none;
  }
  .authBox input:focus { border-color: #00ffc8; }
  .authBox button {
    width: 100%;
    padding: 12px;
    margin-top: 8px;
    background: #00ffc8;
    color: #001a14;
    border: none;
    border-radius: 8px;
    font-size: 16px;
    font-weight: bold;
    cursor: pointer;
    transition: 0.15s;
  }
  .authBox button:hover { background: #00e0b0; transform: scale(1.02); }
  .authBox button.secondary {
    background: #3c3c5a;
    color: #fff;
    margin-top: 12px;
  }
  .authBox button.secondary:hover { background: #505078; }
  .authBox .err {
    color: #ff5050;
    font-size: 14px;
    margin-bottom: 12px;
    min-height: 18px;
  }
  .authBox .tab {
    display: flex;
    margin-bottom: 20px;
    background: #0d0d1a;
    border-radius: 8px;
    overflow: hidden;
  }
  .authBox .tab button {
    flex: 1;
    background: transparent;
    color: #aaa;
    border-radius: 0;
    margin: 0;
    padding: 10px;
  }
  .authBox .tab button.active { background: #00ffc8; color: #001a14; }

  #banOverlay {
    position: absolute; inset: 0;
    display: none;
    justify-content: center;
    align-items: center;
    background: rgba(0,0,0,0.9);
    z-index: 20;
    overflow: auto;
  }
  #banOverlay.active { display: flex; }
  .banBox {
    background: #2a0a10;
    border: 2px solid #ff5050;
    border-radius: 16px;
    padding: 24px;
    width: 520px;
    max-width: 95vw;
    max-height: 90vh;
    overflow-y: auto;
    box-shadow: 0 0 40px rgba(255,80,80,0.5);
    color: #fff;
  }
  .banBox h2 { color: #ff5050; margin-bottom: 8px; text-align: center; font-size: 22px; }
  .banBox .hintText {
    font-size: 12px;
    color: #aaa;
    text-align: center;
    margin-bottom: 12px;
  }
  .banBox .userRow {
    display: flex;
    flex-direction: column;
    padding: 10px 12px;
    background: #1e1e3c;
    border-radius: 8px;
    margin-bottom: 8px;
    font-size: 13px;
    gap: 6px;
  }
  .banBox .userRow.banned { border-left: 3px solid #ff5050; }
  .banBox .userRow .top {
    display: flex;
    justify-content: space-between;
    align-items: center;
    gap: 8px;
  }
  .banBox .userRow .name {
    font-weight: bold;
    color: #00ffc8;
    flex: 1;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }
  .banBox .userRow.banned .name { color: #ff5050; text-decoration: line-through; }
  .banBox .userRow .meta {
    font-size: 11px;
    color: #aaa;
    display: flex;
    flex-wrap: wrap;
    gap: 8px;
  }
  .banBox .userRow .meta span { background: #0d0d1a; padding: 2px 6px; border-radius: 4px; }
  .banBox .userRow .online { color: #00ff88; }
  .banBox .userRow .offline { color: #888; }
  .banBox .userRow .ip { color: #4ab8ff; font-family: monospace; }
  .banBox .userRow .actions {
    display: flex;
    gap: 6px;
    margin-top: 4px;
  }
  .banBox .userRow button {
    background: #ff5050;
    color: #fff;
    border: none;
    padding: 5px 10px;
    border-radius: 6px;
    cursor: pointer;
    font-size: 11px;
    font-weight: bold;
    white-space: nowrap;
  }
  .banBox .userRow.banned button.banBtn { background: #00ff88; color: #001a14; }
  .banBox .userRow button:hover { opacity: 0.85; }
  .banBox .userRow button.ipBan { background: #ff8830; }
  .banBox .empty {
    text-align: center;
    color: #888;
    padding: 20px;
    font-size: 14px;
  }
  .banBox .closeBtn {
    width: 100%;
    padding: 12px;
    margin-top: 16px;
    background: #3c3c5a;
    color: #fff;
    border: none;
    border-radius: 8px;
    cursor: pointer;
    font-size: 16px;
    font-weight: bold;
  }
  .banBox .closeBtn:hover { background: #505078; }
  .banBox .stats {
    font-size: 12px;
    color: #aaa;
    text-align: center;
    margin-bottom: 10px;
  }

  #disclaimerOverlay {
    position: absolute; inset: 0;
    display: none;
    justify-content: center;
    align-items: center;
    background: radial-gradient(ellipse at center, #1a1030 0%, #050510 70%);
    z-index: 40;
    padding: 16px;
  }
  #disclaimerOverlay.active { display: flex; }
  .discBox {
    width: 520px;
    max-width: 96vw;
    max-height: 92vh;
    overflow-y: auto;
    background: linear-gradient(160deg, #1e1e3c 0%, #12122a 100%);
    border: 2px solid #00ffc8;
    border-radius: 18px;
    padding: 28px 24px;
    color: #e8e8f0;
    box-shadow: 0 0 50px rgba(0,255,200,0.25), 0 0 100px rgba(255,80,80,0.08);
    text-align: center;
  }
  .discBox .warnIcon {
    font-size: 48px;
    line-height: 1;
    margin-bottom: 8px;
    filter: drop-shadow(0 0 12px rgba(255,80,80,0.6));
  }
  .discBox h1 {
    color: #ff6060;
    font-size: 22px;
    margin: 0 0 14px;
    letter-spacing: 0.5px;
    text-shadow: 0 0 20px rgba(255,80,80,0.4);
  }
  .discBox .epi {
    background: rgba(255,80,80,0.12);
    border: 1px solid rgba(255,80,80,0.45);
    border-radius: 12px;
    padding: 14px 16px;
    margin-bottom: 16px;
    font-size: 14px;
    line-height: 1.5;
    text-align: left;
    color: #ffd0d0;
  }
  .discBox .epi b { color: #ff8080; }
  .discBox .tosMini {
    background: rgba(0,255,200,0.06);
    border: 1px solid rgba(0,255,200,0.25);
    border-radius: 12px;
    padding: 12px;
    margin-bottom: 14px;
    font-size: 12px;
    line-height: 1.45;
    text-align: left;
    color: #b8c8c8;
    max-height: 140px;
    overflow-y: auto;
  }
  .discBox label.discCheck {
    display: flex;
    align-items: flex-start;
    gap: 10px;
    text-align: left;
    font-size: 13px;
    margin: 12px 0 16px;
    cursor: pointer;
    color: #ccc;
  }
  .discBox label.discCheck input {
    margin-top: 3px;
    width: 18px;
    height: 18px;
    accent-color: #00ffc8;
    flex-shrink: 0;
  }
  .discBox .discBtns {
    display: flex;
    flex-direction: column;
    gap: 10px;
  }
  .discBox .discBtns button {
    padding: 14px 16px;
    border: none;
    border-radius: 10px;
    font-size: 15px;
    font-weight: bold;
    cursor: pointer;
  }
  .discBox #discContinue {
    background: linear-gradient(90deg, #00ffc8, #00c8a0);
    color: #002218;
    opacity: 0.4;
    pointer-events: none;
    transition: opacity 0.2s;
  }
  .discBox #discContinue.ready {
    opacity: 1;
    pointer-events: auto;
    box-shadow: 0 0 24px rgba(0,255,200,0.35);
  }
  .discBox #discReadTos {
    background: transparent;
    border: 1px solid #00ffc8;
    color: #00ffc8;
  }
  .discBox #discReadTos:hover {
    background: rgba(0,255,200,0.1);
  }

  #tosOverlay, #editorOverlay {
    position: fixed;
    inset: 0;
    display: none;
    justify-content: center;
    align-items: center;
    background: rgba(0,0,0,0.88);
    z-index: 50;
    overflow: auto;
    padding: 12px;
  }
  #tosOverlay.active, #editorOverlay.active {
    display: flex !important;
  }
  .tosBox {
    background: linear-gradient(160deg, #1e1e3c, #12122a);
    border: 2px solid #00ffc8;
    border-radius: 16px;
    padding: 22px;
    width: 480px;
    max-width: 96vw;
    max-height: 90vh;
    overflow-y: auto;
    color: #ddd;
    font-size: 13px;
    line-height: 1.45;
    box-shadow: 0 0 40px rgba(0,255,200,0.3);
  }
  .tosBox h2 { color: #00ffc8; text-align: center; margin: 0 0 12px; font-size: 20px; }
  .tosBox .tosBody {
    max-height: 50vh;
    overflow-y: auto;
    background: #0d0d1a;
    padding: 12px;
    border-radius: 8px;
    margin-bottom: 12px;
    text-align: left;
  }
  .tosBox .rowBtns, .tosBox label {
    display: flex;
    gap: 8px;
    align-items: center;
    margin-bottom: 10px;
  }
  .tosBox .rowBtns button, .tosBox > button {
    flex: 1;
    padding: 12px;
    border: none;
    border-radius: 8px;
    background: #00ffc8;
    color: #001a14;
    font-weight: bold;
    cursor: pointer;
    font-size: 14px;
  }
  .tosBox .rowBtns button.secondary {
    background: #3c3c5a;
    color: #fff;
  }
  #disclaimerOverlay { z-index: 40; }

  .editorGrid {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 8px;
    margin-bottom: 10px;
  }
  .editorGrid button {
    padding: 10px;
    border: none;
    border-radius: 8px;
    background: #3c3c5a;
    color: #fff;
    font-weight: bold;
    cursor: pointer;
    font-size: 13px;
  }
  .editorGrid button:hover { background: #505078; }
</style>
</head>
<body>

<div id="disclaimerOverlay" class="active">
  <div class="discBox">
    <div class="warnIcon">⚠️</div>
    <h1>Предупреждение о здоровье</h1>
    <div class="epi">
      <b>Эпилепсия и светочувствительность.</b><br><br>
      Игра содержит <b>яркие вспышки, резкую смену цветов, тряску экрана и быстрое движение</b>.
      Если у вас эпилепсия, светочувствительность или похожие состояния —
      <b>рекомендуем не играть</b>.<br><br>
      Если вы всё же продолжите, <b>вся ответственность лежит на вас</b>.
      Разработчик и администратор не несут ответственности за возможное ухудшение самочувствия.
    </div>
    <div class="tosMini" id="discTosPreview">
      Кратко: данные хранятся в браузере; запрещены читы и обход банов; онлайн — P2P с другом;
      сторонний контент принадлежит правообладателям; игра «как есть».
      Полный текст — по кнопке ниже.
    </div>
    <label class="discCheck">
      <input type="checkbox" id="discCheck">
      <span>Я прочитал(а) предупреждение и пользовательское соглашение. Понимаю риск и принимаю ответственность на себя.</span>
    </label>
    <div class="discBtns">
      <button type="button" id="discContinue">Продолжить</button>
      <button type="button" id="discReadTos">📜 Читать пользовательское соглашение</button>
    </div>
  </div>
</div>

<div id="wrap">
  <canvas id="game"></canvas>
  <div id="hint">Тап / Пробел — прыжок · 🏠 Меню · Esc — пауза · M — музыка · N — трек</div>
  <div id="netStatus">● ...</div>
  <button id="musicBtn" class="uiBtn" type="button" title="Вкл/выкл музыку">♫ Музыка</button>
  <button id="nextTrackBtn" class="uiBtn" type="button" title="Следующий трек">⏭</button>
  <button id="pauseBtn" class="uiBtn" type="button" title="Пауза">⏸ Пауза</button>
  <button id="menuExitBtn" class="uiBtn" type="button" title="В главное меню">🏠 Меню</button>
  <button id="scrollUpBtn" type="button" title="Вверх">▲</button>
  <button id="scrollDownBtn" type="button" title="Вниз">▼</button>
  <div id="trackLabel"></div>
</div>
<audio id="bgMusic" loop preload="auto"></audio>
<!-- Положи рядом: track-standoff.mp3, track-roskomnadzor.mp3, track-napoleon.mp3 -->

<div id="authOverlay">
  <div class="authBox">
    <h2 id="authTitle">Регистрация</h2>
    <div class="tab">
      <button id="tabLogin">Вход</button>
      <button id="tabRegister" class="active">Регистрация</button>
    </div>
    <div class="err" id="authErr"></div>
    <input type="text" id="authUser" placeholder="Логин (мин. 3 символа)" maxlength="20" autocomplete="off">
    <input type="password" id="authPass" placeholder="Пароль (мин. 3 символа)" maxlength="30">
    <button id="authSubmit">Зарегистрироваться</button>
    <button id="authGuest" class="secondary">Играть как гость</button>
  </div>
</div>

<div id="banOverlay">
  <div class="banBox">
    <h2>🔨 Управление игроками</h2>
    <div class="hintText">IP, статус, бан / разбан · данные только в этом браузере</div>
    <div class="stats" id="banStats"></div>
    <div id="banList"></div>
    <button class="closeBtn" id="banClose">Закрыть</button>
  </div>
</div>

<div id="friendsOverlay">
  <div class="panelBox">
    <h2>👥 Друзья</h2>
    <div class="hint">Сохрани ID друга после онлайн-игры. Потом можно быстро подключиться.</div>
    <input type="text" id="friendName" placeholder="Имя друга" maxlength="20">
    <input type="text" id="friendPeerId" placeholder="Его Peer ID (из онлайн-комнаты)" maxlength="80">
    <div class="rowBtns">
      <button type="button" id="friendAdd">➕ Добавить</button>
      <button type="button" id="friendAddCurrent" class="secondary">Добавить текущего</button>
    </div>
    <div id="friendsList" style="margin-top:12px;"></div>
    <div class="rowBtns">
      <button type="button" id="friendOpenChat" class="secondary">💬 Чат</button>
      <button type="button" id="friendsClose" class="secondary">Закрыть</button>
    </div>
  </div>
</div>

<div id="chatOverlay">
  <div class="panelBox">
    <h2>💬 Чат <span id="chatBadge">0</span></h2>
    <div class="hint" id="chatHint">Чат работает, когда вы подключены к другу онлайн</div>
    <div id="chatLog"></div>
    <input type="text" id="chatInput" placeholder="Напиши сообщение..." maxlength="200">
    <div class="rowBtns">
      <button type="button" id="chatSend">Отправить</button>
      <button type="button" id="chatClose" class="secondary">Закрыть</button>
    </div>
  </div>
</div>

<div id="onlineOverlay">
  <div class="panelBox">
    <h2>🟢 Кто в сети</h2>
    <div class="hint">Игроки с этого устройства + друг по онлайн-комнате. Обновляется автоматически.</div>
    <div id="onlineList"></div>
    <div cl
    <script>
(function () {
  "use strict";

  const canvas = document.getElementById('game');
  const ctx = canvas.getContext('2d');
  const netStatusEl = document.getElementById('netStatus');
  const musicBtn = document.getElementById('musicBtn');
  const nextTrackBtn = document.getElementById('nextTrackBtn');
  const pauseBtn = document.getElementById('pauseBtn');
  const menuExitBtn = document.getElementById('menuExitBtn');
  const scrollUpBtn = document.getElementById('scrollUpBtn');
  const scrollDownBtn = document.getElementById('scrollDownBtn');
  const trackLabel = document.getElementById('trackLabel');
  const bgMusic = document.getElementById('bgMusic');

  const VW = 960, VH = 540;
  const GROUND_Y = VH - 80;

  // === Музыка (плейлист) ===
  // Положи mp3-файлы рядом с HTML или замени пути на свои
  const TRACKS = [
    { src: "track-standoff.mp3",     title: "Standoff 2 — New Year 2023" },
    { src: "track-roskomnadzor.mp3", title: "Роскомнадзор" },
    { src: "track-napoleon.mp3",     title: "Napoleon" }
  ];
  let trackIndex = 0;
  let musicEnabled = true;

  function loadTrack(i) {
    trackIndex = (i + TRACKS.length) % TRACKS.length;
    const t = TRACKS[trackIndex];
    bgMusic.src = t.src;
    trackLabel.textContent = "♪ " + t.title;
    if (musicEnabled) {
      bgMusic.play().catch(() => {});
    }
  }
  function nextTrack() { loadTrack(trackIndex + 1); }

  musicBtn.addEventListener('click', () => {
    musicEnabled = !musicEnabled;
    musicBtn.classList.toggle('muted', !musicEnabled);
    if (musicEnabled) bgMusic.play().catch(() => {});
    else bgMusic.pause();
  });
  nextTrackBtn.addEventListener('click', nextTrack);

  // === Игровое состояние ===
  const state = {
    running: false,
    paused: false,
    menu: true,
    t: 0,
    camX: 0,
    speed: 5,
    player: {
      x: 160,
      y: GROUND_Y,
      vy: 0,
      size: 34,
      onGround: true,
      rotation: 0
    },
    gravity: 0.85,
    jumpForce: -14,
    level: {
      length: 6000,
      spikes: [],
      blocks: [],
      coins: [],
      finishX: 6000
    },
    score: 0,
    coins: 0,
    attempts: 0,
    shake: 0,
    flash: 0
  };

  // === Генерация уровня (стандартная кампания) ===
  function buildDefaultLevel() {
    const L = state.level;
    L.spikes = [];
    L.blocks = [];
    L.coins = [];
    L.length = 6000;
    L.finishX = 6000;

    let x = 700;
    while (x < L.length - 400) {
      const r = Math.random();
      if (r < 0.45) {
        L.spikes.push({ x: x, w: 36, h: 36 });
        x += 200 + Math.random() * 260;
      } else if (r < 0.7) {
        L.blocks.push({ x: x, y: GROUND_Y - 36, w: 60, h: 36 });
        x += 240 + Math.random() * 260;
      } else if (r < 0.85) {
        L.coins.push({ x: x, y: GROUND_Y - 120, r: 14, taken: false });
        x += 180 + Math.random() * 200;
      } else {
        L.spikes.push({ x: x, w: 36, h: 36 });
        L.spikes.push({ x: x + 60, w: 36, h: 36 });
        x += 320 + Math.random() * 220;
      }
    }
    // финальные монеты
    for (let i = 0; i < 5; i++) {
      L.coins.push({ x: L.length - 900 + i * 60, y: GROUND_Y - 140 - Math.sin(i) * 20, r: 14, taken: false });
    }
  }
  buildDefaultLevel();

  // === Ресайз canvas ===
  function resizeCanvas() {
    const dpr = Math.min(window.devicePixelRatio || 1, 2);
    canvas.width = VW * dpr;
    canvas.height = VH * dpr;
    ctx.setTransform(dpr, 0, 0, dpr, 0, 0);
    ctx.imageSmoothingEnabled = false;
  }
  window.addEventListener('resize', resizeCanvas);
  resizeCanvas();

  // === Прыжок ===
  function jump() {
    if (!state.running || state.paused) return;
    const p = state.player;
    if (p.onGround) {
      p.vy = state.jumpForce;
      p.onGround = false;
    }
  }

  // === Сброс попытки ===
  function resetAttempt() {
    const p = state.player;
    p.x = 160;
    p.y = GROUND_Y;
    p.vy = 0;
    p.onGround = true;
    p.rotation = 0;
    state.camX = 0;
    state.score = 0;
    state.t = 0;
    state.shake = 0;
    state.flash = 0;
    state.level.coins.forEach(c => c.taken = false);
    state.attempts++;
  }

  // === Старт игры ===
  function startGame(level) {
    if (level) {
      state.level = JSON.parse(JSON.stringify(level));
      state.level.coins.forEach(c => c.taken = false);
    }
    resetAttempt();
    state.running = true;
    state.paused = false;
    state.menu = false;
    pauseBtn.style.display = 'block';
    menuExitBtn.style.display = 'block';
  }

  // === Обновление физики ===
  function update(dt) {
    if (!state.running || state.paused) return;
    const p = state.player;
    const L = state.level;

    state.t += dt;
    state.speed = 5 + Math.min(state.t / 6000, 4);

    // движение камеры
    p.x += state.speed;
    state.camX = p.x - 160;

    // гравитация
    p.vy += state.gravity;
    p.y += p.vy;

    // земля
    if (p.y >= GROUND_Y) {
      p.y = GROUND_Y;
      p.vy = 0;
      p.onGround = true;
    }

    // вращение
    if (!p.onGround) {
      p.rotation += 0.18;
    } else {
      p.rotation = Math.round(p.rotation / (Math.PI / 2)) * (Math.PI / 2);
    }

    // столкновения: блоки (стоим сверху)
    let onBlock = false;
    for (const b of L.blocks) {
      if (p.x + p.size > b.x && p.x - p.size < b.x + b.w) {
        const prevBottom = p.y - p.vy;
        if (prevBottom <= b.y + 4 && p.y >= b.y) {
          p.y = b.y;
          p.vy = 0;
          p.onGround = true;
          onBlock = true;
        }
      }
    }
    if (onBlock) {
      // уже обработано
    } else if (p.y < GROUND_Y) {
      p.onGround = false;
    }

    // столкновения: шипы
    for (const s of L.spikes) {
      const sx = s.x + 6, sw = s.w - 12;
      const sy = GROUND_Y - s.h + 6, sh = s.h - 6;
      if (p.x + p.size / 2 > sx && p.x - p.size / 2 < sx + sw &&
          p.y + p.size / 2 > sy && p.y - p.size / 2 < sy + sh) {
        // смерть
        state.shake = 18;
        state.flash = 1;
        state.running = false;
        setTimeout(() => { resetAttempt(); state.running = true; }, 500);
        return;
      }
    }

    // монеты
    for (const c of L.coins) {
      if (c.taken) continue;
      const dx = p.x - c.x, dy = p.y - p.size / 2 - c.y;
      if (dx * dx + dy * dy < (c.r + 16) * (c.r + 16)) {
        c.taken = true;
        state.coins++;
        state.flash = 0.3;
      }
    }

    // финиш
    if (p.x >= L.finishX) {
      state.running = false;
      state.flash = 1;
      setTimeout(() => {
        alert("Уровень пройден! Монет: " + state.coins + " / " + L.coins.length);
        state.menu = true;
        pauseBtn.style.display = 'none';
        menuExitBtn.style.display = 'none';
        state.running = false;
      }, 300);
    }

    // тряска
    if (state.shake > 0) state.shake *= 0.9;
    if (state.flash > 0) state.flash -= dt * 0.005;
    if (state.flash < 0) state.flash = 0;
  }

  // === Отрисовка ===
  function draw() {
    const L = state.level;
    const shakeX = state.shake > 0.5 ? (Math.random() - 0.5) * state.shake : 0;
    const shakeY = state.shake > 0.5 ? (Math.random() - 0.5) * state.shake : 0;

    ctx.save();
    ctx.translate(shakeX, shakeY);

    // фон
    const grad = ctx.createLinearGradient(0, 0, 0, VH);
    grad.addColorStop(0, '#1a1a3a');
    grad.addColorStop(1, '#0d0d1a');
    ctx.fillStyle = grad;
    ctx.fillRect(-20, -20, VW + 40, VH + 40);

    // сетка (параллакс)
    ctx.strokeStyle = 'rgba(0,255,200,0.08)';
    ctx.lineWidth = 1;
    const gridOff = (state.camX * 0.5) % 60;
    for (let gx = -gridOff; gx < VW; gx += 60) {
      ctx.beginPath();
      ctx.moveTo(gx, 0);
      ctx.lineTo(gx, VH);
      ctx.stroke();
    }
    for (let gy = 0; gy < VH; gy += 60) {
      ctx.beginPath();
      ctx.moveTo(0, gy);
      ctx.lineTo(VW, gy);
      ctx.stroke();
    }

    // звёзды
    ctx.fillStyle = 'rgba(255,255,255,0.6)';
    for (let i = 0; i < 60; i++) {
      const sx = ((i * 137 - state.camX * 0.2) % VW + VW) % VW;
      const sy = (i * 89) % (VH - 100);
      ctx.fillRect(sx, sy, 2, 2);
    }

    // земля
    ctx.fillStyle = '#151528';
    ctx.fillRect(0, GROUND_Y, VW, VH - GROUND_Y);
    ctx.strokeStyle = '#00ffc8';
    ctx.lineWidth = 3;
    ctx.shadowColor = '#00ffc8';
    ctx.shadowBlur = 14;
    ctx.beginPath();
    ctx.moveTo(0, GROUND_Y);
    ctx.lineTo(VW, GROUND_Y);
    ctx.stroke();
    ctx.shadowBlur = 0;

    // блоки
    for (const b of L.blocks) {
      const bx = b.x - state.camX;
      if (bx < -100 || bx > VW + 100) continue;
      ctx.fillStyle = '#00ffc8';
      ctx.shadowColor = '#00ffc8';
      ctx.shadowBlur = 12;
      ctx.fillRect(bx, b.y, b.w, b.h);
      ctx.shadowBlur = 0;
      ctx.strokeStyle = '#ffffff';
      ctx.lineWidth = 2;
      ctx.strokeRect(bx + 2, b.y + 2, b.w - 4, b.h - 4);
    }

    // шипы
    for (const s of L.spikes) {
      const sx = s.x - state.camX;
      if (sx < -100 || sx > VW + 100) continue;
      ctx.fillStyle = '#ff4060';
      ctx.shadowColor = '#ff4060';
      ctx.shadowBlur = 14;
      ctx.beginPath();
      ctx.moveTo(sx, GROUND_Y);
      ctx.lineTo(sx + s.w / 2, GROUND_Y - s.h);
      ctx.lineTo(sx + s.w, GROUND_Y);
      ctx.closePath();
      ctx.fill();
      ctx.shadowBlur = 0;
      ctx.strokeStyle = '#ffffff';
      ctx.lineWidth = 1.5;
      ctx.stroke();
    }

    // монеты
    for (const c of L.coins) {
      if (c.taken) continue;
      const cx = c.x - state.camX;
      if (cx < -50 || cx > VW + 50) continue;
      const bob = Math.sin(state.t / 300 + c.x) * 4;
      ctx.fillStyle = '#ffdd44';
      ctx.shadowColor = '#ffdd44';
      ctx.shadowBlur = 16;
      ctx.beginPath();
      ctx.arc(cx, c.y + bob, c.r, 0, Math.PI * 2);
      ctx.fill();
      ctx.shadowBlur = 0;
      ctx.fillStyle = '#fff8c0';
      ctx.beginPath();
      ctx.arc(cx - 3, c.y + bob - 3, c.r * 0.4, 0, Math.PI * 2);
      ctx.fill();
    }

    // финиш
    const fx = L.finishX - state.camX;
    if (fx > -50 && fx < VW + 50) {
      ctx.strokeStyle = '#00ff88';
      ctx.lineWidth = 4;
      ctx.shadowColor = '#00ff88';
      ctx.shadowBlur = 20;
      ctx.beginPath();
      ctx.moveTo(fx, 0);
      ctx.lineTo(fx, GROUND_Y);
      ctx.stroke();
      ctx.shadowBlur = 0;
      ctx.fillStyle = '#00ff88';
      ctx.font = 'bold 20px Arial';
      ctx.textAlign = 'center';
      ctx.fillText('ФИНИШ', fx, 40);
    }

    // игрок
    const p = state.player;
    const px = p.x - state.camX;
    ctx.save();
    ctx.translate(px, p.y - p.size / 2);
    ctx.rotate(p.rotation);
    ctx.fillStyle = '#00ffc8';
    ctx.shadowColor = '#00ffc8';
    ctx.shadowBlur = 20;
    ctx.fillRect(-p.size / 2, -p.size / 2, p.size, p.size);
    ctx.shadowBlur = 0;
    ctx.fillStyle = '#003a2e';
    ctx.fillRect(-p.size / 2 + 6, -p.size / 2 + 6, p.size - 12, p.size - 12);
    ctx.fillStyle = '#ffffff';
    ctx.fillRect(-p.size / 2 + 10, -p.size / 2 + 10, 6, 6);
    ctx.fillRect(p.size / 2 - 16, -p.size / 2 + 10, 6, 6);
    ctx.restore();

    // HUD
    ctx.fillStyle = '#ffffff';
    ctx.font = 'bold 16px Arial';
    ctx.textAlign = 'left';
    ctx.fillText('Монеты: ' + state.coins + ' / ' + L.coins.length, 20, VH - 20);
    ctx.fillText('Попытка: ' + state.attempts, 220, VH - 20);

    // прогресс
    const prog = Math.min(p.x / L.finishX, 1);
    ctx.fillStyle = 'rgba(255,255,255,0.15)';
    ctx.fillRect(20, VH - 50, 300, 8);
    ctx.fillStyle = '#00ffc8';
    ctx.fillRect(20, VH - 50, 300 * prog, 8);

    // пауза
    if (state.paused) {
      ctx.fillStyle = 'rgba(0,0,0,0.6)';
      ctx.fillRect(0, 0, VW, VH);
      ctx.fillStyle = '#00ffc8';
      ctx.font = 'bold 42px Arial';
      ctx.textAlign = 'center';
      ctx.fillText('ПАУЗА', VW / 2, VH / 2);
      ctx.font = '18px Arial';
      ctx.fillStyle = '#aaa';
      ctx.fillText('Esc — продолжить', VW / 2, VH / 2 + 40);
    }

    // меню
    if (state.menu) {
      ctx.fillStyle = 'rgba(0,0,0,0.7)';
      ctx.fillRect(0, 0, VW, VH);
      ctx.fillStyle = '#00ffc8';
      ctx.font = 'bold 52px Arial';
      ctx.textAlign = 'center';
      ctx.shadowColor = '#00ffc8';
      ctx.shadowBlur = 30;
      ctx.fillText('NEON CAMPAIGN', VW / 2, VH / 2 - 60);
      ctx.shadowBlur = 0;
      ctx.font = '20px Arial';
      ctx.fillStyle = '#ffffff';
      ctx.fillText('Нажми Пробел / Тап чтобы начать', VW / 2, VH / 2 + 20);
      ctx.font = '14px Arial';
      ctx.fillStyle = '#888';
      ctx.fillText('Тап / Пробел — прыжок · M — музыка · N — трек', VW / 2, VH / 2 + 60);
    }

    // вспышка
    if (state.flash > 0) {
      ctx.fillStyle = 'rgba(255,255,255,' + Math.min(state.flash, 0.8) + ')';
      ctx.fillRect(-20, -20, VW + 40, VH + 40);
    }

    ctx.restore();
  }

  // === Игровой цикл ===
  let lastTime = performance.now();
  function loop(now) {
    const dt = Math.min(now - lastTime, 50);
    lastTime = now;
    update(dt);
    draw();
    requestAnimationFrame(loop);
  }
  requestAnimationFrame(loop);

  // === Управление ===
  canvas.addEventListener('pointerdown', () => {
    if (state.menu) {
      startGame();
      return;
    }
    jump();
  });

  document.addEventListener('keydown', (e) => {
    if (e.code === 'Space' || e.code === 'ArrowUp') {
      e.preventDefault();
      if (state.menu) { startGame(); return; }
      jump();
    }
    if (e.code === 'Escape') {
      if (state.running) {
        state.paused = !state.paused;
      }
    }
    if (e.code === 'KeyM') {
      musicBtn.click();
    }
    if (e.code === 'KeyN') {
      nextTrack();
    }
  });

  pauseBtn.addEventListener('click', () => {
    if (state.running) state.paused = !state.paused;
  });

  menuExitBtn.addEventListener('click', () => {
    state.menu = true;
    state.running = false;
    state.paused = false;
    pauseBtn.style.display = 'none';
    menuExitBtn.style.display = 'none';
    resetAttempt();
  });

  // запуск первого трека
  loadTrack(0);

  // заглушка для дальнейшего кода (сеть, авторизация, друзья, чат, бан, редактор)
  // будет в части 3

})();
</script>
</body>
</html>
  // ============================================================
  // ХРАНИЛИЩЕ (localStorage)
  // ============================================================
  const LS = {
    users: 'gd_users',
    session: 'gd_session',
    friends: 'gd_friends',
    bans: 'gd_bans',
    maps: 'gd_maps',
    tos: 'gd_tos_accepted',
    disc: 'gd_disc_accepted'
  };
  function lsGet(k, def) {
    try { const v = localStorage.getItem(k); return v ? JSON.parse(v) : def; }
    catch (e) { return def; }
  }
  function lsSet(k, v) {
    try { localStorage.setItem(k, JSON.stringify(v)); } catch (e) {}
  }

  let currentUser = null; // { login, isGuest, id, banned }
  let myPeer = null;
  let myPeerId = null;
  let currentConn = null; // активное соединение с другом

  // ============================================================
  // ДИСКЛЕЙМЕР / TOS
  // ============================================================
  const discOverlay = document.getElementById('disclaimerOverlay');
  const discCheck = document.getElementById('discCheck');
  const discContinue = document.getElementById('discContinue');
  const discReadTos = document.getElementById('discReadTos');
  const tosOverlay = document.getElementById('tosOverlay');
  const tosCheck = document.getElementById('tosCheck');
  const tosAccept = document.getElementById('tosAccept');
  const tosDecline = document.getElementById('tosDecline');
  const authOverlay = document.getElementById('authOverlay');

  if (lsGet(LS.disc, false)) {
    discOverlay.classList.remove('active');
    showAuthOrGame();
  } else {
    discOverlay.classList.add('active');
  }

  discCheck.addEventListener('change', () => {
    discContinue.classList.toggle('ready', discCheck.checked);
  });
  discContinue.addEventListener('click', () => {
    lsSet(LS.disc, true);
    discOverlay.classList.remove('active');
    showAuthOrGame();
  });
  discReadTos.addEventListener('click', () => {
    tosOverlay.classList.add('active');
  });
  tosCheck.addEventListener('change', () => {
    tosAccept.disabled = !tosCheck.checked;
  });
  tosAccept.addEventListener('click', () => {
    if (!tosCheck.checked) return;
    lsSet(LS.tos, true);
    tosOverlay.classList.remove('active');
    if (!lsGet(LS.disc, false)) {
      discCheck.checked = true;
      discContinue.classList.add('ready');
    }
  });
  tosDecline.addEventListener('click', () => {
    tosOverlay.classList.remove('active');
  });

  // ============================================================
  // АВТОРИЗАЦИЯ
  // ============================================================
  const authTitle = document.getElementById('authTitle');
  const authErr = document.getElementById('authErr');
  const authUser = document.getElementById('authUser');
  const authPass = document.getElementById('authPass');
  const authSubmit = document.getElementById('authSubmit');
  const authGuest = document.getElementById('authGuest');
  const tabLogin = document.getElementById('tabLogin');
  const tabRegister = document.getElementById('tabRegister');
  let authMode = 'register';

  function setAuthMode(m) {
    authMode = m;
    if (m === 'login') {
      authTitle.textContent = 'Вход';
      authSubmit.textContent = 'Войти';
      tabLogin.classList.add('active');
      tabRegister.classList.remove('active');
    } else {
      authTitle.textContent = 'Регистрация';
      authSubmit.textContent = 'Зарегистрироваться';
      tabRegister.classList.add('active');
      tabLogin.classList.remove('active');
    }
    authErr.textContent = '';
  }
  tabLogin.addEventListener('click', () => setAuthMode('login'));
  tabRegister.addEventListener('click', () => setAuthMode('register'));

  function getUsers() { return lsGet(LS.users, {}); }
  function saveUsers(u) { lsSet(LS.users, u); }

  function isBanned(login) {
    const bans = lsGet(LS.bans, {});
    return !!bans[login];
  }

  authSubmit.addEventListener('click', () => {
    const login = authUser.value.trim();
    const pass = authPass.value.trim();
    authErr.textContent = '';
    if (login.length < 3) { authErr.textContent = 'Логин минимум 3 символа'; return; }
    if (pass.length < 3) { authErr.textContent = 'Пароль минимум 3 символа'; return; }

    const users = getUsers();
    if (authMode === 'register') {
      if (users[login]) { authErr.textContent = 'Логин занят'; return; }
      users[login] = { pass: pass, created: Date.now() };
      saveUsers(users);
      finishAuth(login, false);
    } else {
      if (!users[login]) { authErr.textContent = 'Нет такого логина'; return; }
      if (users[login].pass !== pass) { authErr.textContent = 'Неверный пароль'; return; }
      if (isBanned(login)) { authErr.textContent = 'Аккаунт забанен'; return; }
      finishAuth(login, false);
    }
  });

  authGuest.addEventListener('click', () => {
    finishAuth('Гость_' + Math.floor(Math.random() * 9000 + 1000), true);
  });

  function finishAuth(login, isGuest) {
    currentUser = { login: login, isGuest: isGuest, id: null };
    lsSet(LS.session, { login: login, isGuest: isGuest });
    authOverlay.classList.remove('active');
    state.menu = true;
    updateNetStatus();
    // регистрация в "онлайн-списке" локально
    addLocalOnline(login, isGuest);
  }

  function showAuthOrGame() {
    const sess = lsGet(LS.session, null);
    if (sess && sess.login) {
      if (isBanned(sess.login)) {
        lsSet(LS.session, null);
        authOverlay.classList.add('active');
        authErr.textContent = 'Аккаунт забанен';
        return;
      }
      currentUser = { login: sess.login, isGuest: !!sess.isGuest, id: null };
      authOverlay.classList.remove('active');
      state.menu = true;
      addLocalOnline(sess.login, !!sess.isGuest);
    } else {
      authOverlay.classList.add('active');
    }
    updateNetStatus();
  }

  // ============================================================
  // СЕТЬ (PeerJS)
  // ============================================================
  function updateNetStatus() {
    if (myPeerId) {
      netStatusEl.textContent = '● online';
      netStatusEl.classList.add('online');
      netStatusEl.classList.remove('offline');
    } else {
      netStatusEl.textContent = '● offline';
      netStatusEl.classList.add('offline');
      netStatusEl.classList.remove('online');
    }
  }

  function initPeer() {
    if (typeof Peer === 'undefined') {
      console.warn('PeerJS не загружен');
      return;
    }
    if (myPeer) return;
    try {
      myPeer = new Peer({ debug: 1 });
      myPeer.on('open', (id) => {
        myPeerId = id;
        currentUser.id = id;
        updateNetStatus();
        updateOnlineList();
      });
      myPeer.on('error', (err) => {
        console.warn('Peer error', err);
        myPeerId = null;
        updateNetStatus();
      });
      myPeer.on('connection', (conn) => {
        if (currentConn) {
          try { currentConn.close(); } catch (e) {}
        }
        currentConn = conn;
        attachConn(conn);
      });
    } catch (e) {
      console.warn('initPeer failed', e);
    }
  }

  function attachConn(conn) {
    conn.on('open', () => {
      chatHint.textContent = 'Подключено к другу. Можно общаться.';
      addChatMsg('sys', 'Соединение установлено');
      updateOnlineList();
    });
    conn.on('data', (data) => {
      if (data && data.type === 'chat') {
        addChatMsg(data.who || 'Друг', data.text);
        bumpChatBadge();
      } else if (data && data.type === 'hello') {
        addChatMsg('sys', 'Друг: ' + (data.who || '???'));
      }
    });
    conn.on('close', () => {
      chatHint.textContent = 'Соединение закрыто';
      addChatMsg('sys', 'Соединение закрыто');
      currentConn = null;
      updateOnlineList();
    });
    conn.on('error', (e) => {
      console.warn('conn error', e);
    });
  }

  function connectToPeer(peerId, label) {
    if (!myPeer) initPeer();
    if (!myPeer) { alert('Сеть не инициализирована'); return; }
    try {
      const conn = myPeer.connect(peerId, { reliable: true });
      currentConn = conn;
      attachConn(conn);
      conn.on('open', () => {
        conn.send({ type: 'hello', who: currentUser ? currentUser.login : '???' });
      });
      chatHint.textContent = 'Подключение к ' + (label || peerId) + '...';
    } catch (e) {
      alert('Не удалось подключиться: ' + e.message);
    }
  }

  // ============================================================
  // ОНЛАЙН-СПИСОК (локальные + друг)
  // ============================================================
  const onlineOverlay = document.getElementById('onlineOverlay');
  const onlineListEl = document.getElementById('onlineList');
  const onlineRefresh = document.getElementById('onlineRefresh');
  const onlineClose = document.getElementById('onlineClose');

  function addLocalOnline(login, isGuest) {
    // просто для отображения; храним в памяти
    onlineLocal = { login: login, isGuest: isGuest, since: Date.now() };
    updateOnlineList();
  }
  let onlineLocal = null;

  function updateOnlineList() {
    if (!onlineListEl) return;
    const rows = [];
    if (onlineLocal) {
      rows.push({
        name: onlineLocal.login + (onlineLocal.isGuest ? ' (гость)' : ''),
        meta: 'Это устройство',
        on: true
      });
    }
    if (currentConn && currentConn.peer) {
      rows.push({
        name: 'Друг',
        meta: currentConn.peer,
        on: currentConn.open
      });
    }
    if (rows.length === 0) {
      onlineListEl.innerHTML = '<div class="empty">Никого нет</div>';
      return;
    }
    onlineListEl.innerHTML = rows.map(r =>
      '<div class="onlineRow">' +
        '<div class="dot ' + (r.on ? '' : 'off') + '"></div>' +
        '<div class="oname">' + esc(r.name) + '</div>' +
        '<div class="ometa">' + esc(r.meta) + '</div>' +
      '</div>'
    ).join('');
  }
  onlineRefresh.addEventListener('click', updateOnlineList);
  onlineClose.addEventListener('click', () => onlineOverlay.classList.remove('active'));

  // ============================================================
  // ДРУЗЬЯ
  // ============================================================
  const friendsOverlay = document.getElementById('friendsOverlay');
  const friendName = document.getElementById('friendName');
  const friendPeerId = document.getElementById('friendPeerId');
  const friendAdd = document.getElementById('friendAdd');
  const friendAddCurrent = document.getElementById('friendAddCurrent');
  const friendsList = document.getElementById('friendsList');
  const friendOpenChat = document.getElementById('friendOpenChat');
  const friendsClose = document.getElementById('friendsClose');

  function getFriends() { return lsGet(LS.friends, []); }
  function saveFriends(f) { lsSet(LS.friends, f); }

  function renderFriends() {
    const f = getFriends();
    if (f.length === 0) {
      friendsList.innerHTML = '<div class="empty">Пока нет друзей</div>';
      return;
    }
    friendsList.innerHTML = f.map((fr, i) =>
      '<div class="friendRow">' +
        '<div class="fname">' + esc(fr.name) + '</div>' +
        '<div class="fid">' + esc(fr.id.slice(0, 12)) + '…</div>' +
        '<button data-connect="' + i + '">Связь</button>' +
        '<button class="del" data-del="' + i + '">✕</button>' +
      '</div>'
    ).join('');
    friendsList.querySelectorAll('button[data-connect]').forEach(btn => {
      btn.addEventListener('click', () => {
        const idx = +btn.getAttribute('data-connect');
        const fr = getFriends()[idx];
        connectToPeer(fr.id, fr.name);
        friendsOverlay.classList.remove('active');
      });
    });
    friendsList.querySelectorAll('button[data-del]').forEach(btn => {
      btn.addEventListener('click', () => {
        const idx = +btn.getAttribute('data-del');
        const arr = getFriends();
        arr.splice(idx, 1);
        saveFriends(arr);
        renderFriends();
      });
    });
  }

  friendAdd.addEventListener('click', () => {
    const name = friendName.value.trim();
    const id = friendPeerId.value.trim();
    if (!name || !id) { alert('Заполни имя и ID'); return; }
    const arr = getFriends();
    arr.push({ name: name, id: id });
    saveFriends(arr);
    friendName.value = '';
    friendPeerId.value = '';
    renderFriends();
  });

  friendAddCurrent.addEventListener('click', () => {
    if (!currentConn || !currentConn.peer) { alert('Нет активного соединения'); return; }
    const name = friendName.value.trim() || 'Друг';
    const arr = getFriends();
    arr.push({ name: name, id: currentConn.peer });
    saveFriends(arr);
    friendName.value = '';
    renderFriends();
  });

  friendOpenChat.addEventListener('click', () => {
    friendsOverlay.classList.remove('active');
    chatOverlay.classList.add('active');
    clearChatBadge();
  });
  friendsClose.addEventListener('click', () => friendsOverlay.classList.remove('active'));

  // ============================================================
  // ЧАТ
  // ============================================================
  const chatOverlay = document.getElementById('chatOverlay');
  const chatLog = document.getElementById('chatLog');
  const chatInput = document.getElementById('chatInput');
  const chatSend = document.getElementById('chatSend');
  const chatClose = document.getElementById('chatClose');
  const chatHint = document.getElementById('chatHint');
  const chatBadge = document.getElementById('chatBadge');
  let chatUnread = 0;

  function esc(s) {
    return String(s).replace(/[&<>"']/g, c => ({
      '&': '&amp;', '<': '&lt;', '>': '&gt;', '"': '&quot;', "'": '&#39;'
    }[c]));
  }

  function addChatMsg(who, text) {
    const div = document.createElement('div');
    div.className = 'msg';
    const cls = who === 'sys' ? 'sys' : (currentUser && who === currentUser.login ? 'me' : '');
    div.innerHTML = '<span class="who ' + cls + '">' + esc(who) + ':</span> ' + esc(text);
    chatLog.appendChild(div);
    chatLog.scrollTop = chatLog.scrollHeight;
  }

  function bumpChatBadge() {
    if (!chatOverlay.classList.contains('active')) {
      chatUnread++;
      chatBadge.style.display = 'inline-block';
      chatBadge.textContent = chatUnread;
    }
  }
  function clearChatBadge() {
    chatUnread = 0;
    chatBadge.style.display = 'none';
  }

  function sendChat() {
    const text = chatInput.value.trim();
    if (!text) return;
    if (!currentConn || !currentConn.open) {
      alert('Нет соединения с другом');
      return;
    }
    const who = currentUser ? currentUser.login : 'Я';
    try {
      currentConn.send({ type: 'chat', who: who, text: text });
      addChatMsg(who, text);
      chatInput.value = '';
    } catch (e) {
      alert('Ошибка отправки: ' + e.message);
    }
  }
  chatSend.addEventListener('click', sendChat);
  chatInput.addEventListener('keydown', (e) => {
    if (e.key === 'Enter') { e.preventDefault(); sendChat(); }
  });
  chatClose.addEventListener('click', () => {
    chatOverlay.classList.remove('active');
    clearChatBadge();
  });

  // ============================================================
  // БАН-ПАНЕЛЬ
  // ============================================================
  const banOverlay = document.getElementById('banOverlay');
  const banList = document.getElementById('banList');
  const banStats = document.getElementById('banStats');
  const banClose = document.getElementById('banClose');

  function renderBans() {
    const users = getUsers();
    const bans = lsGet(LS.bans, {});
    const logins = Object.keys(users);
    if (logins.length === 0) {
      banList.innerHTML = '<div class="empty">Нет игроков</div>';
      banStats.textContent = '';
      return;
    }
    banStats.textContent = 'Всего: ' + logins.length + ' · Забанено: ' + Object.keys(bans).length;
    banList.innerHTML = logins.map(login => {
      const u = users[login];
      const banned = !!bans[login];
      return '<div class="userRow ' + (banned ? 'banned' : '') + '">' +
        '<div class="top">' +
          '<div class="name">' + esc(login) + '</div>' +
          '<div class="actions">' +
            '<button class="banBtn" data-ban="' + esc(login) + '">' +
              (banned ? 'Разбанить' : 'Забанить') +
            '</button>' +
          '</div>' +
        '</div>' +
        '<div class="meta">' +
          '<span>Создан: ' + new Date(u.created).toLocaleDateString() + '</span>' +
          '<span class="' + (banned ? 'offline' : 'online') + '">' +
            (banned ? 'Забанен' : 'Активен') +
          '</span>' +
        '</div>' +
      '</div>';
    }).join('');
    banList.querySelectorAll('button[data-ban]').forEach(btn => {
      btn.addEventListener('click', () => {
        const login = btn.getAttribute('data-ban');
        toggleBan(login);
      });
    });
  }

  function toggleBan(login) {
    const bans = lsGet(LS.bans, {});
    if (bans[login]) delete bans[login];
    else bans[login] = { since: Date.now() };
    lsSet(LS.bans, bans);
    renderBans();
    if (currentUser && currentUser.login === login && bans[login]) {
      lsSet(LS.session, null);
      location.reload();
    }
  }
  banClose.addEventListener('click', () => banOverlay.classList.remove('active'));

  // ============================================================
  // РЕДАКТОР КАРТ
  // ============================================================
  const editorOverlay = document.getElementById('editorOverlay');
  const edName = document.getElementById('edName');
  const edSpeed = document.getElementById('edSpeed');
  const edLength = document.getElementById('edLength');
  const edX = document.getElementById('edX');
  const edList = document.getElementById('edList');
  const edSave = document.getElementById('edSave');
  const edClose = document.getElementById('edClose');
  let editMap = { name: 'Моя карта', speed: 5, length: 4000, spikes: [], blocks: [], coins: [] };

  function renderEdList() {
    const parts = [];
    editMap.spikes.forEach(s => parts.push('Шип @ ' + s.x));
    editMap.blocks.forEach(b => parts.push('Блок @ ' + b.x));
    editMap.coins.forEach(c => parts.push('Монета @ ' + c.x));
    edList.innerHTML = parts.length ? parts.join('<br>') : '<i>Пусто</i>';
  }

  function openEditor() {
    editorOverlay.classList.add('active');
    renderEdList();
  }
  editorOverlay.querySelectorAll('button[data-ed]').forEach(btn => {
    btn.addEventListener('click', () => {
      const kind = btn.getAttribute('data-ed');
      const x = parseInt(edX.value, 10) || 800;
      if (kind === 'spike') editMap.spikes.push({ x: x, w: 36, h: 36 });
      else if (kind === 'block') editMap.blocks.push({ x: x, y: GROUND_Y - 36, w: 60, h: 36 });
      else if (kind === 'coin') editMap.coins.push({ x: x, y: GROUND_Y - 120, r: 14, taken: false });
      else if (kind === 'undo') {
        if (editMap.spikes.length) editMap.spikes.pop();
        else if (editMap.blocks.length) editMap.blocks.pop();
        else if (editMap.coins.length) editMap.coins.pop();
      } else if (kind === 'clear') {
        editMap.spikes = []; editMap.blocks = []; editMap.coins = [];
      }
      renderEdList();
    });
  });

  edSave.addEventListener('click', () => {
    editMap.name = edName.value.trim() || 'Без названия';
    editMap.speed = parseFloat(edSpeed.value) || 5;
    editMap.length = parseInt(edLength.value, 10) || 4000;
    const maps = lsGet(LS.maps, []);
    maps.push(editMap);
    lsSet(LS.maps, maps);
    alert('Карта сохранена: ' + editMap.name);
  });
  edClose.addEventListener('click', () => editorOverlay.classList.remove('active'));

  // ============================================================
  // ГОРЯЧИЕ КЛАВИШИ ДЛЯ МЕНЮ (доп.)
  // ============================================================
  document.addEventListener('keydown', (e) => {
    if (e.code === 'KeyF') { friendsOverlay.classList.add('active'); renderFriends(); }
    if (e.code === 'KeyC') { chatOverlay.classList.add('active'); clearChatBadge(); }
    if (e.cod
