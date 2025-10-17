---
title: "react-portalslots"
subtitle: "Colocating UI components without prop drilling"
tags: ["react", "javascript", "slots", "portals"]
published: false
---

Building React apps often involves a common problem: you need to render parts of your UI (buttons, toolbars, headers, etc.) into specific areas of your layout (`header`, `sidebar`, `footer`), **without drilling props** all the way up through the component tree or relying on global state.

The **[react-portalslots](https://github.com/beautyfree/react-portalslots)** library solves this elegantly.
It lets you "teleport" UI fragments from deep inside your component tree into named layout slots - powered by React Portals and Context - with zero global state and minimal boilerplate.

---

## The Idea Behind react-portalslots

### The Problem: Prop Drilling & "Upward Passing"

When you have a `Layout` component that defines regions like a header or sidebar, nested components often want to add something there (like a "Save" button in the header).

You usually end up doing something like:

```tsx
<Layout
  header={<button>Save</button>}
  sidebar={<div>Navigation</div>}
>
  <Toolbar />
</Layout>
```

This quickly becomes messy - you have to "lift" components up through layers of unrelated components. It breaks **locality** and **composability**.

### The Solution: Named Portals + Slots

`react-portalslots` introduces the concept of *named slots* - pre-defined areas of your layout that can receive content from anywhere in the React tree.

You define "slots" in your layout:

```tsx
<HeaderSlot.Slot />
<SidebarSlot.Slot />
<FooterSlot.Slot />
```

And later, deep inside your components, just render:

```tsx
<HeaderSlot>
  <button>Save</button>
</HeaderSlot>
```

The library automatically "registers" this content and renders it inside the corresponding slot.
This removes the need for prop-drilling, context stores, or global state management.

---

## API Overview

### `PortalSlotsProvider`

Wrap your app (or the section that uses slots) with the provider:

```tsx
<PortalSlotsProvider>
  <App />
</PortalSlotsProvider>
```

This creates a registry that tracks all active slots and portal contents.

---

### `PortalSlot(name?: string)`

Create a named slot with a simple factory call:

```tsx
const HeaderSlot = PortalSlot('header');
const FooterSlot = PortalSlot('footer');
```

Each slot provides two components:

- `<HeaderSlot>` - the **portal component** used anywhere in the tree.
- `<HeaderSlot.Slot>` - the **container** that marks where content should appear.

Here’s a minimal working example:

```tsx
const HeaderSlot = PortalSlot('header');
const FooterSlot = PortalSlot('footer');

function Layout({ children }) {
  return (
    <div>
      <header>
        <HeaderSlot.Slot />
      </header>
      <main>{children}</main>
      <footer>
        <FooterSlot.Slot />
      </footer>
    </div>
  );
}

function App() {
  return (
    <PortalSlotsProvider>
      <Layout>
        <HeaderSlot>
          <button>Save</button>
        </HeaderSlot>

        <FooterSlot>
          <small>© 2025</small>
        </FooterSlot>

        <div>Page content…</div>
      </Layout>
    </PortalSlotsProvider>
  );
}
```

Result: the `<button>` and `<small>` elements render inside the layout’s header and footer automatically.

---

## Real-World Examples


### 1. Layout

```ts
import { PortalSlotsProvider, PortalSlot } from 'react-portalslots';

const HeaderPortal = PortalSlot('header');
const FooterPortal = PortalSlot('footer');

function Layout({ children }: { children: React.ReactNode }) {
  return (
    <div className="page">
      <header className="page-header">
        <HeaderPortal.Slot />
      </header>

      <main className="page-content">{children}</main>

      <footer className="page-footer">
        <FooterPortal.Slot />
      </footer>
    </div>
  );
}

export function App() {
  return (
    <PortalSlotsProvider>
      <Layout>
        {/* These can live anywhere in the tree */}
        <HeaderPortal>
          <button>Save</button>
        </HeaderPortal>
        <FooterPortal>
          <small>© 2025</small>
        </FooterPortal>

        {/* App content */}
        <div>Dashboard</div>
      </Layout>
    </PortalSlotsProvider>
  );
}
```

### 2. Conditional Toolbars

Slots can render multiple fragments and handle conditional UI easily:

```tsx
const ToolbarSlot = PortalSlot('toolbar');

function Layout({ children }) {
  return (
    <div>
      <div className="toolbar-area">
        <ToolbarSlot.Slot />
      </div>
      <main>{children}</main>
    </div>
  );
}

function Editor() {
  const [isEditing, setEditing] = React.useState(false);

  return (
    <div>
      {isEditing && (
        <ToolbarSlot>
          <button onClick={() => setEditing(false)}>Cancel</button>
          <button>Save</button>
        </ToolbarSlot>
      )}

      {!isEditing && (
        <button onClick={() => setEditing(true)}>Edit</button>
      )}
    </div>
  );
}
```

### 3. Multi-Page Layout with React Router

Each page can inject its own header or footer without touching the layout:

```tsx
const HeaderSlot = PortalSlot('header');
const FooterSlot = PortalSlot('footer');

function Layout({ children }) {
  return (
    <div>
      <header><HeaderSlot.Slot /></header>
      <main>{children}</main>
      <footer><FooterSlot.Slot /></footer>
    </div>
  );
}

function HomePage() {
  return (
    <>
      <HeaderSlot><h1>Home</h1></HeaderSlot>
      <p>Welcome home!</p>
    </>
  );
}

function SettingsPage() {
  return (
    <>
      <HeaderSlot><button>Save</button></HeaderSlot>
      <p>Settings page</p>
    </>
  );
}

function App() {
  return (
    <PortalSlotsProvider>
      <BrowserRouter>
        <Layout>
          <Routes>
            <Route path="/" element={<HomePage />} />
            <Route path="/settings" element={<SettingsPage />} />
          </Routes>
        </Layout>
      </BrowserRouter>
    </PortalSlotsProvider>
  );
}
```

Now every route defines its own layout content in place - no prop drilling.

### 4. Error Boundaries and Notifications

Even within an error boundary, slots still render correctly:

```tsx
const NotificationSlot = PortalSlot('notifications');

class ErrorBoundary extends React.Component {
  state = { hasError: false };

  static getDerivedStateFromError() {
    return { hasError: true };
  }

  render() {
    if (this.state.hasError) {
      return (
        <NotificationSlot>
          <div className="error">Something went wrong</div>
        </NotificationSlot>
      );
    }
    return this.props.children;
  }
}

function Layout({ children }) {
  return (
    <div>
      <div className="notifications">
        <NotificationSlot.Slot />
      </div>
      <main>{children}</main>
    </div>
  );
}
```

---

## 💡 When to Use It

- Dashboards and admin panels where child pages modify shared toolbars or headers.
- Multi-page apps with per-page layout customization.
- Component libraries that need flexible "UI injection points".
- Replacing global "UI stores" for layout coordination.

## 🧠 Summary

`react-portalslots` offers a clean, elegant abstraction over React Portals - enabling **named, colocated UI composition** without the pain of prop drilling or global state.

It’s ideal for apps where **layout regions are shared** but **logic lives deep** in the component tree.

> 💬 If your components often need to say "put this button in the header," this library will make your life much easier.

👉 [GitHub: react-portalslots](https://github.com/beautyfree/react-portalslots)

