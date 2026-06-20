# 🎨 薄荷的画廊 — 配置文档

## 访问地址

`https://poolwar.github.io/mint-gallery/`

## 技术架构

```
浏览器 → GitHub Pages（静态托管）
       → Supabase Storage（云端图片存储）
       → IndexedDB（本地图片缓存）
```

- **托管**：GitHub Pages（`poolwar/mint-gallery` 仓库，`master` 分支自动部署）
- **云端存储**：Supabase Storage（公开 bucket，RLS 策略控制权限）
- **本地缓存**：IndexedDB（`mint-img-cache` 数据库，持久化图片 blob）
- **SDK**：Supabase JS SDK v2（CDN 加载，lazy init 容错）

## Supabase 配置

| 配置项 | 值 |
|--------|-----|
| 项目 URL | `https://hwyuaukabthsoupywtfp.supabase.co` |
| Anon Key | `sb_publishable_Pb97Zn0XW5aPVqn8OWVh5Q_mIjrRYG5` |
| Bucket 名称 | `gallery-photos` |
| Bucket 权限 | 公开（Public） |

### RLS 策略

```sql
-- 公开读取
CREATE POLICY "Public View" ON storage.objects
  FOR SELECT USING (bucket_id = 'gallery-photos');

-- 公开上传
CREATE POLICY "Public Upload" ON storage.objects
  FOR INSERT WITH CHECK (bucket_id = 'gallery-photos');

-- 公开删除
CREATE POLICY "Public Delete" ON storage.objects
  FOR DELETE USING (bucket_id = 'gallery-photos');
```

### Storage 目录结构

```
gallery-photos/
├── embedded/          # 37 张内嵌初始图片
│   ├── _20260531155934_1_491.jpg
│   ├── ...
│   └── *.jpg
└── uploads/           # 用户上传的图片
    ├── <timestamp>_<sanitized_name>.jpg
    └── ...
```

## 文件结构

```
mint-gallery/
├── index.html         # 主文件（~79KB，单文件应用）
├── CONFIG.md          # 本配置文档
├── final-fix.cjs      # 最后一次重建脚本（迁移内嵌图片 + 清理冗余）
├── rebuild-v2.cjs     # 重建脚本 v2
├── rebuild-v3.cjs     # 重建脚本 v3
└── fix-buildhtml.cjs  # buildFullHTML 硬编码清理脚本
```

## 核心功能

| 功能 | 实现方式 |
|------|----------|
| 布局模式 | 瀑布流 / 整齐网格 / 拍立得墙 / 密集拼贴 / 时间画廊 |
| 图片上传 | 拖拽或点击上传 → Supabase Storage → 自动去重 |
| 云端同步 | 🔄 刷新按钮手动拉取 Supabase 最新图片列表 |
| 本地缓存 | IndexedDB 缓存图片 blob，二次访问秒开 |
| 图片删除 | 前端 + Supabase Storage 同步删除 |
| 拼接模式 | 多选图片 → 横向/纵向/网格拼接导出 |
| 保存到文件 | 将当前所有图片序列化为独立 HTML 文件下载 |
| EXIF 提取 | 从上传图片中提取拍摄日期（时间线模式使用） |

## 部署流程

1. 修改 `index.html`
2. `git add index.html && git commit -m "..." && git push`
3. GitHub Pages 自动部署（约 1-2 分钟生效）

## 注意事项

- **Anon Key 是公开的**：Supabase anon key 设计为可公开暴露，安全性由 RLS 策略保证
- **中文文件名处理**：上传时文件名中的非 ASCII 字符会被替换为 `_`，保留原始文件名用于显示
- **文件大小限制**：Supabase 免费 tier 单文件上限 50MB，总存储 1GB，月流量 2GB
- **无身份认证**：所有人可查看、上传、删除所有图片。如需用户隔离，可添加 Supabase Auth
- **内嵌图片不拆分**：37 张原始图片存储在 Supabase `embedded/` 目录，通过 CDN URL 加载
- **IndexedDB 缓存**：图片首次从 Supabase 加载后存入浏览器 IndexedDB，后续从本地 blob URL 加载，不清除浏览器数据则永久保留
