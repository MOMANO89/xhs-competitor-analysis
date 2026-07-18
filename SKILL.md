---
name: xhs-competitor-analysis
description: >
  小红书竞品账号深度分析技能。当用户想要分析竞品或指定品牌/App 在小红书上的官方账号运营情况时使用。
    触发词包括：分析小红书账号、竞品调研、小红书运营分析、看看XXX在小红书怎么做的、
      分析XXX的账号内容/排版/风格、做竞品报告、小红书账号对比等。
        该技能会自动用浏览器浏览小红书，收集账号数据（粉丝数、笔记量、获赞收藏、更新频率等），
          分析内容策略、视觉风格、排版形式，并生成包含对比表格和深度洞察的 Word 文档报告（.docx）
            和 HTML 版报告。如果用户提到"竞品"、"对标账号"、"看看别人怎么做"、"调研一下"之类的话，
              请主动使用本技能。
              ---

              # 小红书竞品账号分析

              本技能指导你系统地分析多个品牌/App 在小红书上的官方账号，并生成专业的竞品分析报告。

              ## 准备阶段

              ### 1. 明确分析对象

              先向用户确认（如对话中已明确则跳过）：
              - 要分析哪些品牌/App 的账号？（列出名称）
              - 期望输出格式？（默认：Word 文档 + HTML 报告）
              - 有无特别关注的维度？（默认：内容策略、视觉风格、运营数据、爆款笔记）

              ### 2. 安装依赖

              在 bash 沙盒中，提前安装 docx 包（用于生成 Word 文档）：

              ```bash
              cd /sessions/sleepy-wizardly-wright/mnt/outputs && npm install docx
              ```

              如果已安装（`node_modules/docx` 存在）则跳过。

              ---

              ## 数据采集阶段

              使用 **Claude in Chrome MCP**（`mcp__Claude_in_Chrome__*`）浏览小红书。

              ### 关键注意事项

              **URL 安全性**：小红书很多页面 URL 带有 `xsec_token` 参数，会触发 JavaScript 工具的 `[BLOCKED: Cookie/query string data]` 错误。解决办法：始终导航到不带 token 的干净 URL：
              ```
              https://www.xiaohongshu.com/user/profile/<用户ID>
              ```

              **获取用户 ID**：在搜索结果中找到账号后，从页面链接中提取 profile ID（32位十六进制字符串）。

              ### 搜索流程（每个账号）

              1. 导航到小红书搜索：
                 ```
                    https://www.xiaohongshu.com/search_result?keyword=<App名称>&type=user
                       ```

                       2. 用 `get_page_text` 读取搜索结果，找到认证蓝V账号，提取其 profile ID。

                       3. 导航到干净的 profile 页面：
                          ```
                             https://www.xiaohongshu.com/user/profile/<profile_id>
                                ```

                                4. 用 `get_page_text` 收集以下数据：

                                   **基础指标**
                                      - 粉丝数
                                         - 笔记数量
                                            - 获赞与收藏总数
                                               - 最近更新时间
                                                  - IP 属地
                                                     - 小红书号
                                                        - 账号简介全文

                                                           **内容分析**（浏览最近 15-20 篇笔记）
                                                              - 笔记标题和点赞数（找出点赞最高的 3-5 篇）
                                                                 - 内容类型分布（功能更新/科普/活动/节日热点等）
                                                                    - 图文 vs 视频比例
                                                                       - 更新频率

                                                                          **视觉风格**（基于笔记封面图）
                                                                             - 主色调
                                                                                - 排版格式（大字标题/产品截图/插画）
                                                                                   - IP 形象/吉祥物
                                                                                      - 是否有统一视觉角标/水印

                                                                                         **运营特征**
                                                                                            - 是否有双账号矩阵
                                                                                               - 是否有置顶笔记
                                                                                                  - 互动风格（简介引流、评论区等）

                                                                                                  5. 收集帖子封面图 CDN URL（格式为 `https://sns-webpic-qc.xhscdn.com/...`），用于 HTML 报告展示。

                                                                                                  ---

                                                                                                  ## 分析与报告生成阶段

                                                                                                  数据采集完成后，生成两份报告：

                                                                                                  ### HTML 报告（附图版）

                                                                                                  HTML 报告直接引用小红书 CDN 图片 URL，用户在 Mac 浏览器打开时图片会自动加载。

                                                                                                  报告结构：
                                                                                                  - 封面（标题、副标题、日期）
                                                                                                  - 账号概览对比表（所有账号核心数据对比）
                                                                                                  - 每个账号独立分析章节：
                                                                                                    - 基础信息卡片
                                                                                                      - 账号简介
                                                                                                        - 内容方向分析（带典型笔记示例和赞数）
                                                                                                          - 视觉风格说明
                                                                                                            - 运营亮点（3-5条）
                                                                                                              - 代表性笔记封面图展示
                                                                                                              - 综合洞察与借鉴建议
                                                                                                              
                                                                                                              保存到：`/sessions/sleepy-wizardly-wright/mnt/outputs/小红书官方账号分析报告.html`
                                                                                                              
                                                                                                              ### Word 文档（.docx）
                                                                                                              
                                                                                                              使用 Node.js + 本地安装的 `docx` 包生成。
                                                                                                              
                                                                                                              **重要**：bash 沙盒无互联网访问，无法下载图片，因此 Word 文档中图片改用可点击的超链接（账号主页链接）替代。
                                                                                                              
                                                                                                              Word 文档结构：
                                                                                                              1. 封面页（标题、副标题、日期，居中大字）
                                                                                                              2. 一、账号数据概览（全局对比表）
                                                                                                              3. 二/三/四/五、各账号深度分析章节（详细表格 + 运营亮点文字）
                                                                                                              4. 六、核心洞察与借鉴建议（内容/视觉/运营三维度表格）
                                                                                                              
                                                                                                              **使用本地 docx 包**：
                                                                                                              ```javascript
                                                                                                              const { Document, Packer, ... } = require('/sessions/sleepy-wizardly-wright/mnt/outputs/node_modules/docx');
                                                                                                              // 写完后保存：
                                                                                                              Packer.toBuffer(doc).then(buf => {
                                                                                                                fs.writeFileSync('/sessions/sleepy-wizardly-wright/mnt/outputs/小红书官方账号分析报告.docx', buf);
                                                                                                                });
                                                                                                                ```
                                                                                                                
                                                                                                                保存脚本到 outputs 目录并用 `node` 运行。
                                                                                                                
                                                                                                                ---
                                                                                                                
                                                                                                                ## 报告尺寸与格式建议
                                                                                                                
                                                                                                                | 元素 | 推荐值 |
                                                                                                                |------|--------|
                                                                                                                | 页面边距 | 1440 DXA（约 2.54cm）|
                                                                                                                | 标题字号 | H1: 40, H2: 32, H3: 26 |
                                                                                                                | 正文字号 | 22（约 11pt）|
                                                                                                                | 表格列宽 | 标签列: 2800 DXA, 内容列: 6560 DXA |
                                                                                                                | 英文字体 | Arial |
                                                                                                                | 中文字体 | PingFang SC |
                                                                                                                | 页眉 | 报告标题（右对齐，灰色） |
                                                                                                                | 页脚 | 页码（居中） |
                                                                                                                
                                                                                                                ---
                                                                                                                
                                                                                                                ## 典型错误处理
                                                                                                                
                                                                                                                | 错误 | 原因 | 解决方法 |
                                                                                                                |------|------|----------|
                                                                                                                | `[BLOCKED: Cookie/query string data]` | URL 含 xsec_token | 导航到无 token 的干净 profile URL |
                                                                                                                | `[BLOCKED: Base64 encoded data]` | JS 工具屏蔽 base64 输出 | 改用 CDN URL 字符串（不转 base64）|
                                                                                                                | `EAI_AGAIN` (bash fetch) | bash 沙盒无网络 | 不在 bash 下载图片，用 CDN URL 在 HTML 中直接引用 |
                                                                                                                | `SyntaxError: Identifier already declared` | 在同一 tab 重复运行 JS | 换用不同变量名（canvas2, link2 等）|
                                                                                                                | npm 安装权限错误 | 全局安装需 root | 改为本地安装：`cd /outputs && npm install docx` |
                                                                                                                
                                                                                                                ---
                                                                                                                
                                                                                                                ## 最终交付
                                                                                                                
                                                                                                                生成完成后，使用 `mcp__cowork__present_files` 呈现两份文件：
                                                                                                                1. `小红书官方账号分析报告.docx`（主要交付物）
                                                                                                                2. `小红书官方账号分析报告.html`（含实际图片，建议在浏览器打开）
                                                                                                                
                                                                                                                告知用户：HTML 版本需在浏览器中打开才能看到账号封面图，Word 版本中图片以可点击链接替代。
                                                                                                                
