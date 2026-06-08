---
name: htaccess-hardening
description: >
  Harden OpenCart .htaccess files idempotently. Use this skill whenever securing or reviewing an
  OpenCart store's Apache/LiteSpeed config — blocking PHP execution in writable folders
  (system/storage/upload, image), adding security response headers (X-Content-Type-Options,
  X-Frame-Options, HSTS, CSP, Permissions-Policy) to the root .htaccess, or extending the
  "Prevent Direct Access" FilesMatch to cover .md files. Always add only what is missing — never
  overwrite an existing .htaccess or duplicate a rule. Includes a one-pass idempotent SSH deploy
  script and server-fleet notes (mod_php vs PHP-FPM/LiteSpeed, Apache 2.2 vs 2.4).
---

# OpenCart `.htaccess` Hardening — Digital4u

Κανόνες σκλήρυνσης (hardening) για κάθε OpenCart project. Ίδια λογική με το `ocmod-modifications`
skill — εδώ μαζεμένα σε ένα έγγραφο αναφοράς.

**Βασική αρχή σε όλα:** προσθέτουμε μόνο ό,τι λείπει. Ποτέ δεν κάνουμε overwrite ολόκληρο
`.htaccess` και ποτέ δεν διπλασιάζουμε κανόνα — άλλοι κανόνες (rewrite, cache, redirects) ζουν ήδη
εκεί.

---

## 1. Φάκελοι `upload` & `image` — μπλοκ εκτέλεσης PHP

Στόχος: οι δύο εγγράψιμοι φάκελοι που είναι κλασικοί στόχοι για web-shell / card skimmer uploads να
μην εκτελούν PHP.

Διαδρομές:
- `system/storage/upload/`
- `image/`

Περιεχόμενο `.htaccess` (ίδιο και για τους δύο):

```apache
<FilesMatch "\.(php|phtml|phar|php5|php7|php8)$">
    Require all denied
</FilesMatch>
<IfModule mod_php.c>
    php_flag engine off
</IfModule>
<IfModule mod_php7.c>
    php_flag engine off
</IfModule>
```

- Το `<FilesMatch> ... Require all denied` είναι η ουσιαστική προστασία σε κάθε server
  (Apache, LiteSpeed, PHP-FPM).
- Το `php_flag engine off` ισχύει μόνο με `mod_php`, γι' αυτό τυλίγεται σε `<IfModule>` ώστε να μη
  ρίχνει **500** σε PHP-FPM/LiteSpeed. Περιλαμβάνονται και `mod_php.c` (PHP 8) και `mod_php7.c`
  (PHP 7) για το μικτό fleet.

**Idempotency:**
- Δεν υπάρχει `.htaccess` στον φάκελο → δημιουργία με το παραπάνω block.
- Υπάρχει ήδη `.htaccess` με `Require all denied` → καμία ενέργεια.
- Υπάρχει `.htaccess` χωρίς τον κανόνα → append του block (όχι overwrite, όχι διπλό).

---

## 2. Κεντρικό (root) `.htaccess` — Security headers

Πρόσθεσε το παρακάτω block αν δεν υπάρχει ισοδύναμο (έλεγχος για το marker comment ή για
`X-Content-Type-Options`):

```apache
# Security related headers
<IfModule mod_headers.c>
Header set X-Content-Type-Options nosniff
Header set X-XSS-Protection "1; mode=block"
Header always append X-Frame-Options SAMEORIGIN
Header set X-Permitted-Cross-Domain-Policies "none"
Header set Strict-Transport-Security "max-age=31536000; includeSubDomains; preload"
Header set Content-Security-Policy "default-src * data:; script-src https: 'unsafe-inline' 'unsafe-eval'; style-src https: 'unsafe-inline'"
Header set Referrer-Policy "no-referrer-when-downgrade"
Header always set Permissions-Policy "geolocation=(),camera=(),microphone=(),fullscreen=(self)"
</IfModule>
```

**Προσοχή — ανά project:**
- `Strict-Transport-Security` με `preload` προϋποθέτει 100% HTTPS. Μην το βάζεις σε staging /
  non-TLS host.
- `Permissions-Policy` μπλοκάρει `camera`/`microphone`. Σε καταστήματα με barcode scanner ή Findbar,
  χαλάρωσέ το τοπικά (π.χ. `camera=(self)`).

---

## 3. Κεντρικό (root) `.htaccess` — προσθήκη `.md` στο "Prevent Direct Access"

Κάθε OpenCart ships έναν τέτοιο κανόνα στο root `.htaccess`:

```apache
# Prevent Direct Access to files
<FilesMatch "(?i)((\.tpl|\.twig|\.ini|\.log|(?<!robots)\.txt))">
Require all denied
## For apache 2.2 and older, replace "Require all denied" with these two lines :
# Order deny,allow
# Deny from all
</FilesMatch>
```

