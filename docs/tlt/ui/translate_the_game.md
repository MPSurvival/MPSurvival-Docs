# Translate the game into another language

!!! info "Outline, not written yet"
    Answers how a buyer ships the game in a second language, and what they have to do for the text they add themselves.

## Planned sections

- What is translatable and what is not
- How text works in this template: every label the player reads is a text value, not a hard coded string
- The language page in the options menu, and what picking a line actually does
- Set up the Localization Dashboard for the project
- Gather the text out of the project
- Translate, then compile the language files
- Add a third language of your own
- Accents and fonts: the one thing that can silently break a language
- When you add your own screens, what to do so they follow the language

## Open points to settle before this page is written

- The localization pass has not been done yet, so this page stays an outline until the languages are in.
- Verified on disk, and worth reusing when the page is written: the menu is already key based. Every row in a settings page Data Asset carries keys such as `audio.overall` and `audio.overall.desc` rather than literal text, and `DA_Page_Language` already offers `lang.en` and `lang.fr`.
- Verified on disk: there is no `Content/Localization` folder, no compiled language file anywhere in the project, no string table asset, and `CulturesToStage` lists English only. Nothing in the manual may suggest a second language works today.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
