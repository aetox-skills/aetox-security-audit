# Desktop, webview and local IPC

## When to use this file

The target is a desktop or mobile app, a webview host, a deep-link or
custom-scheme handler, a local server or daemon, a named pipe or socket, a
privileged helper, an installer or an updater. The attacker may be a
downloaded document, a remote web page, another app, another OS user, a
sandboxed child, or a lower-privileged account. Say which one; not every
local actor holds the same power.

## Core discipline

```text
- Name the realistic local or remote-content attacker: another app,
  another OS user, a sandboxed child, an untrusted document, a remote page.
  Harm a user does to themselves with their own account crosses nothing.
- A path, a process name, a window title, an Origin header from a local
  client, or a sender field in the message is not peer authentication. OS
  peer credentials, code identity, a capability handle, a per-launch
  secret or protected channel state is.
- The native bridge or helper authorizes each operation and the final
  resource after parsing. A trusted UI in front of it does not make the
  arguments trusted.
- OS sandbox, signing, permissions, pipe and file ACLs, and consent
  prompts are real controls when the source sets them.
- A packaging, signing, installer or OS fact the source cannot show makes
  the candidate needs_validation.
```

## Deep links and navigation

**Custom scheme and deep link.** Another app or a web page opens a route
that changes state, imports data, completes sign-in, or picks an account,
with no binding to the current session and no one-time token. Read how the
URL is normalized, duplicate parameters, scheme, host and path matching,
and replayed or stale links.

**Sign-in and pairing hand-off.** OAuth, magic-link, invite, device
pairing or payment callbacks land in the wrong app, profile, tenant or
pending transaction. State must bind to the app, session, account,
operation and expiry.

**Opened files and dropped items.** A file association, drag and drop,
clipboard, notification action or "open with" triggers a privileged
operation without checking content type, the user's intent and the final
target.

## Webview and native bridge

**Navigation reaches the bridge.** A remote or attacker-shaped frame can
call a bridge meant only for packaged content. The origin is checked at
call time and after every navigation, redirect, subframe, popup and error
page; a URL-prefix check at first load is not enough.

**Bridge that does too much.** Page script can pick arbitrary files,
commands, IPC methods, credentials or system actions through a generic
bridge. Look for a method allowlist, normalized arguments, a user gesture
or confirmation, and what the return value discloses.

**File and origin access.** Remote content reads app-local files, a
privileged custom scheme or an internal origin because file access, mixed
content, dev tools, or a custom protocol handler joins origins. A loose
setting with nothing protected behind it is advice.

## Local IPC and listeners

**Peer not authenticated.** A named pipe, Unix socket, loopback HTTP or
WebSocket listener, RPC port or shared memory accepts a lower-trust peer
without checking OS credentials, code identity, a token or channel
ownership. A browser page can reach a loopback HTTP listener; a listener
with no secret and no Origin check is reachable from any site the user
visits. Something worth having must sit behind the channel.

**Claimed principal versus channel.** The channel belongs to one app or
user, but a field in the request picks another user, profile or
capability. Bind each method and resource to the peer's real identity.

**Lifecycle of a channel.** Predictable request ids, reused handles,
inherited descriptors, a pipe or socket created with a broad ACL, a port
any user can take first, restart behaviour that lets one peer answer or
cancel another's call.

## Privileged helpers, install and update

**Confused deputy.** A low-privilege caller picks the command, file,
service, user or setting a privileged helper acts on, with no check per
operation. The helper validates normalized arguments itself.

**Install, update, repair.** A privileged installer or updater reads
manifests, scripts, packages, symlinks or working directories a lower-trust
user can write, after authorization. An update applied without checking
its signature or hash against a key the app ships with.

**Local files and check-then-use.** The app checks a path, then a symlink,
junction, mount or case change swaps it before a privileged read or write.
Use the opened handle and check final ownership.

**Local secrets.** A token file, credential store item, backup, log,
clipboard or notification preview readable by another app, profile or
user with less authority. Plaintext readable only by the same OS account is
not a finding on its own; name the lower-trust reader and what the
credential can do.

## App state

**Logout and account switch.** Cached data, background tasks,
notifications, local databases or webview storage survive logout or an
account change and show under the next account.

**Pending action.** A notification, shortcut or deferred operation
approves a different action than it showed, runs after expiry, or uses
another profile's pending state.

## Universal moves

- List every process, local endpoint, URL scheme, file association,
  webview origin and helper: what identity it runs as, who can call it,
  and what it can do.
- Read what the build actually ships: manifests, installer rules,
  protocol registrations, listener bind addresses, ACLs on pipes and
  files. A declaration in source can be overwritten at packaging.
- Check with dummy profiles and local fixtures only.

## Validation rules

1. Name the attacker's starting capability, the OS or app principal
   crossed, the channel, the argument or state they control, and the
   operation or disclosure they get.
2. Confirm which sandbox, peer credential, signature, ACL, consent and
   installer controls apply. Unknown packaging or runtime facts are
   needs_validation.
3. A bridge finding cites both the navigation or origin control and the
   native sink. An IPC finding cites peer authentication and the check per
   resource. A helper finding checks the final normalized destination.
4. Stop at the boundary result; nothing into persistence or wider system
   change.
