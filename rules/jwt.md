# JWT must not be used for keeping users logged in

JWTs must not be used for keeping your user logged in. 

They are not designed for this purpose, they are not secure.

Instead, use regular cookie sessions.

If you've got a bit of time to watch a presentation on it, I highly recommend this talk: https://www.youtube.com/watch?v=pYeekwv3vC4 (Note that other topics are largely skimmed over, such as CSRF protection. You should learn about other topics from other sources. Also note that "valid" usecases for JWTs at the end of the video can also be easily handled by other, better, and more secure tools. Specifically, [PASETO](https://paseto.io/).)

A related topic: Don't use localStorage (or sessionStorage) for authentication credentials, including JWT tokens: https://www.rdegges.com/2018/please-stop-using-local-storage/

The reason to avoid JWTs comes down to a couple different points:
- The JWT specification is specifically designed only for very short-live tokens (~5 minute or less). Sessions need to have longer lifespans than that.
- "stateless" authentication simply is not feasible in a secure way. You must have some state to handle tokens securely, and if you must have a data store, it's better to just store all the data. Most of this article and the followup it links to describes the specific issues: http://cryto.net/~joepie91/blog/2016/06/13/stop-using-jwt-for-sessions/
- JWTs which just store a simple session token are inefficient and less flexible than a regular session cookie, and don't gain you any advantage.
- The JWT specification itself is not trusted by security experts. This should preclude **all** usage of them for anything related to security and authentication. The original spec specifically made it possible to create fake tokens, and is likely to contain other mistakes. [This article](https://paragonie.com/blog/2017/03/jwt-json-web-tokens-is-bad-standard-that-everyone-should-avoid) delves deeper into the problems with the JWT (family) specification.

You can't securely have truly stateless authentication without having massive resources, see the cryto.net link above. Also, [Stateless is a lie](https://gist.github.com/samsch/259517828ab4557c5c8b72ca1252992d).

*I don't know how to setup sessions!* You don't regularly see articles explaining sessions because the technology isn't particularly new. You also shouldn't need third party information for setup. A session implementation's documentation should take you through the setup process by itself. Almost any web server framework will contain an implementation for sessions, and usually it's very easy to enable if it isn't enabled by default. 

Express and other Node.js frameworks are somewhat exceptions to this rule, primarily because they are highly modular and single purpose. For Express, you simply use the `express-session` middleware and a store connector which works with your store (I recommend `connect-session-knex`, to be used with Postgres, MySQL, or possibly SQLite).

## Short term tokens

If you do need a short-lived, signed token for something, there is a better spec called [PASETO](https://paseto.io/) which *is* designed to be secure. Just make sure you aren't using them for sessions.

## How sessions work

I recommend checking out [this gist by joepie91](https://gist.github.com/joepie91/cf5fd6481a31477b12dc33af453f9a1d) to learn more how sessions work.