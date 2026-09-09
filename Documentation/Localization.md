Package entry point: [README](../README.md)

---

<h1><img src="images/colors.png" alt="Logo" width="80" align="middle" />&nbsp; Themes - Localization</h1>

`Themes` includes optional localization integration when Unity Localization is installed.

## Contents

- [Requirements](#requirements)
- [Available Types](#available-types)
- [Setup](#setup)
- [Usage Notes](#usage-notes)

---

## Requirements

- `com.unity.localization` installed in the project

The localization binding uses version defines and is enabled automatically when Localization, uGUI and TextMeshPro are present. TextMeshPro is bundled with uGUI 2.0.0+, or installed separately for older uGUI versions.

## Available Types

Namespace: `derHugo.Themes.Localization`

- [`BindLocalizedStringToTextMeshPro : Bind<LocalizedString, TMP_Text>`](ScriptingAPI.md#optional-localization-api)

## Setup

1. Install `com.unity.localization`.
2. Add a localization-themed definition in your [`ThemeDatabase`](ScriptingAPI.md#themedatabase-runtime-api) (type `LocalizedString`).
3. Add [`BindLocalizedStringToTextMeshPro`](ScriptingAPI.md#optional-localization-api) to a GameObject with `TMP_Text`.
4. Assign a themed `LocalizedString` (a `ThemeValue<LocalizedString>`) for each binding state you need.
5. Switch theme as usual; the localized string reference updates with theme selection.

## Usage Notes

- [`BindLocalizedStringToTextMeshPro`](ScriptingAPI.md#optional-localization-api) requires `TMP_Text` and `LocalizeStringEvent`.
- If `LocalizeStringEvent` is missing, the component can add it automatically.
- The binding updates `LocalizeStringEvent.StringReference`, not raw `TMP_Text.text`.

Related pages:

- [Scripting API](ScriptingAPI.md#optional-localization-api)
- [ThemeDatabase](ThemeDatabase.md)
- [Getting Started](GettingStarted.md)
