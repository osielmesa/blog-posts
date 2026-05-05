---
title: "One Prompt, One Design, One App: Claude Code + Figma MCP"
datePublished: 2026-05-05T15:04:44.717Z
cuid: cmosrex0p00d42flrfg3l29ui
slug: one-prompt-one-design-one-app-claude-code-figma-mcp
tags: ai, react-native, figma, ai-tool, claude, mcp

---

## What Is This About?

If you've been using Claude Code to build UIs, you've probably wondered: *what if Claude could see my Figma designs directly?*

That's exactly what the **Figma MCP server** enables. MCP (Model Context Protocol) acts as a bridge between Claude and Figma, giving Claude structured access to your design files — components, variables, layouts, and more — so it can generate code that actually matches your designs.

In this guide, you'll learn two ways to set it up, when to use each one, and how to get the most out of the workflow.

* * *

## What Can You Do With It?

Once connected, Claude can:

*   **Read your Figma designs** — components, variables, layout data, auto layout, and design tokens
    
*   **Generate code from selected frames** — useful when building new features from a design spec
    
*   **Work with your design system** — align generated code with your real components via Code Connect
    
*   **Capture live UIs and paste them into Figma** — turn a running app screen into an editable Figma frame
    
*   **Write directly to the Figma canvas** — create frames, components, and variables from Claude (remote server, currently in beta)
    

* * *

## Prerequisites

Before starting, make sure you have:

*   **Claude Code** installed: `npm install -g @anthropic-ai/claude-code`
    
*   **Figma Desktop App** (the web version is not compatible with the desktop MCP server)
    
*   A **Claude Pro or Max** account
    

> **Recommendation:** Use the official plugin (Path 1) — it has the broadest set of features and the simplest setup. The open-source alternative is useful if you need a scriptable or CI-friendly approach.

* * *

## Two Setup Paths

There are two ways to connect Claude to Figma. Pick the one that fits your situation:

|  | Remote Plugin (Official) | Open-Source (figma-developer-mcp) |
| --- | --- | --- |
| Setup | Plugin install | npm + API key |
| Features | Full (write to canvas, code gen, live capture) | Read-only (code gen from files) |
| Requires desktop app | No | No |
| Best for | Most users | CI/scripting |

* * *

## Path 1: Official Figma Plugin (Recommended)

This is the simplest and most feature-complete option.

### Step 1: Install the Figma plugin for Claude Code

In your terminal, run:

```bash
claude plugin install figma@claude-plugins-official
```

Press Enter and restart Claude Code.

### Step 2: Authenticate

Once Claude Code restarts:

1.  Type `/plugin` and press Enter to open the Plugin marketplace
    
2.  Navigate to the **Installed** tab
    
3.  Position in the `figma` entry — this opens a submenu with options like Uninstall, Auth, etc.
    
4.  Select **Auth** (not the parent option — that one shows management actions)
    
5.  A browser page will open → click **Allow Access**
    
6.  Return to your terminal and verify the server shows as **connected**
    

That's it. You're ready to use it.

### Step 3: Verify

You can run `/mcp` in Claude to see your connected MCP servers and confirm Figma is listed.

* * *

## Path 2: Desktop MCP Server

If you prefer to run the server locally through the Figma desktop app:

### Step 1: Enable the server in Figma

1.  Open the **Figma Desktop App** and update it to the latest version
    
2.  Open any design file
    
3.  Switch to **Dev Mode** (with nothing selected)
    
4.  In the right sidebar, find the **MCP server toggle** and enable it
    
5.  You'll see the local address: `http://127.0.0.1:3845/mcp`
    

### Step 2: Register with Claude Code

```bash
claude mcp add --transport http figma-desktop http://127.0.0.1:3845/mcp
```

Then restart Claude Code.

### Step 3: Verify

```bash
claude mcp list
```

You should see `figma-desktop` in the list.

* * *

## Path 3: Open-Source Alternative

The community-maintained `figma-developer-mcp` package is a solid alternative, especially for scripting or CI environments.

### Step 1: Install the package

```bash
npm install -g figma-developer-mcp
```

### Step 2: Generate a Figma API key

1.  Go to **Figma Account Settings** → **Personal Access Tokens**
    
