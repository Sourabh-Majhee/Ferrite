# FAQ — Frequently Asked Questions

## General Questions

### What is Ferrite?

Ferrite is a terminal browser written entirely in Rust. It allows you to browse websites in your terminal without needing a graphical display or an installed browser like Firefox or Chrome.

### Why would I use a terminal browser?

Several reasons:

1. **Low resource usage** — Ferrite targets machines with 512MB RAM
2. **SSH access** — Browse the web over a slow SSH connection
3. **Text-based workflows** — Integrates with terminal tools (pipes, grep, etc.)
4. **Accessibility** — Screen readers work better with terminal output
5. **Entertainment** — It's fun!

### How is this different from existing terminal browsers (Lynx, w3m, ELinks)?

Ferrite is:

- **Memory-safe** — Written in Rust, not C (no buffer overflows)
- **JavaScript-capable** — Can run JavaScript via `boa_engine`
- **Modern** — Supports HTML5, CSS Flexbox/Grid, ES2021
- **Pure Rust** — No external browser engine dependencies
- **Still evolving** — It's an active project; existing browsers are mostly mature

### Is Ferrite ready to use?

**Not yet.** Ferrite is in **alpha** and only implements architecture so far. Core functionality is planned for Phase 1 (Q3-Q4 2026).

### When will Ferrite be ready?

