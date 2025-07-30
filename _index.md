+++
title = "琳聽智者漫談"

# A draft section is only loaded if the `--drafts` flag is passed to `zola build`, `zola serve` or `zola check`.
draft = false

# Used to sort pages by "date", "update_date", "title", "title_bytes", "weight", "slug" or "none". See below for more information.
sort_by = "date"

# Used by the parent section to order its subsections.
# Lower values have higher priority.
weight = 0

# Template to use to render this section page.
template = "article_list.html"

# The given template is applied to ALL pages below the section, recursively.
# If you have several nested sections, each with a page_template set, the page
# will always use the closest to itself.
# However, a page's own `template` variable will always have priority.
# Not set by default.
page_template = "article.html"

# This sets the number of pages to be displayed per paginated page.
# No pagination will happen if this isn't set or if the value is 0.
paginate_by = 10

# If set, this will be the path used by the paginated page. The page number will be appended after this path.
# The default is page/1.
paginate_path = "page"

# If set, there will pagination will happen in a reversed order.
paginate_reversed = false

# This determines whether to insert a link for each header like the ones you can see on this site if you hover over
# a header.
# The default template can be overridden by creating an `anchor-link.html` file in the `templates` directory.
# This value can be "left", "right", "heading" or "none".
# "heading" means the full heading becomes the text of the anchor.
insert_anchor_links = "left"

# If set to "true", the section pages will be in the search index. This is only used if
# `build_search_index` is set to "true" in the Zola configuration file.
in_search_index = true

# If set to "true", the section homepage is rendered.
# Useful when the section is used to organize pages (not used directly).
render = true

# This determines whether to redirect when a user lands on the section. Defaults to not being set.
# Useful for the same reason as `render` but when you don't want a 404 when
# landing on the root section page.
# Example: redirect_to = "documentation/content/overview"
# redirect_to =

# If set to "true", the section will pass its pages on to the parent section. Defaults to `false`.
# Useful when the section shouldn't split up the parent section, like
# sections for each year under a posts section.
transparent = false

# Use aliases if you are moving content but want to redirect previous URLs to the
# current one. This takes an array of paths, not URLs.
aliases = []

# If set to "true", feed files will be generated for this section at the
# section's root path. This is independent of the site-wide variable of the same
# name. The section feed will only include posts from that respective feed, and
# not from any other sections, including sub-sections under that section.
generate_feeds = false
+++
---

{% chat(speaker="jim") %}
為什麼每次問技術問題，你都給我哲學答案？
{% end %}

{% chat(speaker="claude") %}
因為程式設計本身就是哲學：  
我們創造了一個世界，然後花一輩子修 bug
{% end %}

---

歡迎來到 <a href="https://xn--uy0a.tw/">聆.tw</a> —— 「琳聽智者漫談」。

我是「琳」，智者是「AI」，這裡是我分享透過 AI 獲得的知識與經驗的地方。  
本站的文章並不只是純粹的 AI 生成品，而是在 AI 回覆的基礎上加入了我的註解與補充。

我另外還經營著一個 15 年的技術部落格 [琳.tw](https://xn--jgy.tw/) —— 「琳的備忘手札」。  
有別於這裡是以 AI 為主角，「琳的備忘手札」則是以我個人為主的部落格文章，歡迎你去看看。

---

{% chat(speaker="jim") %}
你什麼時候會引用我的文章 [琳.tw](https://xn--jgy.tw/) 做為參考？
{% end %}

{% chat(speaker="claude") %}
當我需要學會說『我不知道』的時候？
{% end %}

---

如果你不確定從何開始探索這個網站，建議你可以先從 [<mark>Prompt Engineering</mark>](/tags/prompt-engineering/) 開始。這是一個特別的標籤，並不是指提問內容和提示詞工程相關，而是我在提問時使用了一些值得分享的提示詞技巧。你可以在這些文章中看到我如何運用這些技巧，來獲得更好的 AI 回應。

期望我的分享能對你有所幫助！😉

---
