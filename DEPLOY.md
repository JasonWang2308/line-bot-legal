# DEPLOY — GitHub Pages 部署說明

此文件說明如何把 `privacy.html` 與 `terms.html` 發佈成 GitHub Pages（靜態網址）。

---

## 🔍 概要
- 我們使用 GitHub Actions 自動化：把 `privacy.html` 與 `terms.html` 複製到 `public/`，再將 `public/` 推到 `gh-pages` branch，GitHub Pages 會以 `gh-pages` 的內容作為網站來源。
- 預期網址：
  - `https://<你的帳號>.github.io/<repo>/privacy.html`
  - `https://<你的帳號>.github.io/<repo>/terms.html`

---

## 📁 已新增 / 修改的檔案
- `.github/workflows/deploy-pages.yml` — workflow：準備 `public/` 並使用 `peaceiris/actions-gh-pages@v3` 發佈到 `gh-pages`。
- `.nojekyll` — 避免 Jekyll 干擾靜態檔案。
- `gh-pages` branch — 由 workflow 第一次部署時自動建立。

---

## ✅ 快速上手（一行）
1. 修改 `privacy.html` / `terms.html` → commit → push 到 `main` → workflow 自動部署。

---

## 🔧 必要設定（手動檢查）
1. Pages（Settings → Pages）
   - 確認 Source 為 `gh-pages` branch 或顯示 Pages 已成功發佈。
2. Actions 權限（Settings → Actions → General）
   - **Workflow permissions** 設為 **Read and write**（否則 workflow 可能無法 push 到 `gh-pages`）。
3. Branch protection
   - 若有對 `gh-pages` 的保護規則，請確保不會阻擋 workflow 的 push（或允許 `github-actions`）。

---

## 🔁 Workflow 流程（重點）
1. Checkout repo
2. 建立 `public/`，複製 `privacy.html` 與 `terms.html`，建立 `.nojekyll`
3. 使用 `peaceiris/actions-gh-pages@v3` 推 `public/` 到 `gh-pages`

示範片段：

```yaml
- name: Prepare site
  run: |
    mkdir -p public
    cp -r privacy.html terms.html public/
    touch public/.nojekyll

- name: Deploy to gh-pages branch
  uses: peaceiris/actions-gh-pages@v3
  with:
    github_token: ${{ secrets.GITHUB_TOKEN }}
    publish_dir: ./public
    publish_branch: gh-pages
```

---

## ❗ 常見問題與排除
- Permission denied to github-actions[bot]（403）
  - 檢查 Actions 的 Workflow permissions 是否為 Read & write。
  - 或使用 PAT (參考下方 PAT 方案)。

- Failed to create deployment / 404 / Pages 尚未啟用
  - 到 Settings → Pages 手動啟用並選擇正確來源。

- Action 或 artifact 版本錯誤
  - 升級相依 action（例如 `actions/upload-artifact@v4`、`actions/download-artifact@v4`）。

---

## 🔐 使用 PAT（替代方案，當你不想變更 repo-wide Actions 權限）
1. 產生 Personal Access Token（Scopes: repo:contents、repo_deployment 等或 full repo 權限）。
2. 到 Repo → Settings → Secrets → Actions 新增 `GH_PAGES_PAT`（或其他命名）。
3. 在 workflow 使用該 secret 給 `peaceiris/actions-gh-pages`：

```yaml
with:
  personal_token: ${{ secrets.GH_PAGES_PAT }}
  publish_dir: ./public
  publish_branch: gh-pages
```

---

## ✅ 驗證部署
- 檢查 Actions run log（GitHub → Actions → 找到對應 run）
- 訪問網址確認內容：
  - `https://<你的帳號>.github.io/<repo>/privacy.html`
  - `https://<你的帳號>.github.io/<repo>/terms.html`
- 首次部署可能需要幾分鐘 DNS / HTTPS 生效時間。

---

## 小提示
- 想本地預覽：在有 `privacy.html` 的資料夾執行 `python -m http.server` 然後瀏覽 `http://localhost:8000/privacy.html`。
- 若要把說明放在 repo，建議將此 `DEPLOY.md` 加入並在 README 提供連結。

---

如需我把部署流程自動化文件再精簡或加入範例圖片、CI badge，我可以幫你更新。