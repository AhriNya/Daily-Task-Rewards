// ==UserScript==
// @name         Bing 自动搜索 - V48.14 智能首启版
// @namespace    http://tampermonkey.net/
// @version      48.14
// @description  每日首次打开默认为关闭状态，手动开启后全天记住状态。极致毛玻璃 UI。
// @author       Gemini Assistant
// @match        https://www.bing.com/*
// @match        https://cn.bing.com/*
// @connect      baidu.com
// @grant        GM_xmlhttpRequest
// @grant        GM_setValue
// @grant        GM_getValue
// @run-at       document-end
// ==/UserScript==

(function() {
    'use strict';

    const I18N = {
        STATUS_READY: "准备开始",
        STATUS_FETCH: "获取数据中",
        STATUS_TYPE: "打字中",
        STATUS_AUDIT: "阅读中",
        STATUS_WAIT: "等待中",
        TEXT_FETCHING: "寻找今日热搜...",
        TEXT_AUDITING: "模拟浏览网页",
        TEXT_WAITING: "任务冷却中...",
        BTN_LOG: "历史",
        BTN_SKIP: "跳过",
        BTN_STOP: "停止",
        LOG_TITLE: "搜索历史",
        LOG_CLOSE: "关闭"
    };

    const CONFIG = {
        readTime: [15000, 30000],
        idleTime: [300000, 900000],
        historyLimit: 2000,
        suffixes: [" 评价", " 怎么回事", " 最新消息", " 2025", " 攻略", " 官网", " 推荐", " 价格", " 怎么样", " 哪个好", " 值得买吗", " 对比", " 排名", " 排行榜", " 评测", " 体验", " 优缺点", " 教程", " 视频", " 图片", " 下载", " 安装", " 购买", " 哪里买", " 多少钱一个", " 效果", " 使用方法", " 功能", " 介绍", " 详细", " 深度", " 全面", " 分析", " 解读", " 报告", " 数据", " 统计", " 趋势", " 预测", " 发展", " 评测", " 分析", " 指南", " 教程", " 报告", " 攻略", " 对比", " 排名", " 购买建议", " 使用技巧", " 入门指南", " 进阶教程", " 深度分析", " 全面解读", " 用户体验", " 效果验证", " 质量评估", " 安全检测", " 性能测试", " 功能解析", " 2025趋势", " 市场前景", " 投资价值", " 性价比", " 竞品分析", " 新手教程", " 实战案例", " 成功经验", " 常见问题", " 解决方案", " 技术原理", " 工作机制", " 优缺点", " brand对比", " 产品对比", " 创新点", " 突破技术", " 发展历程", " 技术演进", " 市场份额", " 用户满意度", " 口碑分析", " 社会价值", " 经济效益", " 未来展望", " 应用场景", " 适用人群", " 操作指南", " 维护保养", " 故障排除"]
    };

    const KEYS = {
        RUNNING: 'bot_v48_run_state',
        HISTORY: 'bot_v48_hist',
        NEXT_TIME: 'bot_v48_next',
        START: 'bot_v48_start',
        LOGS: 'bot_v48_logs',
        TODAY_COUNT: 'bot_v48_daily_count',
        LAST_DATE: 'bot_v48_last_date',
        LAST_RUN_DATE: 'bot_v48_last_run_date' // 新增：记录最后一次运行状态的日期
    };

    const Utils = {
        sleep: ms => new Promise(r => setTimeout(r, ms)),
        rand: (min, max) => Math.floor(Math.random() * (max - min + 1) + min),
        checkDailyReset() {
            const today = new Date().toLocaleDateString();
            // 每日清零计数
            if (today !== GM_getValue(KEYS.LAST_DATE, "")) {
                GM_setValue(KEYS.TODAY_COUNT, 0);
                GM_setValue(KEYS.LAST_DATE, today);
            }
            // 关键逻辑：每日首启检测
            if (today !== GM_getValue(KEYS.LAST_RUN_DATE, "")) {
                GM_setValue(KEYS.RUNNING, false); // 跨天了，默认关闭
                // 注意：这里不更新 LAST_RUN_DATE，只有在用户点击“开启”时才更新
            }
        },
        getSearchInput() {
            return document.querySelector('#sb_form_q') || document.querySelector('input[type="search"]') || document.querySelector('input[name="q"]');
        },
        async humanType(input, text) {
            input.focus();
            input.value = "";
            await this.sleep(400);
            for (let i = 0; i < text.length; i++) {
                input.value += text[i];
                input.dispatchEvent(new InputEvent('input', { bubbles: true }));
                await this.sleep(this.rand(80, 180));
            }
        },
        async autoScroll() {
            if (sessionStorage.getItem('search_pending') === 'true') {
                GM_setValue(KEYS.TODAY_COUNT, GM_getValue(KEYS.TODAY_COUNT, 0) + 1);
                sessionStorage.removeItem('search_pending');
                UI.updateCountUI();
            }
            await this.sleep(2000);
            for (let i = 0; i < this.rand(2, 4); i++) {
                window.scrollBy({ top: this.rand(300, 600), behavior: 'smooth' });
                await this.sleep(this.rand(3000, 5000));
            }
        }
    };

    const UI = {
        init() {
            if (document.getElementById('bot-container')) return;
            const style = document.createElement('style');
            style.innerHTML = `
                #bot-container { position: fixed; bottom: 30px; left: 30px; z-index: 2147483647; font-family: 'Segoe UI', system-ui, sans-serif; }
                .bot-launcher {
                    width: 50px; height: 50px; border-radius: 25px;
                    background: rgba(0, 120, 212, 0.85); backdrop-filter: blur(12px);
                    box-shadow: 0 8px 32px rgba(0,0,0,0.2), inset 0 0 0 1px rgba(255,255,255,0.2);
                    display: flex; align-items: center; justify-content: center;
                    cursor: pointer; transition: all 0.5s cubic-bezier(0.165, 0.84, 0.44, 1); color: white;
                    opacity: 0.1;
                }
                .bot-launcher:hover { opacity: 1; transform: scale(1.1); background: rgba(0, 120, 212, 1); }
                .glass-card {
                    display: none; width: 300px;
                    background: rgba(255, 255, 255, 0.65);
                    backdrop-filter: blur(25px) saturate(180%); -webkit-backdrop-filter: blur(25px) saturate(180%);
                    border: 1px solid rgba(255, 255, 255, 0.4); border-radius: 28px; padding: 22px;
                    box-shadow: 0 20px 50px rgba(0,0,0,0.15);
                    animation: botFlyIn 0.5s cubic-bezier(0.23, 1, 0.32, 1);
                }
                @keyframes botFlyIn { from { opacity: 0; transform: translateY(20px); } to { opacity: 1; transform: translateY(0); } }
                .status-badge { font-size: 10px; font-weight: 900; padding: 5px 12px; border-radius: 50px; background: rgba(0,120,212,0.12); color: #005a9e; }
                .kw-text { font-size: 15px; font-weight: 600; color: #111; margin: 18px 0 10px 0; min-height: 22px; }
                .progress-container { height: 7px; background: rgba(0,0,0,0.06); border-radius: 10px; overflow: hidden; margin-bottom: 22px; }
                .progress-fill { height: 100%; width: 0%; background: linear-gradient(90deg, #0078d4, #2baffc); transition: width 0.15s linear; }
                .btn-group { display: flex; gap: 10px; }
                .action-btn { flex: 1; border: none; padding: 12px; border-radius: 16px; font-size: 11px; font-weight: 700; cursor: pointer; background: rgba(0,0,0,0.05); transition: all 0.2s; color: #333; }
                .action-btn:hover { background: rgba(0,0,0,0.1); transform: translateY(-1px); }
                #bot-timer { font-family: 'Cascadia Code', 'Consolas', monospace; font-size: 13px; color: #0078d4; font-weight: 800; }
                #bot-log-modal { display: none !important; position: fixed; top: 50%; left: 50%; transform: translate(-50%, -50%); width: 380px; background: rgba(255,255,255,0.95); backdrop-filter: blur(30px); z-index: 2147483648; padding: 30px; border-radius: 32px; box-shadow: 0 30px 100px rgba(0,0,0,0.4); border: 1px solid rgba(255,255,255,0.5); }
            `;
            document.head.appendChild(style);

            const container = document.createElement('div');
            container.id = 'bot-container';
            container.innerHTML = `
                <div class="bot-launcher" id="bot-launch-btn"><svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"><path d="M12 2L2 7l10 5 10-5-10-5zM2 17l10 5 10-5M2 12l10 5 10-5"/></svg></div>
                <div class="glass-card" id="bot-panel">
                    <div style="display:flex; justify-content:space-between; align-items:center;">
                        <div><span class="status-badge" id="bot-status">${I18N.STATUS_READY}</span><span id="bot-count-ui" style="font-size:11px; font-weight:900; color:#0078d4; margin-left:8px;">0</span></div>
                        <span id="bot-timer">0.0s</span>
                    </div>
                    <div class="kw-text" id="bot-kw">${I18N.STATUS_READY}</div>
                    <div class="progress-container"><div id="bot-fill" class="progress-fill"></div></div>
                    <div class="btn-group">
                        <button class="action-btn" id="b-log">${I18N.BTN_LOG}</button>
                        <button class="action-btn" id="b-skip" style="background:rgba(0,120,212,0.85); color:white;">${I18N.BTN_SKIP}</button>
                        <button class="action-btn" id="b-stop">${I18N.BTN_STOP}</button>
                    </div>
                </div>
            `;
            document.body.appendChild(container);

            const modal = document.createElement('div');
            modal.id = 'bot-log-modal';
            modal.innerHTML = `<h3 style="margin-top:0; color:#111;">${I18N.LOG_TITLE}</h3><div id="l-content" style="max-height:300px;overflow-y:auto;font-size:12px; color:#444;"></div><button id="b-close" style="width:100%;padding:12px;margin-top:15px;border:none;border-radius:16px;background:#f0f0f0;font-weight:700;cursor:pointer;">${I18N.LOG_CLOSE}</button>`;
            document.body.appendChild(modal);

            document.getElementById('bot-launch-btn').onclick = () => {
                const today = new Date().toLocaleDateString();
                GM_setValue(KEYS.RUNNING, true);
                GM_setValue(KEYS.LAST_RUN_DATE, today); // 只要开启了，就更新今日已运行标识
                location.reload();
            };

            document.getElementById('b-skip').onclick = () => { GM_setValue(KEYS.NEXT_TIME, 0); location.assign('https://www.bing.com'); };

            document.getElementById('b-stop').onclick = () => {
                GM_setValue(KEYS.RUNNING, false);
                GM_setValue(KEYS.LAST_RUN_DATE, ""); // 手动停止后，下次刷新即便同天也会是关闭状态（或者你可以保留这一行来让手动停止仅限本次刷新）
                location.reload();
            };

            document.getElementById('b-log').onclick = () => { document.getElementById('bot-log-modal').style.setProperty('display', 'block', 'important'); this.updateLogs(); };
            document.getElementById('b-close').onclick = () => document.getElementById('bot-log-modal').style.setProperty('display', 'none', 'important');

            const isRunning = GM_getValue(KEYS.RUNNING, false);
            document.getElementById('bot-launch-btn').style.display = isRunning ? 'none' : 'flex';
            document.getElementById('bot-panel').style.display = isRunning ? 'block' : 'none';
            this.updateCountUI();
        },
        updateCountUI() {
            const count = GM_getValue(KEYS.TODAY_COUNT, 0);
            if(document.getElementById('bot-count-ui')) document.getElementById('bot-count-ui').innerText = count;
        },
        update(status, kw, progress, timerText) {
            const s = document.getElementById('bot-status'), k = document.getElementById('bot-kw'), f = document.getElementById('bot-fill'), t = document.getElementById('bot-timer');
            if (s && status) s.innerText = status;
            if (k && kw) k.innerText = kw;
            if (f && progress !== undefined) f.style.width = Math.max(0, Math.min(100, progress)) + "%";
            if (t && timerText) t.innerText = timerText;
        },
        updateLogs() {
            const logs = GM_getValue(KEYS.LOGS, []);
            document.getElementById('l-content').innerHTML = logs.slice().reverse().map(l => `<div style="padding:10px 0;border-bottom:1px solid rgba(0,0,0,0.05);">${l.time} - ${l.word} <b style="color:#0078d4">${l.suffix}</b></div>`).join('');
        }
    };

    const Controller = {
        async fetch() {
            UI.update(I18N.STATUS_FETCH, I18N.TEXT_FETCHING, 50, "GO");
            GM_xmlhttpRequest({
                method: "GET", url: "https://top.baidu.com/board?tab=realtime",
                onload: async (res) => {
                    const doc = new DOMParser().parseFromString(res.responseText, "text/html");
                    const words = Array.from(doc.querySelectorAll('.c-single-text-ellipsis')).map(e => e.innerText.trim());
                    let history = GM_getValue(KEYS.HISTORY, []);
                    let pool = words.filter(w => w && !history.includes(w));
                    if (pool.length === 0) { history = []; pool = words; }
                    const target = pool[Math.floor(Math.random() * pool.length)];
                    const suffix = CONFIG.suffixes[Math.floor(Math.random() * CONFIG.suffixes.length)];
                    const input = Utils.getSearchInput();
                    if (input) {
                        UI.update(I18N.STATUS_TYPE, target, 100, "WAIT");
                        await Utils.humanType(input, target + suffix);
                        sessionStorage.setItem('search_pending', 'true');
                        history.push(target);
                        GM_setValue(KEYS.HISTORY, history.slice(-CONFIG.historyLimit));
                        let logs = GM_getValue(KEYS.LOGS, []);
                        logs.push({ time: new Date().toLocaleTimeString(), word: target, suffix: suffix });
                        GM_setValue(KEYS.LOGS, logs.slice(-100));
                        await Utils.sleep(800);
                        const form = input.closest('form');
                        if (form) form.submit();
                        else {
                            const e = new KeyboardEvent('keydown', { bubbles: true, cancelable: true, key: 'Enter', keyCode: 13 });
                            input.dispatchEvent(e);
                        }
                    }
                }
            });
        },
        startLogic() {
            const isResult = window.location.href.includes('search?q=') || !!document.querySelector('#b_results') || !!document.querySelector('.b_results');
            if (isResult) {
                const duration = Utils.rand(CONFIG.readTime[0], CONFIG.readTime[1]);
                const startTime = Date.now();
                Utils.autoScroll();
                const uiTimer = setInterval(() => {
                    if (!GM_getValue(KEYS.RUNNING, false)) { clearInterval(uiTimer); return; }
                    const now = Date.now();
                    const elapsed = now - startTime;
                    const remaining = Math.max(0, duration - elapsed);
                    UI.update(I18N.STATUS_AUDIT, I18N.TEXT_AUDITING, (1 - (elapsed / duration)) * 100, (remaining / 1000).toFixed(1) + "s");
                    if (elapsed >= duration) {
                        clearInterval(uiTimer);
                        GM_setValue(KEYS.NEXT_TIME, Date.now() + Utils.rand(CONFIG.idleTime[0], CONFIG.idleTime[1]));
                        GM_setValue(KEYS.START, Date.now());
                        location.assign('https://www.bing.com');
                    }
                }, 100);
            } else {
                const waitTimer = setInterval(() => {
                    if (!GM_getValue(KEYS.RUNNING, false)) { clearInterval(waitTimer); return; }
                    const now = Date.now(), next = GM_getValue(KEYS.NEXT_TIME, 0), start = GM_getValue(KEYS.START, now);
                    if (now >= next) {
                        if (!window.isExec) {
                            window.isExec = true;
                            clearInterval(waitTimer);
                            this.fetch();
                        }
                    } else {
                        const progress = ((now - start) / (next - start)) * 100;
                        UI.update(I18N.STATUS_WAIT, I18N.TEXT_WAITING, progress, Math.ceil((next - now) / 1000) + "s");
                    }
                }, 500);
            }
        },
        run() {
            Utils.checkDailyReset();
            UI.init();
            if (GM_getValue(KEYS.RUNNING, false)) {
                this.startLogic();
            }
        }
    };
    Controller.run();
})();
