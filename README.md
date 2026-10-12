# diff-match-patch

A dependency-free JavaScript port of Google's [Diff Match Patch](https://github.com/google/diff-match-patch) library, bundled with a small browser demo that renders a side-by-side, Git-style text comparison.

The library computes the difference between two texts, then lets you apply that difference as a patch onto a third text, tolerating imperfect matches. It is useful for wikis, version control, collaborative editing, and rich-text merging.

## Features

- **Diff** - character-level comparison of two texts, with semantic cleanup.
- **Match** - fuzzy search for a pattern inside a larger body of text.
- **Patch** - create a patch from two texts and apply it elsewhere, even when the target has drifted.
- **Formats** - serialize diffs and patches to text or a URI-encoded delta for storage and transport.
- **Zero dependencies** - a single ES5 file that runs in browsers and Node.js.

## Files

- `index.js` - the library (~78 KB). Exposes `diff_match_patch` plus the `DIFF_DELETE`, `DIFF_INSERT`, and `DIFF_EQUAL` constants.
- `index.html` - the demo page. Loads `index.js` and renders old vs. new text side by side with line numbers and `<ins>`/`<del>` highlighting.

## Quick Start

Open `index.html` directly in a browser, or serve the folder to avoid any `file://` restrictions:

```bash
python -m http.server 8000
# then visit http://localhost:8000
```

Edit the two text areas and press **Compare** to see the diff.

## Usage

### Browser

```html
<script src="index.js"></script>
<script>
  const dmp = new diff_match_patch();
  const diffs = dmp.diff_main("今天我们去北京旅游", "今天我们去上海旅游");
  dmp.diff_cleanupSemantic(diffs);
  document.body.innerHTML = dmp.diff_prettyHtml(diffs);
</script>
```

### Node.js

```js
const diff_match_patch = require('./index.js');

const dmp = new diff_match_patch();
const diffs = dmp.diff_main('Hello world', 'Hello there');
dmp.diff_cleanupSemantic(diffs);
console.log(diffs); // [[0,"Hello "],[-1,"world"],[1,"there"]]
```

## Common Operations

```js
const dmp = new diff_match_patch();

// Diff: compare two texts
const diffs = dmp.diff_main(oldText, newText);
dmp.diff_cleanupSemantic(diffs);          // merge small edits into readable blocks
dmp.diff_prettyHtml(diffs);               // colored HTML output
dmp.diff_toDelta(diffs);                  // "=2\t-1\t+x" - compact, URI-safe

// Match: fuzzy search
const index = dmp.match_main('abcdef', 'cde', 0);   // -> 2

// Patch: transfer an edit onto another text
const patches = dmp.patch_make(oldText, newText);
const patchText = dmp.patch_toText(patches);        // store or send this
const [result, applied] = dmp.patch_apply(patches, otherText);
```

Only apply patches to text that is *similar* to the original. Use `diff_main` for exact comparisons and `patch_apply` when the target may have changed; the second return value reports which patches applied cleanly.

## Configuration

Tune behavior by setting properties on an instance:

| Property | Default | Purpose |
| --- | --- | --- |
| `Diff_Timeout` | `1.0` | Seconds to spend on a single diff before giving up. |
| `Diff_EditCost` | `4` | Cost of an empty edit when cleaning up. |
| `Match_Threshold` | `0.5` | How loose a match may be (0.0 exact, 1.0 anything). |
| `Match_Distance` | `1000` | How far to search for a match. |
| `Patch_DeleteThreshold` | `0.5` | Minimum accuracy before a delete patch is dropped. |
| `Patch_Margin` | `4` | Context characters padded around patches. |
| `Match_MaxBits` | `32` | Maximum pattern length for the Bitap matcher. |

```js
const dmp = new diff_match_patch();
dmp.Diff_Timeout = 0;      // 0 means no time limit
dmp.Match_Threshold = 0.8; // prefer speed over precision
```

## API Reference

**Diff:** `diff_main`, `diff_cleanupSemantic`, `diff_cleanupSemanticLossless`, `diff_cleanupEfficiency`, `diff_commonPrefix`, `diff_commonSuffix`, `diff_levenshtein`, `diff_prettyHtml`, `diff_text1`, `diff_text2`, `diff_toDelta`, `diff_fromDelta`.

**Match:** `match_main`.

**Patch:** `patch_make`, `patch_apply`, `patch_toText`, `patch_fromText`, `patch_addPadding`, `patch_splitMax`.

Methods with a trailing underscore (for example `diff_bisect_`) are internal helpers and are not intended for external use.

## Development

There is no build step, bundler, or test runner.

- Syntax check: `node -c index.js`
- Manual verification: open `index.html` and confirm inserted text appears in `<ins>` and deleted text in `<del>`.

## License

Apache License 2.0. Original library copyright 2018 The diff-match-patch Authors.