2.  Click **Generate new token**
    
3.  Copy the token
    

### Step 3: Create the config file

In your project root, create a `.mcp.json` file:

```json
{
  "mcpServers": {
    "figma-developer-mcp": {
      "command": "figma-developer-mcp",
      "args": ["--stdio"],
      "env": {
        "FIGMA_API_KEY": "YOUR_FIGMA_API_KEY_HERE"
      }
    }
  }
}
```

### Step 4: Verify

Open Claude Code in your project folder, type `/mcp`, and confirm the server is listed and connected.

* * *

## How to Use It: One Prompt, App + Design

This is where it gets interesting. With Claude connected to Figma, you can go from a single prompt to a working **React Native + Expo app** and a **Figma file with the full UI** — at the same time.

### The demo: a Habit Tracker app

Let's say you want to build a simple habit tracker. You open Claude Code (with the Figma MCP connected) and type:

```plaintext
I want to build a React Native Expo app called "HabitLoop" — a minimal daily habit tracker.

Step 1 — Design in Figma first:
Create a new Figma file called "HabitLoop" with two frames:
- Home screen: a list of 3 sample habits, each with an emoji icon, name, streak count, and a checkbox. One habit should appear as completed (green checkmark, strikethrough name). Include a progress bar and a floating "+" button at the bottom right.
- Add Habit modal: a bottom sheet with a text input, a horizontal emoji picker row, a Cancel button, and a Save button in indigo.
Use auto layout, 16px margins, 12px gaps, white background, indigo (#6366f1) as the accent color, and rounded cards (12px radius) with subtle shadows. Name all layers clearly.

Step 2 — Generate the app from the design:
Once the Figma frames are ready, use them as the reference to scaffold a React Native Expo app with TypeScript. Generate:
- app/index.tsx → Home screen
- components/HabitCard.tsx → card component matching the Figma card frame
- components/AddHabitModal.tsx → bottom sheet modal with text input and emoji picker
- hooks/useHabits.ts → state logic (add habit, toggle done, streak counter)
- constants/theme.ts → colors, spacing, radius, typography, and shadow tokens extracted from the Figma design

No third-party UI libraries. Functional components with hooks only.
```

That's it. One prompt. Here's what Claude does:

**On the code side**, it scaffolds the Expo project and generates the screens:

```plaintext
HabitLoop/
├── App.tsx
├── index.ts
├── app/
│   └── index.tsx           ← Home screen with habit list and progress bar
├── components/
│   ├── HabitCard.tsx       ← Card with emoji icon, name, streak, checkbox
│   └── AddHabitModal.tsx   ← Bottom sheet with text input and emoji picker
├── hooks/
│   └── useHabits.ts        ← State logic (add habit, toggle done, streak counter)
├── constants/
│   └── theme.ts            ← Colors, spacing, radius, typography, shadow tokens
├── assets/
└── app.json
```

Here's what `HabitCard.tsx` looks like:

