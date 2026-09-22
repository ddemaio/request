# openSUSE Event Requests — form

A single static HTML page. No build step, no framework, no database. It renders a
request form, scores the request against openSUSE's community criteria as it is
filled in, and emails the resulting summary to one or more people.

Nothing is stored anywhere. The only persistence is a draft kept in the
submitter's own browser (`localStorage`), which never leaves their machine.

---

## Repository layout

```
opensuse-event-requests/
├── index.html      ← the form (rename opensuse-event-request.html to this)
├── README.md
└── LICENSE
```

That is the whole project. Keep it that way — a form nobody can build locally is
a form nobody will contribute to.

---

## 1. Create the repository

```bash
git init opensuse-event-requests
cd opensuse-event-requests
cp /path/to/opensuse-event-request.html index.html
cp /path/to/README.md .
git add .
git commit -m "Add openSUSE event request form"
git branch -M main
git remote add origin git@github.com:<org-or-user>/opensuse-event-requests.git
git push -u origin main
```

Add a licence before anyone else contributes. MIT or the openSUSE project's usual
choice is fine; put it in `LICENSE` and reference it here.

## 2. Turn on GitHub Pages

In the repository: **Settings → Pages → Source: Deploy from a branch**, branch
`main`, folder `/ (root)`. Save.

After a minute the form is live at
`https://<org-or-user>.github.io/opensuse-event-requests/`.

If the project has its own domain, put the hostname in **Settings → Pages →
Custom domain**, add a `CNAME` DNS record pointing at `<org>.github.io`, and tick
**Enforce HTTPS**.

## 3. Set up email delivery

The page is static, so it cannot send mail itself. It posts the finished summary
to a form-relay service, which forwards it to the addresses listed in the code.

The default provider is [FormSubmit](https://formsubmit.co) — free, no account,
no JavaScript SDK, and the recipient list lives in your repository rather than in
somebody's dashboard.

**Activate the primary address once:**

1. Open `index.html` and set `CONFIG.PRIMARY` to the address that should own the
   requests — ideally a shared alias such as `events@opensuse.org`, not an
   individual.
2. Commit, push, open the live page, and submit one test request.
3. FormSubmit emails that address an activation link. Click it. Every later
   submission is delivered without a prompt.

Do this from the live GitHub Pages URL, not from a local file — the service ties
activation to the submitting domain.

## 4. Add or remove recipients

Near the top of the `<script>` block in `index.html`:

```js
var CONFIG = {
  PRIMARY: "events@opensuse.org",

  RECIPIENTS: [
    "board-member@opensuse.org",
    "advocate@example.org"
  ],

  SUBJECT_PREFIX: "openSUSE event request",
  PROVIDER: "formsubmit"
};
```

`PRIMARY` is the To: address. Everyone in `RECIPIENTS` is CC'd, so they all see
each other and can reply-all to the requester — which is what you want for a
review that happens in the open.

To join the review rotation, someone opens a pull request adding one line to
`RECIPIENTS`. That is the whole process, and it leaves a public record of who has
been receiving requests and since when.

Two practical notes: only `PRIMARY` needs activating, and the addresses are
visible in the page source, so expect them to be scraped. Use role aliases with
spam filtering rather than personal mailboxes.

## 5. Test before announcing it

- Submit a complete request and confirm every recipient receives it.
- Submit a deliberately weak one — commercial organiser, €400 ticket, one person,
  no outreach plan — and check the assessment panel lands in the red band.
- Try it on a phone; the layout is responsive and the panel sticks to the bottom.
- Switch your OS to dark mode and reload.
- Block the request in devtools and confirm the failure message appears telling
  the submitter to email the summary manually.

---

## Alternative delivery methods

**Formspree or Web3Forms** — same shape, a single `fetch` to a different URL.
Both need an account, and Web3Forms keeps the recipient list server-side, which
loses the pull-request workflow above. Swap the endpoint in the `send` handler if
the project already pays for one of these.

**GitHub Issues instead of email** — arguably a better fit for an open project.
Replace the `fetch` with a link that opens a prefilled issue:

```js
var url='https://github.com/<org>/opensuse-event-requests/issues/new?title='
        +encodeURIComponent(subject)+'&body='+encodeURIComponent(body);
window.open(url,'_blank');
```

Reviews then happen in public, requests are searchable, and watchers get notified
without any recipient list at all. The trade-off is that requesters need a GitHub
account and their travel dates and contact details become public — so if you go
this way, drop the email field from the summary and ask for contact details
separately.

**Self-hosted relay** — if the project would rather not depend on a third party
with submitters' personal data, run a small SMTP relay (for example
`chibisov/formsend` in a container) on openSUSE infrastructure and point the
`fetch` at it. This is the only option that keeps everything inside the project's
own hands, and worth considering given that the form collects names, emails and
travel intentions.

---

## Logo and toolbar assets

The button mark in the page header and universe toolbar, and the toolbar
markup itself, are hand-built approximations — the sandbox used to write this
page blocks remote images and scripts, so the real assets could not be pulled
in. When deploying on GitHub, swap in the official mark served at
`https://static.opensuse.org/favicon.svg` and align the masthead with the
actual header in the [openSUSE/get-o-o repo](https://github.com/openSUSE/get-o-o),
which uses the shared Chameleon theme. That keeps this page in sync with the
rest of openSUSE's sites as the theme changes, and avoids shipping an
approximated logo.

---

## Tuning the assessment

The scoring lives in one function, `assess()`, near the bottom of the file. Each
signal is a single line:

```js
if(fl.indexOf('FOSS focused')>-1) add('pos','FOSS-focused programme',2);
```

`add(kind, label, points)` — `kind` is `pos`, `neg` or `neu` and only controls the
colour of the bullet. Change the weight, or add a signal, in one place.

The band thresholds are directly below:

```js
if(s>=26){...'Strong fit — full support plausible';}
else if(s>=16){...'Good fit — partial funding likely';}
else if(s>=8){...'Borderline — in-kind support or materials only';}
else if(s>=0){...'Needs a stronger community case';}
else {...'Poor fit — likely declined as not community-driven';}
```

The numbers are a starting point, not policy. Run a dozen past requests through
the form, see where they land, and adjust until the bands match decisions the
Board would actually have made. Record the agreed weights in this README so
changes to them are argued in pull requests rather than made quietly.

The panel is advisory and says so on screen. Keep it that way — the moment people
believe the number decides, they will optimise for the number.

---

## Maintenance

Anything that is policy rather than code should be easy to find and change:

- the 8-week notice window (in the acknowledgement text and in `assess()`)
- the travel-programme pointer near the top — replace the placeholder text with
  the real URL
- the event-report expectation and where reports should be published
- the recipient list

All four are plain strings in `index.html`.
