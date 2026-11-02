# book-proofread

用于中文 DOCX 书稿校对的 Agent Skill。

## 用途

适合待出版书稿的文字校对，也支持读取已有报告后继续校对。

主要检查以下内容：

- **严重语病**：句意不通、句子结构明显有问题，或会造成实质性误解的表达。轻微不顺、赘述和单纯润色建议不标出。
- **错别字、漏字、多字**：能够从原文和上下文确定的文字错误。
- **标点符号**：中文叙述中的标点用法、配对等问题，不把中文标点规则套到代码和命令上。
- **编号层次**：标题与条目中的跳号、重号、层次及格式问题。顺序关系的子项用 `（1）（2）（3）`，并列关系的子项用圆点子弹头。
- **数字用法**：按语境区分汉字数字和阿拉伯数字，不要求正文数字统一改成汉字。定型词语、相邻数字表示的概数、带“几”的约数，以及干支纪年、月份、传统节日等按相应习惯处理。
- **图文引用**：核对引用文字、图号、图题与实际插图是否对应。

本技能不核验技术原理、专业事实、标准条款、代码配置或实验结果，也不判断插图内部内容是否正确。报告只记录确定的问题，不设置“待核实”项。

完整规则见 [SKILL.md](SKILL.md)。

## 安装

仓库地址：[gmjneko/book-proofread](https://github.com/gmjneko/book-proofread)。技能入口 `SKILL.md` 位于仓库根目录。

**优先按照当前 Agent 自己的 Skill 安装规范安装。** 如果 Agent 没有自己的规范，就放到用户文件夹下的 `.agents/skills` 目录，即 `~/.agents/skills/book-proofread/`。

安装时保留以下文件及目录结构，以文件夹为整体复制，不要只复制 `SKILL.md`：

```text
book-proofread/
├── SKILL.md
├── agents/
│   └── openai.yaml
└── references/
    └── advanced-network-book.md
```

`agents/openai.yaml` 提供技能的展示信息；`references/` 存放配套读取说明。书稿和生成的校对报告不属于技能安装文件。

安装完成后，按当前 Agent 的方式加载或调用 `book-proofread`。

## DOCX 技能依赖

本技能规定校对流程和判断标准，DOCX 的读取、内容提取和页面查看由独立的 DOCX 技能完成。

开始校对前，Agent 应先检查是否存在可用的 DOCX 技能：

- 用户指定了技能时，优先读取并使用用户指定的技能。
- 用户未指定时，选择当前环境中可用的技能，例如 `docx` 或支持 DOCX 的 `documents`。
- 没有可用技能时，先告知用户需要安装，不能绕过这项依赖开始校对。例如可以安装 [Anthropic DOCX Skill](https://github.com/anthropics/skills/blob/main/skills/docx/SKILL.md)，并按照该技能的说明准备运行环境。

## 使用方式

安装并加载后，在支持斜杠技能命令的 Agent 中，可以显式调用：

```text
/book-proofread 校对《高级网络技术.docx》，只检查项目1，报告写入书稿目录的 proofread.md
```

命令后面的内容就是本次任务要求，可以指定文件、范围、输出位置，或要求继续上次的校对。Agent 收到显式调用后应直接使用本技能，无需再询问是否启用。

斜杠命令是否可用取决于宿主 Agent 的安装和命令机制，仅修改 `SKILL.md` 不会为所有 Agent 自动注册命令。桌面端的已启用技能可出现在斜杠菜单中；应从菜单选择对应技能。参见 [官方斜杠命令说明](https://learn.chatgpt.com/docs/reference/slash-commands)。

Codex 也可以使用 `$book-proofread` 显式调用：

```text
$book-proofread 校对《高级网络技术.docx》，只检查项目1
```

这一写法适用于 Codex 的显式技能调用，见 [官方技能说明](https://learn.chatgpt.com/docs/build-skills)。其他 Agent 使用各自的原生技能入口。

向 Agent 提供书稿路径，并说明使用本技能。例如：

```text
使用 book-proofread 技能校对《高级网络技术.docx》。
先检查并读取可用的 DOCX 技能，再按原文顺序逐小节校对。
每完成一小节，把确定的问题写入书稿所在目录的 proofread.md。
只检查技能约定的文字编辑问题，不修改原 DOCX。
```

也可以指定校对范围和输出位置，例如只校对某一章。继续校对时，要求 Agent 先读取已有的 `proofread.md` 和进度记录，避免覆盖或重复记录。

技能适用于其他中文 DOCX 书稿。`references/advanced-network-book.md` 是《高级网络技术》的专属读取说明，换用其他书稿时应重新了解其结构。

## 输出格式

默认在书稿所在目录生成 `proofread.md`，按书名和原文标题组织。每个问题使用以下格式：

```markdown
# 书名
## 章节名
### 下一级标题的名

问题：说明具体问题。

> 这里是原文的那一句话

错误类型：错别字

修改方式：给出明确的修改建议或改后的完整句子。

---
```

没有下级标题时省略对应标题；有更深层级时保留完整标题路径。同一小节有多个问题时逐条记录，每条之间用 `---` 分隔。没有问题的小节不生成空问题条目。
