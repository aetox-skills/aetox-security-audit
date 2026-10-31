# Audit questions, by category

Each row is a question to ask of the code, and what to search for to find
the places where the question applies. A hit on the search is a place to
read, not a finding; SKILL.md says what a finding is and how a candidate
is refuted. The fix a finding proposes is the row for its weakness in
`aetox-security`'s `references/defenses.md`, in the project's own idiom.

In a full audit, paste the rows that fit a unit into its hunter brief,
beside its block from `classes.md`. An app with an agent, tools or MCP
servers is also read against `agentic.md`.

## Application code

| Category | The question | Where to look (grep leads) |
|---|---|---|
| Command injection | Does anything from outside reach a shell or a process argument list without being one argument, or being validated against a fixed set? | `exec.Command`, `sh -c`, `subprocess`, `child_process`, `os.system`, `Runtime.exec`, backticks; then trace the arguments backwards |
| SQL / NoSQL injection | Is a query built by string concatenation or format with a value from outside? | `fmt.Sprintf(` next to `SELECT`/`INSERT`/`UPDATE`/`WHERE`, `+ "` in a query, `$where`, `.raw(`, `text(` |
| Path traversal | Can a name from outside reach `open`/`read`/`write`/`delete`/`serve` without being resolved against a root and checked to still be under it? Symlinks resolved? | `filepath.Join(root, userInput)`, `os.Open`, `sendFile`, `path.join`, `../`, `ReadFile` |
| Template / expression injection | Does user input become the template rather than a value in it? | `template.New(...).Parse(userInput)`, `eval`, `new Function`, `render_template_string`, `Jinja2` with `autoescape=False` |
| Deserialisation | Is untrusted bytes decoded into objects by a format that can carry code or references? | `pickle`, `yaml.load` without `SafeLoader`, `unserialize`, `ObjectInputStream`, `Marshal.load`; JSON is fine, what is done with it after may not be |
| XSS | Does a value from outside reach HTML, an attribute or a script without escaping for that context? | `innerHTML`, `dangerouslySetInnerHTML`, `{@html`, `v-html`, `template.HTML(`, `\|safe`, `document.write` |
| Header injection | Does a value from outside reach a response header, a cookie or a redirect `Location` with a CR or LF still in it? | `header[...] =`, `Header().Set(`, `setHeader(`, `Set-Cookie`, a `Location` built from input |
| Log injection | Does a value from outside reach a log line with its newlines intact, so it can forge an entry that looks like the app's own? | logger calls handed request fields, `f"... {msg}"` or `%s` into a log line |
| Pattern DoS | Is a regular expression from outside compiled and run, or input run through a backtracking pattern, with no RE2 engine and no timeout? | `re.compile(`, `re.search(pattern,`, `new RegExp(`, `regexp.MustCompile(` on anything from a request |
| XPath / XML injection | Is an XPath query built from input by concatenation, or XML from outside parsed with external entities resolved? | `.xpath(f"`, `+ "']"` in a query, `resolve_entities=True`, a parser without `disallow-doctype-decl` |
| SSRF | Does a URL, a host or an id from outside end up in a URL the server fetches? Can it reach localhost, link-local (169.254.x.x) or the metadata endpoint, or climb the path with `..`? | `http.Get(userURL)`, `fetch(url)` on the server, `requests.get(`, image proxies, webhooks, "import from URL", f-strings that build a URL |
| Authentication | Can the check be skipped: a route without the middleware, a token accepted without signature or `alg: none`, a password compared with `==`, a reset token that is guessable or never expires? | route tables, `jwt.Parse` without key verification, `subtle.ConstantTimeCompare` absent, `math/rand` for tokens |
| Authorization | Does the handler check that the caller may touch *this* object, not just that the caller is logged in? | any handler that takes an id from the request and loads by it; admin-only routes; "is owner" checks |
| Session | Cookie without `HttpOnly`/`Secure`/`SameSite`, session id not rotated at login, logout that does not invalidate | `http.SetCookie`, `session.regenerate`, `Set-Cookie` |
| Secrets in source or history | A key, token, password or private key in a file or in a past commit | `git log -p -S "BEGIN PRIVATE KEY"`, `-S "sk-"`, `-S "AKIA"`, `.env` committed, `password =` in config; a scanner (`gitleaks`) does this well |
| Crypto | MD5/SHA1 for passwords, ECB mode, a fixed IV or nonce, `math/rand` where `crypto/rand` is needed, `InsecureSkipVerify: true`, `verify=False` | those literals |
| Data exposure | Secrets, tokens, or personal data in log lines, error messages, stack traces returned to the client, debug endpoints left on; a lookup by credentials that hands back the password it matched on | `log.Printf` with request bodies or headers, `pprof` on a public listener, `DEBUG = True`, verbose error handlers |
| Code execution | `eval`, dynamic import or `require` of a name from input, plugins loaded from a path the user controls | `eval(`, `exec(`, `importlib.import_module(userInput)`, `require(variable)`, `plugin.Open` |
| Supply chain (A03:2025) | A dependency with a known vulnerability, unpinned (`latest`, a branch), a package with an install script, a lockfile missing or not used in CI. And the build and release path: secrets a pull request from a fork can reach, a release artifact nobody signs or checksums, an internal package name a public registry could claim | `go.mod`, `package.json` + lock, `requirements.txt`, `postinstall`, `curl \| sh` in build scripts; CI workflows (`pull_request_target`, `secrets.` in jobs that run fork code), release scripts, registry config |
| Configuration | Fail-open defaults: auth off unless configured, CORS `*` with credentials, bound to `0.0.0.0` when local was meant, TLS optional, file permissions world-writable | config defaults, `AllowAllOrigins`, `Listen("0.0.0.0` vs `127.0.0.1`, `0777`, `chmod` |
| Exceptional conditions (A10:2025) | When something fails, does the code fail closed? An error, timeout or nil that ends in "allowed"; an exception nobody catches; an error message that shows internals (CWE-209); a lock, file or transaction left half done when the error comes, so a retry or a second request runs against it | `catch`, `recover()`, `except: pass`, `if err != nil { return nil }` beside a permission or payment check, `defer` and `finally` blocks, rollback paths |
| Logging and alerting (A09:2025) | Does a failed login, a denied permission or an admin action leave a record that reaches someone who will act on it? And do the logs hold no password, token or personal data? A record nobody is alerted to is half the job | auth and admin handlers, audit-log writes, alert or metric hooks, what the logger is handed (`%+v` of a request, whole bodies) |
| Race / TOCTOU | A security decision made, then the thing re-read: check a file then open it by name; check a balance then debit in a second query | check-then-act pairs on files, money, quotas; missing transactions or locks |
| Uploads | Extension trusted, content type trusted, file written under a name from the client, served back from the same origin as the app | upload handlers, `Content-Type` from the request used to decide, `filename` from the form |
| Redirects | Open redirect to a URL from the request; low on its own, high when it carries a token | `Redirect(w, r, userURL` , `next=`, `returnTo=` |

