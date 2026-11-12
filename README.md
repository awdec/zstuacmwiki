# ZSTU ACM 集训队官方网站

浙江理工大学 ACM 集训队官方网站，基于 [VitePress](https://vitepress.dev/) 构建。

本站用于介绍集训队、ACM 系列赛事、训练与参赛安排，并整理历年 ICPC、CCPC、ZJCPC、天梯赛、蓝桥杯、百度之星等竞赛的荣誉战绩。

- 官网：<https://zstuacm.top/>
- 更详细的近年 ICPC/CCPC 区域赛数据：[XCPCAtlas](https://awdec.github.io/XCPCAtlas/#/)
- 集训队 QQ 群：761713905

## 内容板块

- **首页**：集训队简介、目标与联系方式
- **ACM**：ACM/ICPC 介绍、赛季时间安排、学习资料
- **集训队**：集训队简介、管理条例、教练、比赛报名与参赛相关
- **荣誉殿堂**：ICPC、CCPC、ZJCPC、天梯赛、蓝桥杯、百度之星历年战绩

## 技术栈

- VitePress 1.6
- Markdown + MathJax
- Vue 3 自定义主题与组件
- GitHub Pages 自动部署

## 本地开发

需要 Node.js 18 及以上版本。

```bash
npm install
npm run docs:dev
```

默认访问地址：<http://localhost:5173/>。

构建与预览：

```bash
npm run docs:build
npm run docs:preview
```

## 目录结构

```text
docs/
├── .vitepress/       # 站点配置、主题与样式
├── ACM/              # ACM 赛事介绍、赛程与学习资料
├── group/            # 集训队介绍、管理条例、报名与参赛
├── honor/            # 历年竞赛荣誉战绩
└── public/           # 图片等静态资源
```

## 部署

推送到 `master` 分支后，GitHub Actions 会自动构建并部署到 GitHub Pages。站点域名通过根目录的 `CNAME` 配置为 `zstuacm.top`。

## 数据说明

ICPC/CCPC 等赛事数据来自公开赛事榜单与 [XCPCAtlas](https://awdec.github.io/XCPCAtlas/#/)。部分历史比赛存在资料缺失，页面中已作标注；更完整的近年区域赛数据可在 XCPCAtlas 查看。

## 贡献

欢迎集训队队员补充和校对赛事数据、训练资料与队伍信息。提交前请确认内容准确，并保持现有 Markdown 表格与标题格式。
