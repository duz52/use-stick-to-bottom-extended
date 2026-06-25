# `use-stick-to-bottom-extended`

> Designed with AI chat bots in mind

This package is a fork of the original [`use-stick-to-bottom`](https://github.com/stackblitz-labs/use-stick-to-bottom). The original implementation and copyright belong to StackBlitz/Sam Denty. This extended fork is maintained by [duz52](https://github.com/duz52/).

This fork adds `append`, `appendPreserveScrollPosition`, and `resize={false}` / `resize: false`, allowing chat UIs to scroll once when a new message element is appended while avoiding sticky scrolling during streaming content growth or window resize.

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
npm install use-stick-to-bottom-extended
```

## Usage

### `<StickToBottom>` Component

```jsx
import { StickToBottom, useStickToBottomContext } from 'use-stick-to-bottom-extended';

function Chat() {
  return (
    <StickToBottom
      className="h-[50vh] relative"
      initial="smooth"
      append="smooth"
      appendPreserveScrollPosition={false}
      resize={false}
    >
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

### Scroll on appended messages only

Set `append` to control scrolling when a new direct child is appended to the content element. Set `resize={false}` to disable automatic scrolling for content height changes that do not append a new direct child, such as streaming text growth, image loading, markdown reflow, or window resize.

```jsx
import { useStickToBottom } from 'use-stick-to-bottom-extended';

function Messages({ messages }) {
  const { scrollRef, contentRef } = useStickToBottom({
    initial: 'smooth',
    append: 'smooth',
    appendPreserveScrollPosition: false,
    resize: false,
  });

  return (
    <div ref={scrollRef} style={{ overflow: 'auto' }}>
      <div ref={contentRef}>
        {messages.map((message) => (
          <Message key={message.id} message={message} />
        ))}
      </div>
    </div>
  );
}
```

### `useStickToBottom` Hook

```jsx
import { useStickToBottom } from 'use-stick-to-bottom-extended';

function Component() {
  const { scrollRef, contentRef } = useStickToBottom({
    append: 'smooth',
    appendPreserveScrollPosition: false,
    resize: false,
  });

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

### `append`

The original package uses `resize` for every positive content resize after initial layout. This fork adds `append` for the narrower case where the content element's direct child count increases.

If `append` is omitted, it falls back to `resize`, preserving the original behavior. If `append` is set and `resize` is `false`, a chat can scroll once when a new message element is appended without continuing to stick while that message streams or reflows.

### `appendPreserveScrollPosition`

By default, automatic append scrolling preserves the user's scroll position and only scrolls when the user is already at the bottom.

Set `appendPreserveScrollPosition: false` when a new appended child should always trigger one scroll-to-bottom, even if the user is currently reading older content. This only affects append events; non-append content resizes are still controlled by `resize`.

### `resize: false`

The original package accepts `resize` as an animation setting, such as `"smooth"`, `"instant"`, or a spring animation object. This fork also accepts `false`.

When `resize` is `false`, the hook still observes content size changes so scroll state stays accurate, but it does not automatically call `scrollToBottom` for positive content resizes after the first layout.