## Reading a stack quickly

- Go: the sandbox is a `SandboxRoot`-style resolve; check every file tool
  goes through it. `exec.Command` with a slice is safe from shell parsing
  and unsafe when the binary name comes from input. `html/template`
  escapes, `text/template` does not.
- Node / TypeScript: `child_process.exec` parses a shell, `execFile` does
  not. Svelte and React escape by default; `{@html}` and
  `dangerouslySetInnerHTML` are the exceptions to read.
- Python: `subprocess` with `shell=True`, `yaml.load` without a loader,
  `pickle`, f-strings into SQL. Flask `debug=True` on a public bind.
- SQL: any driver has a parameter form; the finding is the query that does
  not use it.

## Sources

| Kind | Source | Version, checked | What Aetox changed |
|---|---|---|---|
| Industry standard | [OWASP Top 10:2025](https://top10.owasp.org/2025/), [A09](https://top10.owasp.org/2025/A09_2025-Security_Logging_and_Alerting_Failures/), [A10](https://top10.owasp.org/2025/A10_2025-Mishandling_of_Exceptional_Conditions/) | 2025 edition (final January 2026), checked 2026-09-26 | Rows written as questions to the code; resource exhaustion from A10 is a finding when `exhaustion.md`'s rules hold, and a resource left held by an unhandled error is a finding |
| Industry standard | OWASP Top 10 for Agentic Applications | 2026, ASI01 to ASI10 | Phrased for an app like this one |
| Method | anthropics/claude-code-security-review (MIT) | category list | Category names |
| Method | cloudflare/security-audit-skill (MIT) | checked 2026-10-13 | The full loop, the hunter, critic and verifier briefs, the boundary gate, the severity anchors and the domain files, rewritten and fitted to read-only helpers |
| Method | trailofbits/skills fp-check, variant-analysis (CC BY-SA 4.0) | checked 2026-09-26 | Method only, no text taken: the refute gates cut from six to four, the variant search after a finding |
| Method | OpenAI Codex Security | research preview, March 2026 | The threat model written first and shown to the user |
| Research | BRACE, arXiv 2606.16244 | June 2026 | `defenses.md` is read by the rows named per unit, three to five cues beat a long list |
| Guide | OpenSSF Security-Focused Guide for AI Code Assistant Instructions | September 2025 | The attack-it pass and the normal input beside it; no persona |