Πρόσθεσε `\.md` **στο υπάρχον pattern** (επιτόπου, όχι δεύτερο block), ώστε να μη διαβάζονται απευθείας
τα Markdown αρχεία (README.md, SKILL.md, install notes) που έρχονται με extensions/skills:

```apache
<FilesMatch "(?i)((\.tpl|\.twig|\.ini|\.log|\.md|(?<!robots)\.txt))">
```

**Idempotency:** αν το pattern έχει ήδη `\.md`, άστο ως έχει. Αν λείπει εντελώς ο κανόνας (σπάνιο),
πρόσθεσε ολόκληρο το block με το `.md` μέσα.

---

## Σύνοψη idempotency

| Στόχος | Αν λείπει | Αν υπάρχει σωστό |
|--------|-----------|------------------|
| `upload/.htaccess`, `image/.htaccess` | δημιουργία/append του PHP-deny block | καμία ενέργεια |
| Root security headers | append του block | καμία ενέργεια |
| Root `.md` στο FilesMatch | επεξεργασία της γραμμής επιτόπου | καμία ενέργεια |

---

## One-pass deploy script (SSH)

Τρέξε το από το **document root** του store. Είναι idempotent — μπορείς να το ξανατρέξεις χωρίς
παρενέργειες. Πάντα κάνε ένα backup πρώτα (`cp .htaccess .htaccess.bak`).

```bash
#!/usr/bin/env bash
# OpenCart .htaccess hardening — run from the store's document root.
set -euo pipefail

# --- 1. upload & image: block PHP execution ---
read -r -d '' PHP_DENY <<'EOF' || true
<FilesMatch "\.(php|phtml|phar|php5|php7|php8)$">
    Require all denied
</FilesMatch>
<IfModule mod_php.c>
    php_flag engine off
</IfModule>
<IfModule mod_php7.c>
    php_flag engine off
</IfModule>
EOF

for d in system/storage/upload image; do
  f="$d/.htaccess"
  if [ -d "$d" ]; then
    if [ ! -f "$f" ] || ! grep -q "Require all denied" "$f"; then
      printf '%s\n' "$PHP_DENY" >> "$f"
      echo "hardened: $f"
    else
      echo "already protected: $f"
    fi
  else
    echo "skip (no dir): $d"
  fi
done

# --- 2. root .htaccess: security headers ---
ROOT=".htaccess"
if [ -f "$ROOT" ]; then
  cp "$ROOT" "$ROOT.bak.$(date +%Y%m%d%H%M%S)"

  if ! grep -q "X-Content-Type-Options" "$ROOT"; then
    cat >> "$ROOT" <<'EOF'

# Security related headers
<IfModule mod_headers.c>
Header set X-Content-Type-Options nosniff
Header set X-XSS-Protection "1; mode=block"
Header always append X-Frame-Options SAMEORIGIN
Header set X-Permitted-Cross-Domain-Policies "none"
Header set Strict-Transport-Security "max-age=31536000; includeSubDomains; preload"
Header set Content-Security-Policy "default-src * data:; script-src https: 'unsafe-inline' 'unsafe-eval'; style-src https: 'unsafe-inline'"
Header set Referrer-Policy "no-referrer-when-downgrade"
Header always set Permissions-Policy "geolocation=(),camera=(),microphone=(),fullscreen=(self)"
</IfModule>
EOF
    echo "added: security headers"
  else
    echo "already present: security headers"
  fi

  # --- 3. root .htaccess: add \.md to Prevent Direct Access FilesMatch ---
  if grep -q 'tpl|\\.twig|\\.ini|\\.log' "$ROOT" && ! grep -q '\\.log|\\.md' "$ROOT"; then
    sed -i 's/\\\.log|(?<!robots)/\\.log|\\.md|(?<!robots)/' "$ROOT"
    echo "added: .md to Prevent Direct Access"
  else
    echo "already present (or rule missing): .md in FilesMatch"
  fi
else
  echo "WARNING: no root .htaccess found — is this the store document root?"
fi
```

> Σημείωση για το βήμα 3: το `sed` στοχεύει το ακριβές default pattern του OpenCart
> (`...\.log|(?<!robots)...`). Αν ένα project έχει τροποποιημένο pattern, έλεγξε/πρόσθεσε το `\.md`
> με το χέρι αντί να βασιστείς στο sed.

---

## Σημειώσεις για το server fleet

- **mod_php vs PHP-FPM/LiteSpeed:** στους περισσότερους cPanel servers τρέχει PHP-FPM/suEXEC, όχι
  `mod_php`. Εκεί το `php_flag` αγνοείται — γι' αυτό υπάρχει το `<IfModule>` wrapper και η ουσιαστική
  γραμμή είναι το `Require all denied`.
- **Apache 2.2 / CentOS 7.9:** σε πολύ παλιό Apache το `Require all denied` δεν ισχύει· αντικατέστησέ
  το με `Order deny,allow` + `Deny from all` (όπως ήδη σχολιάζει το default OpenCart block).
- **AlmaLinux 8.x:** Apache 2.4 — το `Require all denied` δουλεύει κανονικά.
