# ❓ Frequently Asked Questions (FAQ)

This document compiles common questions users encounter during use and their solutions.

---

## 📊 Data-Related

### Q1: US Stock Codes (e.g., AMD, AAPL) Show Incorrect Prices During Analysis?

**Symptom**: After entering US stock codes, displayed prices are clearly incorrect (e.g., AMD shows ¥7.33) or are misidentified as A-shares.

**Reason**: Early version code matching logic prioritized A-share rules first, causing code conflicts.

**Solution**:
1. Fixed in v2.3.0; the system now supports automatic US stock code recognition
2. If issues persist, set in `.env`:
   ```bash
   YFINANCE_PRIORITY=0
   ```
   This prioritizes Yahoo Finance data source for US stock data

> 📌 Related Issue: [#153](https://github.com/ZhuLinsen/daily_stock_analysis/issues/153)

---

### Q2: "Volume Ratio" Field in Reports Shows Empty or N/A?

**Symptom**: Volume ratio data missing from analysis reports, affecting AI judgment of volume scaling.

**Reason**: Some default real-time quote sources (like Sina interface) don't provide volume ratio fields.

**Solution**:
1. Fixed in v2.3.0; Tencent interface now supports volume ratio parsing
2. Recommend configuring real-time quote source priority:
   ```bash
   REALTIME_SOURCE_PRIORITY=tencent,akshare_sina,efinance,akshare_em
   ```
3. System has built-in 5-day average volume calculation as fallback logic

> 📌 Related Issue: [#155](https://github.com/ZhuLinsen/daily_stock_analysis/issues/155)

---

### Q3: Tushare Data Fetch Fails with "Token Error" Message?

**Symptom**: Logs show `Tushare data fetch failed: Your token is incorrect, please confirm`

**Solution**:
1. **No Tushare Account**: No need to configure `TUSHARE_TOKEN`; system automatically uses free data sources (AkShare, Efinance)
2. **Have Tushare Account**: Confirm token is correct, check in [Tushare Pro](https://tushare.pro/weborder/#/login?reg=834638) account center
3. All core features of this project work normally without Tushare

---

### Q4: Data Fetching Rate-Limited or Returns Empty?

**Symptom**: Logs show `circuit breaker triggered` or `RemoteDisconnected`, or connection to `push2his.eastmoney.com` closed, etc.

**Reason**: Free data sources (East Money Finance, Sina, etc.) have anti-crawl mechanisms; heavy requests in short time trigger rate limiting.

**Solution**:
1. System has built-in multi-data-source auto-switching and circuit breaker protection
2. Reduce number of watchlist stocks or increase request intervals
3. Avoid frequent manual analysis triggers
4. If East Finance interface frequently fails, enable `ENABLE_EASTMONEY_PATCH=true` to inject NID token and random User-Agent (reduces rate-limit probability)
5. Change `MAX_WORKERS=1` to serial fetching to reduce concurrent pressure on East Finance

---

## ⚙️ Configuration-Related

### Q5: GitHub Actions Fails with "Environment Variable Not Found"?

**Symptom**: Actions logs show `GEMINI_API_KEY` or `STOCK_LIST` undefined

**Reason**: GitHub distinguishes `Secrets` (encrypted) and `Variables` (plain), wrong location causes read failure.

**Solution**:
1. Go to repository `Settings` → `Secrets and variables` → `Actions`
2. **Secrets** (click `New repository secret`): Store sensitive information
   - `GEMINI_API_KEY`
   - `OPENAI_API_KEY`
   - `TELEGRAM_BOT_TOKEN`
   - Various Webhook URLs
3. **Variables** (click `Variables` tab): Store non-sensitive configuration
   - `STOCK_LIST`
   - `GEMINI_MODEL`
   - `REPORT_TYPE`

---

### Q6: Configuration Doesn't Take Effect After Modifying .env?

**Solution**:
1. Ensure `.env` is in project root directory (same directory as `main.py`)
2. **Docker deploy / WebUI system settings**:
   - WebUI-saved `STOCK_LIST`, `SCHEDULE_ENABLED`, `SCHEDULE_TIME`, `SCHEDULE_RUN_IMMEDIATELY`, `RUN_IMMEDIATELY` write back to container `.env`
   - After WebUI save, triggers configuration reload in current process; running read paths will use latest saved `.env`, e.g., scheduled tasks will continuously hot-read saved `STOCK_LIST`
   - If container startup command explicitly passed same-named environment variables (like `docker run -e ...` or Compose `environment:`), they still override at restart; to let WebUI saved values take over, update or remove these explicit overrides
   - `SCHEDULE_*` and `RUN_IMMEDIATELY` are **startup-period scheduling configuration**; after save, does not immediately trigger analysis or hot-rebuild scheduler in current process
   - To immediately apply scheduling toggles in current container, restart container and ensure starting in schedule mode
3. **Docker manual `.env` modification**: After modification, recommend restarting container
   ```bash
   docker-compose down && docker-compose up -d
   ```
4. **GitHub Actions**: `.env` file has no effect; must configure in Secrets/Variables
5. Check for multiple `.env` files (like `.env.local`) causing overrides

---

### Q7: How to Configure Proxy for Gemini/OpenAI API?

**Solution**:

Configure in `.env`:
```bash
USE_PROXY=true
PROXY_HOST=127.0.0.1
PROXY_PORT=10809
```

> ⚠️ Note: Proxy configuration only works for local runs; GitHub Actions environment needs no proxy configuration.

---

### LLM Configuration Common Questions

> Complete documentation at [LLM Configuration Guide](LLM_CONFIG_GUIDE.md).

**Q: Why use only one channel after configuring GEMINI_API_KEY and LLM_CHANNELS?**

System uses priority-based single selection: Advanced model routing YAML (`LITELLM_CONFIG`) > `LLM_CHANNELS` > legacy keys. But YAML only takes effect when file parses correctly and produces valid `model_list`; if YAML path invalid or content empty, system auto-fallback to `LLM_CHANNELS` or legacy keys. Once one priority level actually takes effect, lower priority configuration does not participate in parsing.

**Q: check_env outputs "No available AI models configured," what do I do?**

First select one service provider and fill in corresponding API Key; if you need to fix main model, then add `LITELLM_MODEL=provider/model`; if you want multi-model switching, then configure `LLM_CHANNELS` or advanced model routing YAML. Run `python scripts/check_env.py --config` to validate configuration, `python scripts/check_env.py --llm` to actually call API testing.

**Q: How to use multiple models simultaneously (e.g., AIHubmix + DeepSeek + Gemini)?**

Use channel mode: set `LLM_CHANNELS=aihubmix,deepseek,gemini` and configure each channel's `LLM_{NAME}_BASE_URL`, `LLM_{NAME}_API_KEY`, `LLM_{NAME}_MODELS`. Or configure visually in Web settings page → AI Models → AI Model Access.

**Q: Stock Q&A/Agent shows no available LLM configured, but I only have old `GEMINI_*` / `OPENAI_*` / `ANTHROPIC_*` configuration, what to do?**

First confirm whether current session enabled `LITELLM_CONFIG` or `LLM_CHANNELS`; if enabled, upper-level configuration overrides legacy keys. If you didn't enable these two levels and `AGENT_LITELLM_MODEL` is empty, Q&A Agent still auto-inherits legacy provider model: `GEMINI_MODEL`, `OPENAI_MODEL`, `ANTHROPIC_MODEL` respectively map to corresponding LiteLLM model names with provider prefix. This fix does not silently migrate or clear old configuration, only directly returns "real missing cause" to frontend for you to judge whether it's missing key, missing model name, or being overridden by upper-level configuration. Complete compatibility semantics in [LLM Configuration Guide](LLM_CONFIG_GUIDE.md) "Q&A Agent / LiteLLM Configuration Compatibility".

---

## 📱 Push-Related

### Q8: Bot Push Fails with "Message Too Long"?

**Symptom**: Analysis succeeds but no push received; logs show 400 error or `Message too long`

**Reason**: Different platforms have different message length limits:
- WeChat Work: 4KB
- Feishu: 20KB
- DingTalk: 20KB

**Solution**:
1. **Auto-chunking**: Latest version already implements long message auto-splitting
2. **Single stock push mode**: Set `SINGLE_STOCK_NOTIFY=true` to push immediately after analyzing each stock
3. **Simplify report**: Set `REPORT_TYPE=simple` to use simplified format

---

### Q9: Can't Receive Telegram Push Messages?

**Solution**:
1. Confirm both `TELEGRAM_BOT_TOKEN` and `TELEGRAM_CHAT_ID` are configured
2. Get Chat ID method:
   - Send any message to Bot
   - Visit `https://api.telegram.org/bot<TOKEN>/getUpdates`
   - Find `chat.id` in returned JSON
3. Ensure Bot is added to target group (if group chat)
4. Local run requires access to Telegram API (may need proxy)

---

### Q10: WeChat Work Markdown Format Displays Incorrectly?

**Solution**:
1. WeChat Work has limited Markdown support; try setting:
   ```bash
   WECHAT_MSG_TYPE=text
   ```
2. This sends plain text format messages

---

## 🤖 AI Model-Related

### Q11: Gemini API Returns 429 Error (Too Many Requests)?

**Symptom**: Logs show `Resource has been exhausted` or `429 Too Many Requests`

**Solution**:
1. Gemini free tier has rate limits (approx 15 RPM)
2. Reduce number of stocks analyzed simultaneously
3. Increase request delay:
   ```bash
   GEMINI_REQUEST_DELAY=5
   ANALYSIS_DELAY=10
   ```
4. Or switch to OpenAI compatible API as backup

---

### Q12: How to Use DeepSeek and Other Domestic Models?

**Configuration Method**:

```bash
# No need to configure GEMINI_API_KEY
OPENAI_API_KEY=sk-xxxxxxxx
OPENAI_BASE_URL=https://api.deepseek.com
OPENAI_MODEL=deepseek-v4-flash
# deepseek-chat / deepseek-reasoner still compatible but officially deprecated after 2026/07/24
```

Supported model services:
- DeepSeek: `https://api.deepseek.com`
- Tongyi Qianwen: `https://dashscope.aliyuncs.com/compatible-mode/v1`
- Moonshot: `https://api.moonshot.cn/v1`

---

### Q12b: How to Use Ollama Local Models?

**Configuration Method**: Use `OLLAMA_API_BASE` + `LITELLM_MODEL`, or channel mode (`LLM_CHANNELS=ollama` + `LLM_OLLAMA_BASE_URL` + `LLM_OLLAMA_MODELS`).

**Pitfall**: Don't configure Ollama in `OPENAI_BASE_URL`, or system will incorrectly splice URL (like 404, `api/generate/api/show`). See [LLM Configuration Guide](LLM_CONFIG_GUIDE.md) example 4 and channel example.

---

### Q12c: Runtime Shows `OllamaException / APIConnectionError` (All LLM models failed)?

**Symptoms**: Logs show `litellm.APIConnectionError: OllamaException` or `Analysis failed: All LLM models failed (tried 1 model(s))`.

Check following 5 checkpoints sequentially:

1. **Is Ollama service started?**
   ```bash
   # Check process
   pgrep -a ollama
   # If no output, start it first
   ollama serve
   ```
   Confirm service listening: `curl http://localhost:11434`, should return `Ollama is running`.

2. **Is `OLLAMA_API_BASE` configured correctly?**
   - ✅ Correct: `OLLAMA_API_BASE=http://localhost:11434`
   - ❌ Wrong: Put Ollama address in `OPENAI_BASE_URL`, causes URL path errors (e.g., `…/api/generate/api/show`)

3. **Is model name prefixed with `ollama/`?**
   - ✅ Correct: `LITELLM_MODEL=ollama/qwen3:8b`
   - ❌ Wrong: `LITELLM_MODEL=qwen3:8b` (missing prefix, litellm cannot route to Ollama)

4. **Is model already downloaded locally?**
   ```bash
   ollama list          # View existing models
   ollama pull qwen3:8b # Download if missing
   ```

5. **Remote deploy / Docker network and firewall**
   - If Ollama and program not on same host, change `OLLAMA_API_BASE` to actual IP, like `http://192.168.1.100:11434`
   - Confirm firewall allows port 11434 and Ollama started binding correct address (`OLLAMA_HOST=0.0.0.0:11434`)

> Complete configuration example in [LLM Configuration Guide → Example 4 (Ollama)](LLM_CONFIG_GUIDE.md#example-4-ollama).

---

## 🐳 Docker-Related

### Q13: Docker Container Exits Immediately After Starting?

**Solution**:
1. Check container logs:
   ```bash
   docker logs <container_id>
   ```
2. Common causes:
   - Environment variables not configured correctly
   - `.env` file format errors (e.g., extra spaces)
   - Package version conflicts

---

### Q14: API Service in Docker Cannot Access?

**Solution**:
1. Ensure startup command includes `--host 0.0.0.0` (cannot be 127.0.0.1)
2. Check port mapping is correct:
   ```yaml
   ports:
     - "8000:8000"
   ```

---
