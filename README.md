# claude_skills

Προσωπικός **Claude Code skills marketplace** για OpenCart projects (digital4u).
Ένα κεντρικό σημείο: γράφω/ενημερώνω τα skills εδώ, και κάθε μηχάνημα τα τραβάει από το repo.

> Repo: **`Opencart4u/claude_skills`** · marketplace: **`claude_skills`** · plugin: **`oc-tools`**

---

## Plugin: `oc-tools`

| Skill | Πότε ενεργοποιείται |
|---|---|
| `file-ownership` | Σε κάθε file/folder operation — ελέγχει & διορθώνει root ownership. |
| `ocmod-modifications` | Όταν γράφω/διορθώνω OCMOD `install.xml` modifications για OpenCart. |
| `htaccess-hardening` | Όταν σκληραίνω/ελέγχω το `.htaccess` ενός OpenCart store (PHP-deny, security headers, .md block). |

---

## Εγκατάσταση σε ένα μηχάνημα

Μέσα στο **Claude Code prompt** (όχι στο shell):

```
/plugin marketplace add Opencart4u/claude_skills
/plugin install oc-tools@claude_skills
```

Επιβεβαίωση φόρτωσης: `/reload-plugins` (δείχνει «Reloaded X plugins, 2 skills…») ή `/plugin` → tab **Installed**.

Shell ισοδύναμα (για headless/scripts):

```bash
claude plugin marketplace add Opencart4u/claude_skills
claude plugin install oc-tools@claude_skills
```

---

## Καθημερινή χρήση

- **Προσθήκη/αλλαγή skill:** edit στο repo → `git push` → στα μηχανήματα τραβάς τη νέα έκδοση (δες **Ενημέρωση** πιο κάτω).
- Μετά το install, τα skills φορτώνονται **αυτόματα σε κάθε νέο session** — δεν κάνεις τίποτα.

---

## Ενημέρωση σε νέα έκδοση (όταν προστεθεί/αλλάξει skill)

> ⚠️ **Προσοχή:** το `/plugin marketplace update` ανανεώνει **μόνο τον κατάλογο** — ΔΕΝ αναβαθμίζει
> το ήδη εγκατεστημένο plugin. Ομοίως το `/reload-plugins` φορτώνει μόνο ό,τι είναι ήδη στον δίσκο,
> δεν τραβάει νέα έκδοση. Γι' αυτό μετά από ένα `git push` το νέο skill **δεν εμφανίζεται** με σκέτο
> `marketplace update` (π.χ. ο μετρητής μένει 15 αντί 16).

**Σωστός τρόπος — uninstall + reinstall:**

```
/plugin uninstall oc-tools@claude_skills
/plugin marketplace update claude_skills
/plugin install oc-tools@claude_skills
/reload-plugins
```

Επιβεβαίωση: `/plugin` → tab **Installed** → `oc-tools` πρέπει να δείχνει τη νέα `version` και τον
σωστό αριθμό skills.

**Αν πάλι δείχνει παλιά έκδοση → καθάρισε το cache** (κρατιέται τοπικά):

```bash
rm -rf ~/.claude/plugins/cache         # Windows: C:\Users\<user>\.claude\plugins\cache
```
Μετά restart το Claude Code και ξανά `install` + `/reload-plugins`.

**💡 Auto-update (για να μη γίνεται χειροκίνητα):** `/plugin` → tab **Marketplaces** → `claude_skills`
→ **Enable auto-update**. Από εκεί και πέρα κάθε `git push` τραβιέται μόνο του στο startup.

> 🔖 Καλή πρακτική: σε κάθε αλλαγή skill, ανέβαζε το `version` στο `oc-tools/.claude-plugin/plugin.json`
> (π.χ. 0.2.0 → 0.3.0). Έτσι ξεχωρίζει καθαρά ότι υπάρχει νέα έκδοση.

---

## SSH / πολλά μηχανήματα — τι ισχύει ανά server

**Το install γίνεται μία φορά ανά SERVER, όχι ανά project/session.** Το plugin αποθηκεύεται στο
`~/.claude` του χρήστη του server.

| Σενάριο | Χρειάζεται install; |
|---|---|
| Ίδιος server, ξανά SSH / άλλα projects / νέα sessions | ❌ Όχι — μένει στο `~/.claude`, φορτώνει αυτόματα |
| **Καινούργιος** server (πρώτη φορά) | ✅ Ναι, μία φορά τα δύο `/plugin` commands |
| Άλλαξε κάτι στο repo (`git push`) | Στους servers: uninstall + reinstall (δες **Ενημέρωση σε νέα έκδοση**) ή auto-update |

⚠️ **Εφήμεροι servers** (containers/VMs που στήνονται καθαροί κάθε φορά, ή χάνεται το `~/.claude`):
εκεί το install χάνεται και ξαναχρειάζεται. Αυτοματοποίησέ το στο provisioning / `.bashrc` / dotfiles
(τρέχουν headless, χωρίς interactive prompt):

```bash
claude plugin marketplace add Opencart4u/claude_skills
claude plugin install oc-tools@claude_skills
```

⚠️ **Auth:** αν το repo είναι **private**, το `marketplace add` σε νέο server θέλει git auth (GitHub
login / credential manager / SSH key / token). Αν είναι **public**, δεν χρειάζεται τίποτα.

---

## Προσθήκη νέου skill

1. Φτιάξε φάκελο `oc-tools/skills/<name>/` και μέσα ΥΠΟΧΡΕΩΤΙΚΑ ένα `SKILL.md`.
2. Ο φάκελος = το `name`. Το `description` πρέπει να λέει καθαρά «πότε χρησιμοποιείται» — βάσει αυτού
   αποφασίζει το Claude αν θα το φορτώσει.
3. `git push` → στα μηχανήματα `/plugin marketplace update claude_skills`.

Μορφή `SKILL.md`:

```markdown
---
name: kebab-case-όνομα
description: Πότε/γιατί ενεργοποιείται. Σαφές — βάσει αυτού το φορτώνει το Claude.
---

# Τίτλος

Οδηγίες / βήματα / κανόνες που θέλω να ακολουθεί το Claude όταν τρέξει αυτό το skill.
Μπορεί να έχει και βοηθητικά αρχεία στον ίδιο φάκελο (scripts, templates) και να τα αναφέρει.
```

---

## Δομή repo

```
claude_skills/
├── .claude-plugin/
│   └── marketplace.json        ← κατάλογος plugins (source: ./oc-tools)
├── README.md
└── oc-tools/                   ← το plugin
    ├── .claude-plugin/
    │   └── plugin.json
    └── skills/
        ├── file-ownership/
        │   └── SKILL.md
        ├── ocmod-modifications/
        │   └── SKILL.md
        └── htaccess-hardening/
            └── SKILL.md
```

---

## Σημειώσεις

- Υπάρχουν **δύο διαφορετικά** συστήματα «Skills»: τα **claude.ai dashboard Skills** (μόνο web/desktop & API)
  και τα **Claude Code CLI Skills** (τοπικά `SKILL.md`). **Δεν** συγχρονίζονται μεταξύ τους — αυτό το repo
  αφορά τα **CLI** skills.
- Επίπεδα φόρτωσης CLI skills: **Plugin** (αυτή η λύση, παντού όπου εγκατασταθεί) · **User**
  (`~/.claude/skills/`) · **Project** (`<project>/.claude/skills/`).
