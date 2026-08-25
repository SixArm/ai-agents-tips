# Web form local storage exports

Web form must provide these:

- localStorage: HTMO form fields save 400 ms after each keystroke or tick, and restore on the next visit. Every read and write is wrapped in try/catch, so private browsing or a full store shows "This browser will not let the page save your answers. Export before you leave." instead of failing silently. Its buttons are wired by id, not by Svelte — content routes have csr = false, so nothing on the page hydrates.

- Button "Clear" with a confirm: reset all HTML form inputs to defaults, and erase localStorage. This is important because of security.

- Button "Export TSV": export Tab Separated Values. Text is two rows (one heading row and one data row), escaping \ → \\, tab → \t, CR → \r, newline → \n, so a multi-line answer can't break the row. Columns are one per HTML form input name. Downloads as export.tsv.

- Button "Export JSON": export JavaScript Object Notation. Text is a string, a tick list is an array, a rating is a number (or null if unanswered), etc.

Verify each works.
