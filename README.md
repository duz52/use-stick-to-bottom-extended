# `use-stick-to-bottom-extended`

> Designed with AI chat bots in mind

This package is a fork of the original [`use-stick-to-bottom`](https://github.com/stackblitz-labs/use-stick-to-bottom). The original implementation and copyright belong to StackBlitz/Sam Denty. This extended fork is maintained by [duz52](https://github.com/duz52/).

This fork adds `resize={false}` / `resize: false`, which disables ResizeObserver-driven automatic scrolling after initial layout. Use it when a chat should scroll once after sending a new message only if the user was already at the bottom, without continuing to stick to the bottom while the window is being resized.

A lightweight **zero-dependency** React hook + Component that automatically sticks to the bottom of container and smoothly animates the content to keep it's visual position on screen whilst new content is being added.

## Features

- Does not require [`overflow-anchor`](https://developer.mozilla.org/en-US/docs/Web/CSS/overflow-anchor) browser-level CSS support which Safari does not support.
- Can be connected up to any existing component using a hook with refs. Or simply use the provided component, which handles the refs for you plus provides context - so child components can check `isAtBottom` & programmatically scroll to the bottom.
- Uses the modern, yet well-supported, [`ResizeObserver`](https://developer.mozilla.org/en-US/docs/Web/API/ResizeObserver) API to detect when content resizes.
  - Supports content shrinking without losing stickiness - not just getting taller.
- Correctly handles [Scroll Anchoring](https://developer.mozilla.org/en-US/docs/Web/CSS/overflow-anchor/Guide_to_scroll_anchoring). This is where when content above the viewport resizes, it doesn't cause the content currently displayed in viewport to jump up or down.
- Allows the user to cancel the stickiness at any time by scrolling up.
  - Clever logic distinguishes the user scrolling from the custom animation scroll events (without doing any debouncing which could cause some events to be missed).
  - Mobile devices work well with this logic too.
- Uses a custom implemented smooth scrolling algorithm, featuring velocity-based spring animations (with configurable parameters).
  - Other libraries use easing functions with durations instead, but these doesn't work well when you want to stream in new content with variable sizing - which is common for AI chatbot use cases.
  - `scrollToBottom` returns a `Promise<boolean>` which will resolve to `true` as soon as the scroll was successful, or `false` if the scroll was cancelled.

## Installation

```bash
npm install github:duz52/use-stick-to-bottom-extended
```

## Usage

### `<StickToBottom>` Component

```jsx
import { StickToBottom, useStickToBottomContext } from 'use-stick-to-bottom-extended';

function Chat() {
  return (
    <StickToBottom className="h-[50vh] relative" resize="smooth" initial="smooth">
      <StickToBottom.Content className="flex flex-col gap-4">
        {messages.map((message) => (
          <Message key={message.id} message={message} />
        ))}
      </StickToBottom.Content>

      <ScrollToBottom />

      {/* This component uses `useStickToBottomContext` to scroll to bottom when the user enters a message */}
      <ChatBox />
    </StickToBottom>
  );
}

function ScrollToBottom() {
  const { isAtBottom, scrollToBottom } = useStickToBottomContext();

  return (
    !isAtBottom && (
      <button
        className="absolute i-ph-arrow-circle-down-fill text-4xl rounded-lg left-[50%] translate-x-[-50%] bottom-0"
        onClick={() => scrollToBottom()}
      />
    )
  );
}
```

### One-shot send scroll without resize stickiness

Set `resize={false}` to disable automatic scrolling from content resize events after the initial layout. Capture whether the user was at the bottom before appending the message, then scroll once after the message is rendered.

```jsx
import { useLayoutEffect, useRef, useState } from 'react';
import { StickToBottom, useStickToBottomContext } from 'use-stick-to-bottom-extended';

function Chat() {
  const [messages, setMessages] = useState([]);

  return (
    <StickToBottom className="h-[50vh] relative" resize={false} initial="instant">
      <ChatContent messages={messages} setMessages={setMessages} />
    </StickToBottom>
  );
}

function ChatContent({ messages, setMessages }) {
  const { isAtBottom, scrollToBottom } = useStickToBottomContext();
  const shouldScrollAfterSend = useRef(false);

  const sendMessage = (text) => {
    shouldScrollAfterSend.current = isAtBottom;
    setMessages((current) => [...current, { id: crypto.randomUUID(), text }]);
  };

  useLayoutEffect(() => {
    if (!shouldScrollAfterSend.current) {
      return;
    }

    shouldScrollAfterSend.current = false;
    scrollToBottom({ animation: 'instant' });
  }, [messages.length, scrollToBottom]);

  return (
    <>
      <StickToBottom.Content className="flex flex-col gap-4">
        {messages.map((message) => (
          <Message key={message.id} message={message} />
        ))}
      </StickToBottom.Content>

      <ChatBox onSend={sendMessage} />
    </>
  );
}
```

### `useStickToBottom` Hook

```jsx
import { useStickToBottom } from 'use-stick-to-bottom-extended';

function Component() {
  const { scrollRef, contentRef } = useStickToBottom({ resize: false });

  return (
    <div style={{ overflow: 'auto' }} ref={scrollRef}>
      <div ref={contentRef}>
        {messages.map((message) => (
          <Message key={message.id} message={message} />
        ))}
      </div>
    </div>
  );
}
```

## Extended API

### `resize: false`

The original package accepts `resize` as an animation setting, such as `"smooth"`, `"instant"`, or a spring animation object. This fork also accepts `false`.

When `resize` is `false`, the hook still observes content size changes so scroll state stays accurate, but it does not automatically call `scrollToBottom` for positive content resizes after the first layout. That lets applications implement explicit, one-shot scroll behavior on send while avoiding continued bottom locking during window resize.
