# Examples

Adapted from the upstream repository's `examples/` folder.

## Deep clone (option 3: platform feature)

Task: "Deep clone this object."

Without ponytail: `npm install lodash` for `cloneDeep`, or `JSON.parse(JSON.stringify(obj))`, which drops `Date`, `Map`, `Set`, `undefined` and circular references.

With ponytail:

```js
const copy = structuredClone(original);
```

Skipped: lodash. `structuredClone` is built into browsers and Node.js 17+ and handles the types the JSON round trip loses.

## Debounce (option 5: minimal obvious code)

Task: "Add debounce to a search input in vanilla JavaScript. It currently fires an API call on every keystroke."

Without ponytail: 116 lines. A reusable `debounce` utility, an "advanced" version with `immediate` and `cancel` options, a loading indicator, HTML, CSS and a benefits table.

With ponytail:

```javascript
const searchInput = document.querySelector('input[type="search"]');
let debounceTimer;

searchInput.addEventListener('input', (e) => {
  clearTimeout(debounceTimer);
  debounceTimer = setTimeout(() => {
    fetch(`/api/search?q=${encodeURIComponent(e.target.value)}`)
      .then(r => r.json())
      .then(data => console.log(data)); // replace with your render
  }, 300);
});
```

Skipped: debounce utility, options, loading state, styling. `setTimeout` plus `clearTimeout` is the debounce. Extract a utility once three inputs need it.