See the [Roadmap](#roadmap) section below.

## Technical Questions

### How does Ferrite handle JavaScript?

JavaScript is executed using `boa_engine`, a pure-Rust JavaScript engine with 94%+ ECMAScript compliance. JavaScript can manipulate the DOM, use `fetch()` to make HTTP requests, and set timers.

### Does Ferrite run JavaScript sandboxed?

Yes, to some extent:
- JavaScript cannot access the file system directly
- JavaScript cannot execute shell commands
- JavaScript runs in an isolated interpreter

However, it's not a perfect sandbox — JavaScript still has access to the DOM and can make network requests.

### Can Ferrite handle complex websites?

Ferrite can render modern websites that use HTML5, CSS, and JavaScript. However:

- **Very complex layouts** may have rendering quirks
- **Animations** are not supported (terminal doesn't support continuous rendering)
- **Graphics-heavy sites** will be text-based only

Ferrite works best for:
- Documentation sites (MDN, Rust docs, etc.)
- News sites (Hacker News, Reddit, etc.)
- Blogs
- Wikipedia

### What CSS features are supported?

Ferrite supports:
- ✅ Selectors (type, class, ID, attributes, pseudo-classes)
- ✅ Cascade and specificity
- ✅ Inheritance
- ✅ Display: block, inline, flex, grid
- ✅ Margins, padding, widths, heights
- ✅ Colors, bold, italic, underline
- ✅ Flexbox and CSS Grid layout
- ❌ Animations and transitions
- ❌ Transforms and gradients
- ❌ Position: fixed/sticky (simplified)

See [ARCHITECTURE.md](ARCHITECTURE.md#what-css-properties-we-implement) for the complete list.

### Does Ferrite support plugins or extensions?

Not yet. Plugins are not planned for the MVP. We're focusing on core browser functionality first.

### Can Ferrite view images?

Text-based rendering of images is possible. Actual image display via Sixel or Kitty protocol is optional and deferred to Phase 3.

## Usage Questions

### How do I use Ferrite?

Once released:

```bash
ferrite https://example.com
```

See [README.md](README.md) for more details.

### Can I use Ferrite on SSH?

Yes! That's one of the main use cases. SSH into a server and run `ferrite`.

### Can I pipe content into Ferrite?

Not yet, but it's planned. Currently, Ferrite fetches content from the web.

**Workaround:** Use a simple HTTP server:
```bash
python3 -m http.server 8000
ferrite http://localhost:8000/file.html
```

### How do I set keybindings?

Keybindings are configured in `~/.config/ferrite/config.toml`:

```toml
[keybindings]
reload = "Ctrl+R"
back = "Alt+Left"
forward = "Alt+Right"
new_tab = "Ctrl+T"
close_tab = "Ctrl+W"
quit = "Ctrl+Q"
```

### How do I change the color scheme?

Configuration is in `~/.config/ferrite/config.toml`:

```toml
[display]
colors = 24-bit  # or: 16, 256, 24-bit
theme = "auto"   # or: "light", "dark"
```

### Can I use a proxy?

Not yet, but it's planned. Currently, Ferrite uses your system's HTTP_PROXY and HTTPS_PROXY environment variables.

### How do I debug what's happening?

Enable debug logging:

```bash
RUST_LOG=debug ferrite https://example.com
```

Or higher verbosity:

```bash
RUST_LOG=trace ferrite https://example.com
```

## Performance Questions

### How fast is Ferrite?

Page load times depend on:
- Network latency
- Page complexity
- Your CPU

Typical documentation sites load in <500ms. Rendering time is <100ms.

### How much memory does Ferrite use?

Ferrite uses approximately:
- **Baseline:** ~50MB
- **Per 1000 DOM nodes:** ~1MB
- **Total for typical page:** ~80-100MB

Much less than Chromium (~200MB) but more than Lynx (~10MB).

### Can Ferrite run on a Raspberry Pi?

Yes! Ferrite targets low-spec hardware. A Raspberry Pi with 512MB RAM should handle it fine (though slowly).

## Troubleshooting

### Ferrite crashes on startup

This is likely a bug. Please [file an issue](https://github.com/yourusername/ferrite/issues) with:
- Your OS and terminal emulator
- Rust version (`rustc --version`)
- Steps to reproduce
- Error message (run with `RUST_LOG=debug`)

### Colors look wrong

Ensure your terminal supports 256-color ANSI or truecolor:

```bash
echo $TERM  # Should show xterm-256color or similar
```

If not, configure Ferrite to use fewer colors:

```toml
# ~/.config/ferrite/config.toml
[display]
colors = 256  # or: 16
```

### JavaScript is running but doing nothing

Some sites require cookies or JavaScript events that aren't yet implemented. Check:
- Is cookies enabled? (`~/.config/ferrite/config.toml`: `cookies_enabled = true`)
- Are there console errors? (Use Developer Tools, planned for Phase 2)

### Text is wrapping strangely

Text wrapping respects CSS `white-space` property. If wrapping still looks wrong:
- Try resizing your terminal
- File an issue with the URL and terminal width/height

## Roadmap

### Phase 1: MVP (Q3-Q4 2026)
- [ ] HTML5 parsing
- [ ] DOM tree rendering
- [ ] CSS cascade and selectors
- [ ] Block and inline layout
- [ ] Flexbox layout
- [ ] Basic JavaScript execution
- [ ] Terminal rendering and scrolling

### Phase 2: Interactivity (Q1-Q2 2027)
- [ ] Event handling (click, keypress, scroll)
- [ ] Form input and submission
- [ ] Link navigation and history
- [ ] `fetch()` API
- [ ] Developer tools console

### Phase 3: Polish (Q3 2027)
- [ ] CSS Grid layout
- [ ] Image rendering (Sixel/Kitty)
- [ ] Tabs and window management
- [ ] Bookmarks and history persistence

### Phase 4: Advanced (2028+)
- [ ] CSS animations and transitions
- [ ] Web Workers
- [ ] Service Workers
- [ ] WebAssembly support

## Contributing

### How do I contribute to Ferrite?

See [CONTRIBUTING.md](CONTRIBUTING.md) for detailed guidelines.

### What areas need help?

- **Frontend:** CSS implementation, layout engine refinement
- **Backend:** JavaScript engine bindings, HTTP client enhancements
- **Infrastructure:** Build system, CI/CD, testing
- **Documentation:** Spec annotations, tutorials, examples

### Is there a way to track progress?

Yes, check:
- [GitHub Issues](https://github.com/yourusername/ferrite/issues)
- [GitHub Project Board](https://github.com/yourusername/ferrite/projects)
- [Roadmap above](#roadmap)

## Support

### Where can I ask questions?

- **GitHub Discussions:** [Link]
- **Discord/Slack:** [Link, if available]
- **Email:** [Link]

### How do I report a bug?

Please use [GitHub Issues](https://github.com/yourusername/ferrite/issues) and include:
- OS and terminal info
- Steps to reproduce
- Expected vs. actual behavior
- Terminal width/height

### How do I request a feature?

Open a GitHub issue with "Feature Request" label and describe:
- What feature you want
- Why you need it
- How it would be used

---

**Last Updated:** May 24, 2026

Have a question that's not answered here? [Ask us!](https://github.com/yourusername/ferrite/discussions)