```tsx
import React from 'react';
import { View, Text, TouchableOpacity, StyleSheet } from 'react-native';
import { Habit } from '../hooks/useHabits';
import { colors, spacing, radius, typography, shadow } from '../constants/theme';

interface HabitCardProps {
  habit: Habit;
  onToggle: (id: string) => void;
}

export default function HabitCard({ habit, onToggle }: HabitCardProps) {
  const done = habit.completedToday;

  return (
    <TouchableOpacity
      activeOpacity={0.8}
      onPress={() => onToggle(habit.id)}
      style={[styles.card, done && styles.cardDone]}
    >
      <View style={[styles.emojiCircle, done && styles.emojiCircleDone]}>
        <Text style={styles.emojiText}>{habit.emoji}</Text>
      </View>

      <View style={styles.textCol}>
        <Text style={[styles.habitName, done && styles.habitNameDone]} numberOfLines={1}>
          {habit.name}
        </Text>
        <View style={styles.streakRow}>
          <Text style={styles.fireEmoji}>🔥</Text>
          <Text style={[styles.streakText, done && styles.streakTextDone]}>
            {habit.streak} day streak
          </Text>
        </View>
      </View>

      <View style={[styles.checkbox, done && styles.checkboxDone]}>
        {done && <Text style={styles.checkmark}>✓</Text>}
      </View>
    </TouchableOpacity>
  );
}

const styles = StyleSheet.create({
  card: {
    flexDirection: 'row',
    alignItems: 'center',
    backgroundColor: colors.card,
    borderRadius: radius.lg,
    padding: spacing.base,
    gap: spacing.md,
    ...shadow.card,
  },
  cardDone: { backgroundColor: colors.successLight },
  emojiCircle: {
    width: 40,
    height: 40,
    borderRadius: 20,
    backgroundColor: colors.indigoLight,
    alignItems: 'center',
    justifyContent: 'center',
  },
  emojiCircleDone: { backgroundColor: colors.successLight },
  emojiText: { fontSize: 20 },
  textCol: { flex: 1, gap: spacing.xs },
  habitName: { ...typography.body, color: colors.gray900 },
  habitNameDone: { color: colors.successText, textDecorationLine: 'line-through' },
  streakRow: { flexDirection: 'row', alignItems: 'center', gap: spacing.xs },
  fireEmoji: { fontSize: 12 },
  streakText: { ...typography.caption, color: colors.gray500 },
  streakTextDone: { color: colors.successText },
  checkbox: {
    width: 24,
    height: 24,
    borderRadius: radius.sm,
    borderWidth: 1.5,
    borderColor: colors.gray300,
    backgroundColor: colors.white,
    alignItems: 'center',
    justifyContent: 'center',
  },
  checkboxDone: { backgroundColor: colors.success, borderColor: colors.success },
  checkmark: { color: colors.white, fontSize: 14, fontWeight: '700', lineHeight: 18 },
});
```

Note how the styles reference design tokens from `constants/theme.ts` (`colors.card`, `radius.lg`, `spacing.base`) instead of hardcoded values — those tokens were extracted directly from the Figma file.

**On the Figma side**, Claude uses the MCP to write directly to the canvas:

*   Creates a `HabitLoop` file with two frames: `Home` and `Add Habit Modal`
    
*   The Home frame has an indigo header, three sample habit cards with auto layout, and a floating action button
    
*   The modal frame has an input field, cancel/save buttons, and the same design tokens (colors, corner radius, spacing)
    
*   Everything is properly named in the layers panel so a designer can pick it up and work with it
    

<img src="https://github.com/osielmesa/blog-posts/blob/main/drafts/claude+figma/figma.png?raw=true" alt="HabitLoop Figma file — Home Screen and Add Habit Modal frames generated by Claude"/>

The result is a design and a codebase that are in sync from day one — same colors, same spacing, same component structure.

* * *

### Run the app

```bash
cd HabitLoop
npx expo start
```

Scan the QR code with Expo Go and you'll see the app running on your phone, matching the Figma frames Claude just created.

<div class="imgs">
  <img src="https://github.com/osielmesa/blog-posts/blob/main/drafts/claude+figma/mob1.PNG?raw=true" alt="HabitLoop — Home screen with habit list and progress bar" />
  <img src="https://github.com/osielmesa/blog-posts/blob/main/drafts/claude+figma/mob2.PNG?raw=true" alt="HabitLoop — Add Habit modal with emoji picker" />
</div>

<style>
.imgs {
  display: flex;
  gap: 12px;              /* espacio entre imágenes */
  align-items: center;    /* alinea verticalmente */
  flex-wrap: nowrap;      /* evita que se vaya a la siguiente línea; cambiar a wrap si quieres que baje en pantallas pequeñas */
}
.imgs img {
  max-width: 50%;         /* que las dos entren; ajustar según diseño */
  height: auto;
  display: block;
}
@media (max-width: 600px) {
  .imgs { flex-direction: column; } /* en móvil apilan */
  .imgs img { max-width: 100%; }
}
</style>


* * *

### What just happened?

Claude used two channels at the same time:

1.  **File system** — scaffolded the Expo project, wrote TypeScript components and hooks
    
2.  **Figma MCP** — opened a new Figma file and placed frames, components, and design tokens on the canvas
    

Neither required you to switch tools, copy-paste specs, or manually align design decisions with code. The prompt was the single source of truth.

* * *

### Taking it further

Once you have the base app and the Figma file, the loop becomes very fast:

