---
title: Deprecating notes in Obsidian without resorting to an Archived folder
slug: deprecating-notes-in-obsidian-without-resorting-to-an-archived-folder
tags: obsidian,notes,writing,research,documentation,pkm
domain: til.jacklinke.com
---

## Background

I [asked on Mastodon](https://social.jacklinke.com/@jack/112793088080634414) for input on how folks using various [PKM](https://en.wikipedia.org/wiki/Personal_knowledge_management) tools deprecate older notes that are still useful for their historical or contextual information, but not relevant to the current context or progress of the note's topic.

While I didn't get an answer that quite met my goals, I did receive a number of wonderful responses and recommendations for other tools and approaches to my problem. And now I'm excited to try emacs after years of eyeing it from a distance.

## Solutuion

My ideal solution to achieve the goal of deprecating notes would be to add a property in the markdown frontmatter, which the PKM tool would use to make the note less visible in the navigation pane. Unfortunately, I was unable to find any plugin, setting, or existing process within [Obsidian](https://obsidian.md/) to achieve this. And I'm not currently equpped with the time required to build a custom plugin to achieve this.

Instead, I ended up using the [TreeFocus](https://github.com/iOSonntag/obsidian-plugin-treefocus/) community plugin. This plugin can highlight or dim entries in the navigation pane based on the contents of the note's filename or path. You can match exact names, starting/contains/ending values, or use a regex pattern.

Not perfect, but I can work with that.

## Customization

I did not like the formatting options that ship with the plugin, so I added a small [css snippet](https://help.obsidian.md/Extending+Obsidian/CSS+snippets) to override the default formatting option.

The [original css](https://github.com/iOSonntag/obsidian-plugin-treefocus/blob/master/styles.css#L48-L54) makes highlighted items bigger **and** bolder. It shinks **and** lowers the opacity of dimmed items.

![Default Options](https://raw.githubusercontent.com/jacklinke/til/main/obsidian/20240717-default.png)

```css
.tree-item-self[data-treefocus-theme="DEFAULT"] {
  --treefocus-highlight-font-scale: 1.3;
  --treefocus-highlight-font-weight: 700;

  --treefocus-dim-font-scale: 0.7;
  --treefocus-dim-opacity: 0.5;
}
```

That was both too much and not enough for me. I found the changes to text size make navigation confusing, the font weight for highlighting was **too** bold, and the change in opacity for dimmed items was not dim enough. This new very simple snippet causes highlighted items to be slightly more **bold** than regular items, and further increases the opacity for dimmed items.

![Default Options](https://raw.githubusercontent.com/jacklinke/til/main/obsidian/20240717-custom.png)

```css
.tree-item-self[data-treefocus-theme="DEFAULT"] {
  --treefocus-highlight-font-scale: 1.0;
  --treefocus-highlight-font-weight: 600;

  --treefocus-dim-font-scale: 1.0;
  --treefocus-dim-opacity: 0.3;
}
```

## My Current Settings

I set it up to dim notes that are prepended with `_`, `.`, or contain the word `deprecated`. I don't really have a use-case for highlighting at this time, but I could see it being useful to identify those notes within a topic that are currently being worked or researched.

![Default Options](https://raw.githubusercontent.com/jacklinke/til/main/obsidian/20240717-settings.png)
