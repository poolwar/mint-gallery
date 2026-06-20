# 🎨 薄荷的画廊

一个共享的艺术画廊，支持上传、浏览和拼接照片。

## 功能

- 📤 上传照片（支持拖拽、多选）
- 🖼️ 多种布局：瀑布流、网格、拍立得、拼贴、时间线
- 🧩 拼接模式：将多张照片拼接成一幅作品
- 🔍 灯箱查看：点击照片全屏浏览
- 📅 EXIF 日期提取与时间线展示
- ☁️ 云端存储：上传的照片所有人可见

## 部署

1. 将此仓库部署到 GitHub Pages
2. 配置 Supabase 项目用于云端存储
3. 修改 `index.html` 中的 `SUPABASE_URL` 和 `SUPABASE_ANON_KEY`

## 技术栈

- 纯 HTML/CSS/JS（无框架）
- Supabase Storage（云端图片存储）
- GitHub Pages（静态托管）
