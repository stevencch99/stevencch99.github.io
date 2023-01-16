---
layout: post
title: "使用 Phoenix.LiveComponent 保存狀態的代價"
description: "使用 Phoenix.LiveComponent 保存狀態的代價"
crawlertitle: "使用 Phoenix.LiveComponent 保存狀態的代價"
date: 2023-01-16 21:47:29 +0800
categories: Phoenix
tags: Phoenix LiveView
comments: true
---

最近複習到使用 [LiveComponent](https://hexdocs.pm/phoenix_live_view/Phoenix.LiveComponent.html) 保存狀態的代價，試著重新咀嚼並翻譯紀錄在這。

儘管 [Phoenix.LiveView](https://www.phoenixframework.org/) 內部用來追蹤 stateful component 的機制相當輕量，但是為了追蹤和傳送 assigns 的差異，component 所有的 assigns 是存放在記憶體內的。

所以要避免一股腦地直接將所有 LiveView 的 assigns 直接倒給 LiveComponent。

```elixir
# DON'T do this
<.live_component module={MyComponent} {assigns} />

# Instead pass only the keys that you need:
<.live_component module={MyComponent} user={@user} org={@org} />
```

好消息是，因為 LiveView 和 LiveComponent 處於同一個 process ，因此資料結構在記憶體中共享同樣的位置，以上面程式碼為例，view 和 component 共享同樣的 `@user` 和 `@org` 這兩個 assigns。

另外要避免用 LiveComponent 卻只是單純地抽 DOM 畫面的元件來複用，其實這類型的任務交給 function component ([Phoenix.Component](https://hexdocs.pm/phoenix_live_view/Phoenix.Component.html)) 來處理會更加合適：

```elixir
def my_button(text, click) do
  assigns = %{text: text, click: click}

  ~H"""
  <button class="css-framework-class" phx-click={@click}>
    <%= @text %>
  </button>
  """
end
```

一個好的 LiveComponent 應當用來封裝應用程式的關注點，而非 DOM 的功能性操作。

## 參考
- <https://hexdocs.pm/phoenix_live_view/Phoenix.LiveComponent.html#module-cost-of-stateful-components> 















