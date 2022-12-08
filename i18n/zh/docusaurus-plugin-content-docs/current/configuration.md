---
sidebar_position: 3
---

# Configuration

All configuration files are located in `.i18n-cms` folder in root

## config.json
`config.json` is required for using this app. It helps us to define existing namespaces and languages and locate your translation files. 

| Key             | Required  |
|-----------------|-----------|
| languages       | *         |
| defaultLanguage | *         |
| fileType        | *         |
| pattern         |           |
| useCustomPath   |           |
| namespaces      |           |

### defaultLanguage
Required <br/>
`string` <br/>
For example : `en`<br/>
Specify the default language.
We will sort the translation key by order of the translation file in the default language

### languages
Required <br/>
`string[]` <br/>
Specify the existing languages of your translations file.<br/>
`languages` value will be updated if your add/remove languages through this app.<br/>
If you add any languages manually, please also update `languages` value in this config file.

### fileType
Required <br/>
Specify the file type of your translation files. We only support `json` and `yaml` for now. <br/>
Valid options:
- `json`
- `yaml`
- `json_flatten`
- `yaml_flatten`

### pattern
Required if `useCustomPath` is `false` or `undefined`<br/>
`string`<br/>
Specify the file path pattern to your translation file. <br/>
`pattern` must contain `:ns` and `:lng` named segments representing namespace and language.

We will load your namespaces and languages from `${pattern}.${fileType}` by default.<br/>
Check `useCustomPath` below if your translation files are not structured like this.

#### Example with default file structure

```json title=".i18n-cms/config.json"
{
  "fileType": "json",
  "pattern": "public/locales/:lng/:ns",
  "languages": ["en", "zh"],
  "defaultLanguage": "en"
  ...
}
```

```md title="File structure:"
- public
  - locales
    - en
      - translationA.json
      - translationB.json
    - zh
      - translationA.json
      - translationB.json
    - vi <-- ignored, because 'vi' not defined in 'languages'
      - translationA.json
      - translationB.json
```

### useCustomPath
`boolean` <br/>
If your translation files are not structured as `${pattern}.${fileType}`, you can 
- set `useCustomPath` to `true`
- define `namespaces` in `.i18n-cms/config.json`
- Add `.i18n-cms/getCustomPath.js`


### namespaces
`string[]`  <br/>
Required if `useCustomPath` is `true`<br/>
Specify the existing namespaces of your translations file.<br/>
`namespaces` value will be updated if your add/remove languages through this app.<br/>
If you add any namespaces manually, please also update `namespaces` value in this config file.


## getCustomPath.js
Not required by default. Only work when
- `useCustomPath` is `true`
- `namespaces` is defined

in `.i18n-cms/config.json`

Export a function to get the path of translation file by language and namespace.
```js title='.i18n-cms/getCustomPath.js'
/** 
 * Get the path of translation file by language and namespace
 * @param {Object} data
 * @param {string} data.namespace - The namespace of translation file
 * @param {string} data.language - The language of translation file
 * @param {Object} data.repoConfig - config defined in `.i18n-cms/config.json`
 * @return {string} path of translation file
 */
```

`FILE_TYPE_MAP_DATA` is a global value that you can access in the function.
```json title='FILE_TYPE_MAP_DATA'
{
  "json":{"ext":"json","label":"JSON"},
  "yaml":{"ext":"yaml","label":"YAML"},
  "json_flatten":{"ext":"json","label":"JSON (flatten)"},
  "yaml_flatten":{"ext":"yaml","label":"YAML (flatten)"}
}
```

### Example with custom file structure

```json title=".i18n-cms/config.json"
{
  "defaultLanguage": "en",
  "languages": ["en", "zh"],
  "fileType": "json",
  "useCustomPath": true,
  "namespaces": [
    "common",
    "featureA",
    "featureB",
  ]
}
```

```md title="File structure:"
- locales
  - en.json
  - zh.json
- feature
  - featureA
    - locales
      - en.json
      - zh.json
  - featureB
    - locales
      - en.json
      - zh.json
```
```js title='.i18n-cms/getCustomPath.js'
/** 
 * Get the path of translation file by language and namespace
 * @param {Object} data
 * @param {string} data.namespace - The namespace of translation file
 * @param {string} data.language - The language of translation file
 * @param {Object} data.repoConfig - config defined in `.i18n-cms/config.json`
 * @return {string} path of translation file
 */
export default function getCustomPath({ namespace, language, repoConfig }) {
  const ext = FILE_TYPE_MAP_DATA[repoConfig.fileType].ext;
  switch (namespace) {
    case "common":
      return `locales/${language}.${ext}`;
    default:
      return `feature/${namespace}/locales/${language}.${ext}`;
  }
}
```