*   **Iterate on the design** in Figma → select the updated frame → ask Claude to regenerate the component
    
*   **Add a new screen** by prompting Claude to both code it and add the corresponding frame to Figma
    
*   **Adjust the design tokens** (colors, spacing) in one place and ask Claude to sync both files
    

```plaintext
Update the accent color from indigo to violet (#7c3aed) in both the app theme and the Figma variables.
```

### Make Claude smarter about your project

The more context Claude has about your codebase and conventions, the better the generated code fits your project. A few things that help a lot:

**CLAUDE.md** — Add a `CLAUDE.md` file at the root of your project. Claude reads it automatically at the start of every session. Use it to document your stack, folder conventions, naming rules, and any patterns you want Claude to follow consistently:

```markdown
# HabitLoop

## Stack
- React Native + Expo (SDK 53)
- TypeScript strict mode
- No third-party UI libraries

## Conventions
- Functional components with hooks only
- Styles via StyleSheet.create, never inline
- All design tokens imported from constants/theme.ts
- export default for components, named exports for hooks and types
```

**Skills** — Claude Code supports agent skills, which are pre-packaged instructions for common workflows. The Figma plugin ships with its own skills (like `generate-from-selection`) that tell Claude exactly how to extract design context and map it to code. For a React Native project there are useful skills for tasks like generating a new screen scaffold, creating a typed hook, or setting up navigation — so instead of explaining the pattern every time, you just invoke the skill.

There are public repositories of community-built skills you can browse and install directly. They're a great starting point: pick one that's close to what you need, install it, and adjust it to fit your project's conventions. The [Claude Code skills directory](https://claude.ai/code) and the `awesome-claude-code` repos on GitHub are good places to start.

**Rules** — You can define persistent rules that Claude follows in every interaction. What makes this powerful in React Native is that you can encode your own conventions once and never repeat them: always use `StyleSheet.create`, never inline styles; always use `Pressable` instead of `TouchableOpacity`; always wrap screens in `SafeAreaView`. Claude will apply them automatically.

**Memory** — Claude Code can store notes between sessions. If you tell it something important mid-conversation, it saves it and recalls it next time without you having to repeat it.

The best part is you don't have to set these up manually upfront. You can just ask Claude Code directly while you're working:

```plaintext
Remember that in this project we always use StyleSheet.create,
never inline styles. Add it as a rule.
```

```plaintext
Save a note that the Habit type lives in hooks/useHabits.ts
and uses completedToday, not done.
```

Claude will write the rule or memory entry on the spot. Over time, the project accumulates context and Claude's output gets progressively more aligned with how you actually build things.

Together these tools turn Claude from a generic code generator into something that knows your project, your conventions, and your design system.

* * *

## Troubleshooting

| Issue | Fix |
| --- | --- |
| Server not appearing in `/mcp` | Fully restart Claude Code |
| Authentication error | Run `/mcp`, select the server, choose **Authenticate** |
| Wrong code or shapes generated | Be more explicit in your prompt, or switch from link-based to selection-based |
| Desktop server not connecting | Make sure Dev Mode is active and nothing is selected in Figma before enabling the toggle |

* * *

## Wrapping Up

Connecting Claude Code to Figma via MCP changes how you start a project. Instead of designing first, then writing code that tries to match the design — or writing code first and documenting it in Figma later — you can do both at once from a single prompt. The HabitLoop demo is a small example of that: one prompt produces a working React Native + Expo app and a Figma file that are in sync from day one.

The setup itself takes a few minutes with the official plugin. From there, the more you invest in giving Claude context — a `CLAUDE.md`, a few rules, some memory notes, the right skills — the more the generated code feels like something you actually wrote, not something you have to heavily clean up.

For most people, the **official plugin (Path 1)** is the right starting point. If you need a scriptable or CI-friendly approach, the open-source `figma-developer-mcp` covers you.

If you have any corrections or suggestions, they're more than welcome. Happy coding! 🚀

* * *

*Based on the video* [*Connect Claude to Figma with MCP*](https://www.youtube.com/watch?v=aQ3vOzsuHIs) *and the* [*official Figma documentation*](https://help.figma.com/hc/en-us/articles/32132100833559-Guide-to-the-Figma-MCP-server)*.*