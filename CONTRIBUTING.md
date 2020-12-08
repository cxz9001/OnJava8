## 如何貢獻項目

領取或建立新的 [Issue](https://github.com/lingcoder/OnJava8/issues)，如 [issue 14](https://github.com/lingcoder/OnJava8/issues/14) ，在編輯欄的右側添加自己為 `Assignee`。

在 [GitHub](https://github.com/lingcoder/OnJava8/fork) 上 `fork` 項目到自己的倉庫，你的如 `user_name/OnJava8`，然後 `clone` 到本機，並設定使用者訊息。

```bash
$ git clone git@github.com:user_name/OnJava8.git

$ cd OnJava8
```

修改程式碼後提交，並推送到自己的倉庫，注意修改提交消息為對應 Issue 號和描述。

```bash
# 更新內容

$ git commit -a -s

# 在提交對話框裡添加類似如下內容 "Fix issue #14: 描述你的修改內容"

$ git push
```

在 [GitHub](https://github.com/lingcoder/OnJava8/pulls) 上提交 `Pull Request`，添加標籤，並邀請維護者進行 `Review`。

定期使用源項目倉庫內容同步更新自己倉庫的內容。

```bash
$ git remote add upstream https://github.com/lingcoder/OnJava8

$ git fetch upstream

$ git rebase upstream/master

$ git push -f origin master
```

## 影片示範教學

- https://www.bilibili.com/video/av58040840

## 排版規範

本開源書籍排版布局和翻譯風格上參考**阮一峰**老師的 [中文技術文件的寫作規範](https://github.com/ruanyf/document-style-guide)
