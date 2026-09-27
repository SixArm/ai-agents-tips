# Contents for glbal sharing with Svelte

a.k.a. table of contents

If there is a nav bar link label "Table of Contents"  then change to "Contents"

If there is a route "/table-of-contents/" or "/toc/" then change to "/contents/"

## If there is a contents area with parts and chapters

If there is a contents area (e.g. on the home page, or a contents route, or a contents page)
and parts and chapters (e.g. a guide book), then:

- List parts and chapters as a nested list.
- Each part label is "<integer> <title>". Example "1 Lorem Ipsum".
- Each chapter label is "<decimal> <title>". Example "1.0 Lorem Ipsum".

Example:

- 1 <title>
  - 1.0 <title>
  - 1.1 <title>
  - 1.2 <title>
- 2 <title>
  - 2.0 <title>
  - 2.1 <title>
  - 2.2 <title>

NO:

- NO tiles.
- NO display tables.
- NO display columns.
- NO CSS flexbox.
- NO CSS grid.
