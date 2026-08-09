# Translate the game into another language

The template ships in **English and French**. Every label the player reads is a text value that goes through Unreal's localization system, so the language swaps at runtime with no Blueprint work.

What comes with the template is the compiled language data, in `Content/TheLastTemplate/Localization/Game/`:

| File | What it is |
|---|---|
| `Game.locmeta` | The description of the target. |
| `en/Game.locres` | Every English string of the template. |
| `fr-FR/Game.locres` | The French of the same strings. |

---

## The three config lines it needs

Localization is read from paths listed in the project config. Open `Config/DefaultGame.ini` and make sure these are there. Add them if they are not, under `[Internationalization]`:

```
+LocalizationPaths=%GAMEDIR%Content/TheLastTemplate/Localization
+CultureMappings="fr;fr-FR"
+CultureMappings="fr-BE;fr-FR"
+CultureMappings="fr-CA;fr-FR"
+CultureMappings="fr-CH;fr-FR"
```

The first line is the one that matters. Without it the `.locres` files sit on disk and nothing reads them, the game runs in English, and there is no warning of any kind.

The culture mappings send every French variant to the same file, so a player in Belgium, Canada or Switzerland gets French instead of falling back to English.

For a packaged build, `Config/DefaultGame.ini` also needs the cultures staged:

```
+CulturesToStage=en
+CulturesToStage=fr-FR
+DirectoriesToAlwaysStageAsUFS=(Path="TheLastTemplate/Localization")
```

---

## How the player changes language

The options menu has a **Language** page, built from `DA_Page_Language` like every other page. It holds one selector row with the languages on it.

When the player picks one, `SG_MenuSettings` calls `Apply Language`, which calls **Set Current Culture**. The interface swaps on the spot. The choice is stored with the rest of the settings under the key `lang.index`, so it survives a restart.

Nothing else in the project sets a culture. To force a language from your own graph, call **Set Current Culture** the same way and save the settings.

---

## Add your own language

The template gives you compiled files, not the translation sources, so a new language is a target of your own. That is the normal way to work anyway: your text and the template's text stay in separate files and Unreal merges them at runtime.

1. Open **Tools**, then **Localization Dashboard**.
2. Create a target, or use the `Game` target your project already has.
3. Under **Cultures**, add the languages you want.
4. Click **Gather Text**. It walks the project and collects every text value, yours and the template's.
5. Translate. **Export** writes a `.po` file that any translation tool opens, and **Import** brings it back.
6. Click **Compile Text**.
7. Add your culture to `CulturesToStage` in `Config/DefaultGame.ini`.

Leave `+LocalizationPaths` pointing at the template folder while you do this. Unreal reads every path in the list and merges them, so the template's own English and French keep working next to your target instead of being replaced by it.

---

## Put the new language in the options menu

The page lists what the player can pick, so a compiled language nobody can select is invisible.

1. Open `DA_Page_Language`.
2. Find the selector row.
3. Add an entry to `Options Keys`, following the ones already there: `lang.en`, `lang.fr`.
4. Add the matching text for that key so the option shows a name.

The order of the entries is the order of the values saved in `lang.index`. Adding a language at the end is safe. Inserting one in the middle shifts every saved choice by one, so players come back to the wrong language.

---

## Your own text

Two rules cover almost everything.

**Use Text, not String.** A `Text` pin, a `Text` variable and a text block in a widget are all gathered. A `String` is never gathered and never translated. If a label of yours stays English in a translated build, that is the first thing to check.

**Gather again after you add screens.** New text does not appear in the manifest until you click **Gather Text** again. Then translate the new entries and compile.

If you build a sentence out of pieces, use **Format Text** with named arguments rather than joining strings. Word order changes between languages, and a format string lets the translator move the pieces around. The template already does this for the item counter and the ingredient list.

---

## Fonts

The fonts that ship carry accented characters, which is what French needs. A language that uses another script, such as Russian, Greek, Japanese or Arabic, needs a font that has those glyphs. Swap the font on the widgets, or add a fallback font in the font asset itself.

Nothing in the interface assumes a language. The menus and the HUD are laid out with boxes that grow, so a German label twice as long as its English original pushes its row instead of being cut.

---

[Join the Discord](https://discord.gg/EqHCtq38jy)
