# search-combobox

Searchable dropdown with async search results.

## Attributes

|  Attribute           |  Type        |  Description                                        |  Support JS expressions  |
| -------------------- | ------------ | --------------------------------------------------- | ------------------------ |
|  `data-placeholder`  |  `string`    |  Placeholder text for search input                  |  ✅                     |
|  `data-values`       |  `function`  |  Search function (receives query, returns results)  |  ✅                     |
|  `data-default`      |  `function`  |  Function returning default value                   |  ✅                     |

## Output Value

`string` - Selected value, typically an ID returned by the search provider

## Example

Manifest:

```json
"modules": [
    "modules/CitySearch.mjs"
]
```

HTML:

```html
<meta name="x-icue-property" content="city"
      data-label="tr('City')"
      data-type="search-combobox"
      data-values="CitySearch.search"
      data-default="CitySearch.getDefault"
      data-placeholder="tr('Search city...')">
```

## Usage in JavaScript

```javascript
console.log(city); // "12345" (selected city ID)
```

## Reference Implementation

See the **Weather** widget for a complete implementation: `<<iCUE install dir>>/widgets/Weather/`
