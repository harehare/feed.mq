<h1 align="center">feed.mq</h1>

An [RSS 2.0](https://www.rssboard.org/rss-specification) / [Atom](https://validator.w3.org/feed/docs/atom.html) feed parser implemented as an [mq](https://github.com/harehare/mq) module.

## Features

- Parses both RSS 2.0 and Atom 1.0 documents into a single normalized structure
- Handles `content:encoded`, `dc:creator`, CDATA sections, enclosures, and categories
- Resolves Atom's multiple `<link>` elements (`alternate` / `self` / `enclosure`)
- Renders a Markdown table summary of a feed's items

## Installation

Copy `feed.mq` to your mq module directory, or place it anywhere and reference it with `-L`.

```sh
cp feed.mq ~/.local/mq/config/
```

### HTTP Import (no local installation needed)

If `mq` was built with the `http-import` feature, you can import directly from GitHub without any local setup:

```sh
mq -I raw 'import "github.com/harehare/feed.mq" | feed::feed_parse(.) | feed::feed_items(.)' feed.xml
```

Pin to a specific release with `@vX.Y.Z`:

```sh
mq -I raw 'import "github.com/harehare/feed.mq@v0.1.0" | feed::feed_parse(.) | feed::feed_items(.)' feed.xml
```

## Usage

```sh
mq -L /path/to/modules -I raw \
  'import "feed" | feed::feed_parse(.) | feed::feed_items(.)' feed.xml
```

If you copied it to the mq built-in module directory:

```sh
mq -I raw 'import "feed" | feed::feed_parse(.) | feed::feed_items(.)' feed.xml
```

## API

### `feed_parse(input)`

Parses an RSS 2.0 or Atom document and returns a normalized structure, regardless of
which format the input was in:

```
{
  "format": "rss" | "atom",
  "title": <string|None>,
  "link": <string|None>,
  "description": <string|None>,
  "language": <string|None>,
  "id": <string|None>,
  "updated": <string|None>,
  "generator": <string|None>,
  "image": <string|None>,
  "items": [<item>, ...]
}
```

where each item/entry is:

```
{
  "title": <string|None>,
  "link": <string|None>,
  "description": <string|None>,
  "content": <string|None>,
  "published": <string|None>,
  "updated": <string|None>,
  "id": <string|None>,
  "author": <string|None>,
  "categories": [<string>, ...],
  "enclosure": {"url": <string>, "type": <string|None>, "length": <string|None>} | None
}
```

Dates (`pubDate` / `published` / `updated`) are kept as the raw strings found in the
document — this module does not parse or normalize them.

Raises an error if the input is not a `<rss>` or `<feed>` document.

### Other functions

| Function | Description |
|---|---|
| `feed_is_rss(feed)` | `true` if the parsed feed is RSS |
| `feed_is_atom(feed)` | `true` if the parsed feed is Atom |
| `feed_items(feed)` | The normalized `items` array |
| `feed_to_markdown_table(feed)` | Renders a Markdown table of title / link / date per item |

## Example

Given `feed.xml` (RSS 2.0):

```xml
<?xml version="1.0" encoding="UTF-8"?>
<rss version="2.0">
  <channel>
    <title>Example Blog</title>
    <link>https://example.com/</link>
    <description>An example RSS feed</description>
    <item>
      <title>Hello, World!</title>
      <link>https://example.com/posts/1</link>
      <pubDate>Tue, 28 Jul 2026 09:00:00 GMT</pubDate>
      <category>mq</category>
    </item>
  </channel>
</rss>
```

```sh
mq -L . -I raw 'import "feed" | feed::feed_parse(.) | ."title"' feed.xml
# => "Example Blog"

mq -L . -I raw 'import "feed" | feed::feed_parse(.) | feed::feed_items(.)[0]["categories"]' feed.xml
# => ["mq"]

mq -L . -I raw 'import "feed" | feed::feed_parse(.) | feed::feed_to_markdown_table(.)' feed.xml
# | Title | Link | Date |
# | --- | --- | --- |
# | Hello, World! | https://example.com/posts/1 | Tue, 28 Jul 2026 09:00:00 GMT |
```

The same query works unmodified against an Atom document — `feed_parse` normalizes
both formats to the same shape.

## Limitations

- RSS 1.0 (RDF) feeds are not supported, only RSS 2.0 and Atom.
- Atom `xhtml`-typed content (nested XHTML instead of plain text) is not
  reconstructed; only direct text content is captured.

## Compatibility

Requires [mq](https://github.com/harehare/mq) v0.7 or later.

## License

MIT
