# NIP-99

## Linking to web clients

`draft` `optional` `author:AngusP`

This NIP describes a simple way for web-based (linkable) clients to provide the event ID of a given URL, so that clients can opt to fetch the event directly, or offer to swap the URL for a event ID when a user is writing a note.

As an example: I'm reading an interesting article on https://blabla.news, and want to publish a note about it. I copy the URL, and paste that into my note, hit publish. But now anyone that sees that and wants to read it will have to open up Blabla, even though the client they're using might support that event kind (`30023`) and the user may prefer how their client presents it to how Blabla does.

By using the method from this NIP, both the authoring client and the viewing client can resolve the webpage to an event or entity ID, and offer users writing the option of using `@nevent...` or `@naddr...` instead of a web link, and natively render the entity for viewing users. Clients can automatically replace the link with the nostr entity if they so choose.

The point of doing this is to keep nostr content in nostr, to acknowledge that different people have different nostr client preferences, and make sharing and linking to nostr content easier for users.

This NIP does note describe any changes to the nostr protocol, and instead a standard way for webpages to indicate that they are showing content that can also be found at an event ID.

### HTML meta tag

A web page that displays a nostr event or profile will add a `<meta />` tag to its `<head>` that gives the key or ID that the webpage shows. They should only do this if it "makes sense" and the event or profile won't change over time (unless it's a replaceable event), i.e. on a homepage that shows a feed, don't include the tag and point to the most-recent event or whatever, but do include it for a text note, a thread or a long-form post, etc.

 - The meta tag's `name` attribute must begin `nostr:` (the 'prefix')

Then either:

 1. The `name` attribute suffix must be `kind`
     - The meta tag's `value` attribute must match a nostr event kind.
 2. The `name` attribute suffix must be one of `npub`, `note`, `nprofile`, `nevent`, `naddr` [NIP-19](19.md)
     - The meta tag's `value` attribute must be the bech32 formatted string per [NIP-19](19.md) without the prefix. It must not be hex.

```html
<meta name="nostr:kind" content="1" />
<meta name="nostr:nevent" content="10elfcs4fr0l0r8af98jlmgdh9c8tcxjvz9qkw038js35mp4dma8qzvjptg" />
```

```html
<meta name="nostr:kind" content="30023" />
<meta name="nostr:naddr" content="1vl029mgpspedva04g90vltkh6fvh240zqtv9k0t9af8935ke9laqsnlfe5" />
```

```html
<meta name="nostr:npub" content="10elfcs4fr0l0r8af98jlmgdh9c8tcxjvz9qkw038js35mp4dma8qzvjptg" />
<!-- nostr:kind does not apply for a npub -->
```

The page can include multiple `nostr:` tags if it makes sense, e.g. both the `npub` of the author and the `note`.

The webpage must not include multiple tags with the same `name` attribute, as this is ambiguous.

### HTTP Header

In addition to providing HTML `<meta />` tags, a hint for a nostr identifier for a webpage can be given in the HTTP headers, so a client can call `HEAD` and avoid fetching the whole page.

```
x-nostr-kind: 1
x-nostr-nevent: 10elfcs4fr0l0r8af98jlmgdh9c8tcxjvz9qkw038js35mp4dma8qzvjptg
```

```
x-nostr-kind: 30023
x-nostr-nevent: 1vl029mgpspedva04g90vltkh6fvh240zqtv9k0t9af8935ke9laqsnlfe5
```

```
x-nostr-npub: 10elfcs4fr0l0r8af98jlmgdh9c8tcxjvz9qkw038js35mp4dma8qzvjptg
```

The scheme is the same as for the meta HTML tag, but prefixed with `x-` and replacing `:` with `-`.

Clients should be case-insensitive for the HTTP header keys.

Web pages should support both the header and meta tag if able, though the meta tag is likely to be easier to implement.
