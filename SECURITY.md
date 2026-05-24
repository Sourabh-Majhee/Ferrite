# Security Policy

## Reporting Security Vulnerabilities

If you discover a security vulnerability in Ferrite, please report it responsibly:

### Reporting Process

1. **Do not create a public GitHub issue** — This exposes the vulnerability to the community
2. **Email security@ferrite.dev** with:
   - Description of the vulnerability
   - Steps to reproduce (if possible)
   - Potential impact
   - Your contact information (name, email)
   - Preferred timeline for disclosure

3. **We will respond within 48 hours** and work with you to:
   - Verify the vulnerability
   - Develop a fix
   - Coordinate disclosure

### Responsible Disclosure

We request a responsible disclosure period of **30 days** to develop and test a fix before public announcement. This allows us to release a secure version to users before details become widely known.

## Security Considerations for Ferrite

### Threat Model

Ferrite is a terminal browser designed to render web content in a character-cell terminal. The following security considerations apply:

#### Protected Against
- **XSS (Cross-Site Scripting)** — JavaScript runs in a sandboxed Boa engine (no access to file system)
- **Man-in-the-Middle (MITM)** — TLS 1.2/1.3 via `rustls` (no OpenSSL vulnerabilities)
- **DNS Poisoning** — Uses system resolver (same as other browsers)
- **Memory Safety Issues** — Written in Rust (no buffer overflows, use-after-free, etc.)

#### Not Protected Against (Out of Scope)
- **Malicious JavaScript** — Still executes (no full sandbox; needed for web compatibility)
- **Social Engineering** — User must decide to trust content
- **Local File Access** — Terminal process has user's permissions
- **Network Eavesdropping** — Before TLS handshake (DNS lookups are plaintext)

### JavaScript Sandbox

JavaScript execution is provided by `boa_engine`, which:
- ✅ Runs in an isolated interpreter (no access to host process memory)
- ✅ No access to the file system (unless explicitly enabled by user)
- ✅ No access to environment variables
- ✅ No access to host command execution
- ❌ Cannot prevent infinite loops (denial of service)

**Future:** Web Workers will add thread-level isolation.

### Certificate Validation

TLS certificates are validated using `rustls` and `rustls-webpki`:
- ✅ Verifies certificate chain
- ✅ Checks certificate validity dates
- ✅ Validates hostname matching
- ✅ No support for self-signed certificates by default (can be bypassed with `--insecure` flag)

### Cookie Security

Cookies are stored in `~/.config/ferrite/cookies.json`:
- ✅ `Secure` flag respected (only sent over HTTPS)
- ✅ `HttpOnly` flag respected (not accessible to JavaScript)
- ✅ `SameSite` attribute supported (`Strict`, `Lax`, `None`)
- ❌ Cookies are stored in plaintext (file permissions are key)

**Recommendation:** Use file permissions to protect the cookies file:
```bash
chmod 600 ~/.config/ferrite/cookies.json
```

### Content Security Policy (CSP)

CSP headers are recognized but not fully enforced:
- 🟡 Headers are parsed
- 🟡 Warnings logged for violations
- ❌ Violations do not block content (enforcement deferred to Phase 2)

### Subresource Integrity (SRI)

Not yet implemented. Planned for Phase 2.

## Known Security Limitations

### 1. No Sandboxing for Iframes
Iframes run in the same context as parent document. Full isolation not yet implemented.
- **Mitigation:** Be cautious with untrusted iframe content
- **Planned Fix:** Phase 2 (thread-based sandbox for iframes)

### 2. No WebAssembly Support
WebAssembly execution could allow arbitrary native code. Currently not supported.
- **Mitigation:** N/A (feature not implemented)
- **Future:** Will require careful sandboxing

### 3. JavaScript Execution Timeout
Long-running scripts can freeze the terminal. No timeout mechanism yet.
- **Mitigation:** Use `Ctrl+C` to interrupt
- **Planned Fix:** Phase 2 (configurable timeout for scripts)

### 4. History Leaks
Navigation history could reveal sensitive information. Currently stored in plaintext.
- **Mitigation:** Use `--no-history` flag or manually clear `~/.config/ferrite/history.txt`
- **Future:** Consider encryption

## Security Best Practices for Users

1. **Keep Ferrite Updated** — Security patches are released regularly
2. **Use HTTPS** — Always prefer HTTPS over HTTP
3. **Verify SSL Certificates** — Do not ignore certificate warnings
4. **Protect Cookie Files** — Set file permissions to `600`
5. **Distrust JavaScript** — Be aware that JavaScript can run code
6. **Use Untrusted Mode** — `--untrusted` flag (planned) disables JavaScript

## Security Contact

- **Email:** security@ferrite.dev
- **GPG Key:** [Public key here]
- **Security Advisories:** [Link to GitHub Security Advisories]

## Acknowledgments

We thank the security research community for reporting issues responsibly. All vulnerability reporters are credited in security advisories (unless they request anonymity).

## References

- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [OWASP Security Headers](https://owasp.org/www-project-secure-headers/)
- [CWE/SANS Top 25](https://cwe.mitre.org/top25/)
- [Rustls Security Documentation](https://docs.rs/rustls/)
